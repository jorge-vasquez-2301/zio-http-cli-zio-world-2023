---
info: Slides for presentation at ZIO World 2023
theme: 'default'
presenter: false
download: false
exportFilename: 'presentation'
export:
  format: pdf
  timeout: 30000
  dark: auto
  withClicks: true
  withToc: true
highlighter: 'shiki'
lineNumbers: true
monaco: false
remoteAssets: true
selectable: true
record: false
colorSchema: 'dark'
aspectRatio: '16/9'
canvasWidth: 980
favicon: 'https://cdn.jsdelivr.net/gh/slidevjs/slidev/assets/favicon.png'
drawings:
  enabled: false
  persist: false
  presenterOnly: false
  syncAll: true

transition: slide-left
background: /megaphone.jpg
class: 'text-right'
---

### Teach Your Web API to Speak Loud and Clear CLI!

ZIO World - April 2023

---
transition: slide-left
layout: image-right
image: /yo2.jpg
class: "flex items-center text-2xl"
---

<div>
  <p>Jorge Vásquez</p>
  <p>Scala Developer/ZIO Trainer</p>
  <p class="text-red-500">@Scalac</p>
</div>


---
transition: slide-left
layout: image
image: /laptop.jpg
class: "flex h-screen justify-end items-center"
---

# Introduction

<style>
h1 {
  @apply text-6xl !important
}
</style>

---
transition: slide-left
layout: default
---

# **Everything** is an **API**

<div class="flex w-full h-3/5 items-center gap-5">
  <div>
    <ul>
      <li v-click>Most Scala-based backend applications are <strong>web APIs</strong></li>
      <li v-click>
        <p>They typically use:</p>
        <ul>
          <li><strong>HTTP</strong> protocol</li>
          <li><strong>JSON</strong> format</li>
        </ul>
      </li>
    </ul>
  </div>
  <div v-click><img src="/apisEverywhere.jpg" class="h-60"/></div>
</div>

---
transition: slide-left
layout: default
---

# **Testing** your APIs

<div class="flex w-full h-3/5 items-center gap-5">
  <div>
    <ul>
      <li v-click>As a developer, it's <strong>crucial</strong> to <strong>test your APIs</strong> during development</li>
      <li v-click>Testing your APIs <strong>from Scala</strong> is easy with <strong>ZIO HTTP</strong></li>
    </ul>
  </div>
  <div v-click><img src="/tooEasy.jpg"/></div>
</div>

---
transition: slide-left
layout: default
---

# **Example:** Users API with **ZIO HTTP Endpoints**

```scala {1-2|4|5-8|10-13|15|16-20|18|19|20|22|23-30|25|26|27|29|30|32|33-37|35|36|37|39|40-43|45-48|50-53|55|57|58} {maxHeight:'400px'}
import zio.http.endpoint._
import zio.schema._

// Domain models
final case class User(id: Int, name: String, email: Option[String])
object User {
  implicit val schema = DeriveSchema.gen[User]
}

final case class Post(userId: Int, postId: Int, contents: String)
object Post {
  implicit val schema = DeriveSchema.gen[Post]
}

// Endpoint to get a User by ID
val getUser =
  Endpoint
    .get("users" / int("userId") ?? Doc.p("The unique identifier of the user"))
    .header(HeaderCodec.location ?? Doc.p("The user's location"))
    .out[User] ?? Doc.p("Get a user by ID")

// Endpoint to get a User's posts by userId and postId
val getUserPosts =
  Endpoint
    .get(
      "users" / int("userId") ?? Doc.p("The unique identifier of the user") /
        "posts" / int("postId") ?? Doc.p("The unique identifier of the post"),
    )
    .query(query("name") ?? Doc.p("The user's name"))
    .out[List[Post]] ?? Doc.p("Get a user's posts by userId and postId")

// Endpoint to create a new User
val createUser =
  Endpoint
    .post("users")
    .in[User]
    .out[String] ?? Doc.p("Create a new user")

// Routes
val getUserRoute =
  getUser.implement { case (id, _) =>
    ZIO.succeed(User(id, "Juanito", Some("juanito@test.com")))
  }

val getUserPostsRoute =
  getUserPosts.implement { case (userId, postId, name) =>
    ZIO.succeed(List(Post(userId, postId, name)))
  }

val createUserRoute =
  createUser.implement { user =>
    ZIO.succeed(user.name)
  }

val routes = getUserRoute ++ getUserPostsRoute ++ createUserRoute

// Running the server
val run = Server.serve(routes.toApp).provide(Server.default)
```

---
transition: slide-left
layout: default
---

# **Example:** Calling your APIs from Scala, using **ZIO HTTP**

```scala {all|3|4|5|6|7}
val clientExample: URIO[EndpointExecutor[Unit], Unit] =
  for {
    executor <- ZIO.service[EndpointExecutor[Unit]] // Request an executor from the environment
    // Invoke your endpoints as normal functions, and execute them to ZIO effects
    _        <- executor(getUser(42, Location.parse("some-location").toOption.get)).debug("result1")
    _        <- executor(getUserPosts(42, 200, "adam")).debug("result2")
    _        <- executor(createUser(User(2, "john", Some("john@test.com")))).debug("result3")
  } yield ()
```

---
transition: slide-left
layout: default
---

<div class="flex w-full h-full justify-center items-center">
  <div><img src="/tooEasyDog.jpg" class="h-96"/></div>
</div>

---
transition: slide-left
layout: image
image: /laptop.jpg
class: "flex h-screen justify-end items-center text-right"
---

# Problem: <br/>Calling your APIs from the command-line

<style>
h1 {
  @apply text-3xl !important
}
</style>

---
transition: slide-left
layout: default
---

# **Problem:** Calling your APIs from the command-line

<div class="flex w-full h-3/5 items-center gap-5">
  <div>
    <ul>
      <li v-click>As developers, we can also test our APIs from the <strong>command-line</strong>, using tools like <code>curl</code></li>
      <li v-click>DevOps and SREs in your company may also need to call your APIs from <strong>scripts</strong></li>
    </ul>
  </div>
  <div v-click><img src="/rickEasy.jpg"/></div>
</div>

---
transition: slide-left
layout: default
---

# **Example:** Get User by ID

```bash {1|2}
$ curl --request GET 'http://localhost:8080/users/1'
# Got nothing 🤔
```
<div v-click class="flex w-full h-3/4 justify-center items-center">
  <div><img src="/huh.jpeg" class="h-60"/></div>
</div>

---
transition: slide-left
layout: default
---

# **Example:** Get User by ID

```bash {1|2-13|10}
$ curl --request GET 'http://localhost:8080/users/1' --verbose
# *   Trying 127.0.0.1:8080...
# * Connected to localhost (127.0.0.1) port 8080 (#0)
# > GET /users/1 HTTP/1.1
# > Host: localhost:8080
# > User-Agent: curl/7.79.1
# > Accept: */*
# > 
# * Mark bundle as not supporting multiuse
# < HTTP/1.1 500 Internal Server Error
# < content-length: 0
# < 
# * Connection #0 to host localhost left intact
```
<div v-click class="flex w-full h-1/3 justify-center items-center">
  <div><img src="/huh.jpeg" class="h-40"/></div>
</div>

---
transition: slide-left
layout: default
---

<div class="flex w-full h-full justify-center items-center">
  <div><img src="/sheldonWhy.jpg" class="h-96"/></div>
</div>

---
transition: slide-left
layout: default
---

# **Example:** Get User by ID

```bash {1|2|3}
# It turns out we were missing a header!
$ curl --request GET 'http://localhost:8080/users/1' --header 'location: some location'
# {"id":1,"name":"Joseph","email":"joseph@test.com"}
```

<div v-click class="flex w-full h-3/4 justify-center items-center">
  <div><img src="/rockyVictory.jpg" class="h-60"/></div>
</div>

---
transition: slide-left
layout: default
---

# **Example:** Get Posts by `userId` and `postId`

```bash {1|2|4|5-16|13|18|19|20}
$ curl --request GET 'http://localhost:8080/users/1/posts/1'
# Got nothing 🤔

$ curl --request GET 'http://localhost:8080/users/1/posts/1' --verbose
# *   Trying 127.0.0.1:8080...
# * Connected to localhost (127.0.0.1) port 8080 (#0)
# > GET /users/1/posts/1 HTTP/1.1
# > Host: localhost:8080
# > User-Agent: curl/7.79.1
# > Accept: */*
# > 
# * Mark bundle as not supporting multiuse
# < HTTP/1.1 500 Internal Server Error
# < content-length: 0
# < 
# * Connection #0 to host localhost left intact

# It turns out we were missing a query param!
$ curl --request GET 'http://localhost:8080/users/1/posts/1?name=Luke'
# [{"userId":1,"postId":1,"contents":"API2 RESULT parsed: users/1/posts/1?name=Luke"}]
```

---
transition: slide-left
layout: default
---

# **Example:** Create a new User

```bash {1-4|5|7-10|11-22|19|24|25-29|27|30} {maxHeight:'400px'}
$ curl --request POST 'http://localhost:8080/users' --header 'Content-Type: application/json' --data-raw '{
    "id": 2,
    "name": "Test"
}'
# Got nothing 🤔

$ curl --request POST 'http://localhost:8080/users' --header 'Content-Type: application/json' --data-raw '{
    "id": 2,
    "email": "test@test.com"
}' --verbose
# *   Trying 127.0.0.1:8080...
# * Connected to localhost (127.0.0.1) port 8080 (#0)
# > GET /users/1/posts/1 HTTP/1.1
# > Host: localhost:8080
# > User-Agent: curl/7.79.1
# > Accept: */*
# > 
# * Mark bundle as not supporting multiuse
# < HTTP/1.1 500 Internal Server Error
# < content-length: 0
# < 
# * Connection #0 to host localhost left intact

# It turns out we were missing a JSON field!
$ curl --request POST 'http://localhost:8080/users' --header 'Content-Type: application/json' --data-raw '{
    "id": 2,
    "name": "Test",
    "email": "test@test.com"
}'
# "Created User(2,Test,test@test.com) successfully"
```

---
transition: slide-left
layout: default
---

# **Veredict**

<div class="flex w-full h-3/5 items-center gap-5">
  <div>
    <ul>
      <li v-click>Using tools like <code>curl</code> is <strong>hard</strong></li>
      <li v-click>Zero <strong>discoverability</strong></li>
      <li v-click>Requires writing <strong>headers</strong>, <strong>query params</strong> and constructing <strong>JSON</strong> manually</li>
      <li v-click><strong>Time</strong> consuming</li>
      <li v-click><strong>Error</strong> prone</li>
      <li v-click><strong>Complex</strong> error messages</li>
    </ul>
  </div>
  <div v-click><img src="/caseClosed.jpg"/></div>
</div>

---
transition: slide-left
layout: quote
class: "flex h-screen justify-center items-center"
---

# Can we do **better?**

<style>
h1 {
  @apply text-6xl !important
}
</style>

---
transition: slide-left
layout: image
image: /theater.jpg
class: "flex h-screen justify-center items-center"
---

# Enter ZIO HTTP CLI!

<style>
h1 {
  @apply text-6xl !important
}
</style>

---
transition: slide-left
layout: default
---

# ZIO HTTP **CLI**

<div class="flex w-full h-3/5 items-center gap-5">
  <div>
    <ul>
      <li v-click>Now you can obtain a <code>CLI</code> for <strong>free</strong>, from the definition of your <code>Endpoints</code>!</li>
      <li v-click>Just include <code>zio-http-cli</code> as a dependency in your project to start using it!</li>
    </ul>
  </div>
  <div v-click><img src="/excellent.jpg"/></div>
</div>

---
transition: slide-left
layout: default
---

# **Example:** Obtaing a CLI for your Endpoints, using **ZIO HTTP CLI**

```scala {1-2|4|5|6-8|10|11-20|12|13|14|15|16|17|18|19} {maxHeight:'350px'}
import zio.http.endpoint._
import zio.http.endpoint.cli._

object TestCliApp extends zio.cli.ZIOCliDefault {
  // We already have our Endpoints
  val getUser = ...
  val getUserPosts = ...
  val createUser = ...

  // Now we can obtain a `CliApp` from our Endpoints, for free!
  val cliApp =
    HttpCliApp.fromEndpoints(
      name = "users-mgmt",
      version = "0.0.1",
      summary = "Users management CLI",
      footer = "Copyright 2023",
      host = "localhost",
      port = 8080,
      endpoints = Chunk(getUser, getUserPosts, createUser),
    )
}
```

---
transition: slide-left
layout: default
---

# Using the **CLI:** Nice **documentation**!

```bash {1|2-17|11|12|13}
$ users-mgmt --help
#    __  __________  __________      ____ ___  ____ _____ ___  / /_
#   / / / / ___/ _ \/ ___/ ___/_____/ __ `__ \/ __ `/ __ `__ \/ __/
#  / /_/ (__  )  __/ /  (__  )_____/ / / / / / /_/ / / / / / / /_  
#  \__,_/____/\___/_/  /____/     /_/ /_/ /_/\__, /_/ /_/ /_/\__/  
#                                           /____/                 
#  users-mgmt v0.0.1 -- Users management CLI
#  USAGE
#    $ users-mgmt <command>
#  COMMANDS
#    - get-users --userId integer --location text                     Get a user by ID 
#    - create-users --id integer --name text [--email text]           Create a new user
#    - get-users-posts --userId integer --postId integer --name text  Get a user's posts by userId and postId
#    
#  Copyright 2023
```

---
transition: slide-left
layout: default
---

# Using the **CLI:** Get User by ID

```bash {1|2|3|5|6-8}
$ users-mgmt get-users --userId 1
# Expected to find --location option
# (Now you don't have to guess anymore that you need a `location` header!)

$ users-mgmt get-users --userId 1 --location test-location
# Got response
# Status: Ok
# Body: {"id":1,"name":"Juanito","email":"juanito@test.com"}
```

---
transition: slide-left
layout: default
---

# Using the **CLI:** Get Posts by `userId` and `postId`

```bash {1|2|3|5|6-8}
$ users-mgmt get-users-posts --userId 1 --postId 100
# Expected to find --name option
# (Now you don't have to guess anymore that you need a `name` query param!)

$ users-mgmt get-users-posts --userId 1 --postId 100 --name Pepito
# Got response
# Status: Ok
# Body: [{"userId":1,"postId":100,"contents":"Pepito"}]
```

---
transition: slide-left
layout: default
---

# Using the **CLI:** Create a new User

```bash {1|2|3|4|6|7-9}
$ users-mgmt create-users --id 1 --email juanito@juan.test
# Expected to find --name option
# (Now you don't have to guess anymore that you need a `name` field for the JSON body!)
# (Also, notice you don't have to manually build JSON anymore!)

$ users-mgmt create-users --id 1 --name Juanito --email juanito@juan.test
# Got response
# Status: Ok
# Body: "Juanito"
```

---
transition: slide-left
layout: image
image: /laptop.jpg
class: "flex h-screen justify-end items-center"
---

# Summary

<style>
h1 {
  @apply text-6xl !important
}
</style>

---
transition: slide-left
layout: image-right
image: /summary.jpg
---

# **Summary**

<div class="flex w-full h-3/5 items-center gap-5">
  <div>
    <ul>
      <li v-click>Calling your APIs with <code>curl</code> is <strong>terrible</strong></li>
      <li v-click>On the other hand, ZIO HTTP now gives you CLI apps for your APIs, <strong>for free!</strong></li>
    </ul>
  </div>
</div>

---
transition: slide-left
layout: image-right
image: /summary.jpg
---

# **Summary**

<div class="flex w-full h-4/5 items-center gap-5">
  <div>
    <h3 v-click>You can get advantage of all the <strong>nice features</strong> from ZIO CLI you know and love!</h3>
    <ul>
      <li v-click>Rich <strong>documentation</strong> pages</li>
      <li v-click><strong>Discoverability</strong> of functionality</li>
      <li v-click>Inputs <strong>validation</strong></li>
      <li v-click><strong>Auto-completion</strong></li>
      <li v-click><strong>Spelling correction</strong></li>
    </ul>
  </div>
</div>

---
transition: slide-left
layout: image-right
image: /learn.jpg
---

# **To learn more...**

<div class="flex w-full h-3/5 items-center gap-5">
  <div>
    <ul>
      <li>Visit the <strong>ZIO HTTP</strong> <a href="https://github.com/zio/zio-http" target="_blank">GitHub repo</a></li>
      <li>Visit the <strong>ZIO CLI</strong> <a href="https://github.com/zio/zio-cli" target="_blank">GitHub repo</a></li>
      <li>Watch my <a href="https://www.youtube.com/watch?v=0c3zbUq4lQo" target="_blank">talk</a> about ZIO CLI presented at <strong>Functional Scala 2022</strong></li>
    </ul>
  </div>
</div>

---
transition: slide-left
layout: image-right
image: /thanks.jpg
---

# **Special Thanks**

<div class="flex w-full h-3/5 items-center gap-5">
  <div>
    <ul>
      <li><strong>Ziverge</strong> for organizing this conference</li>
      <li><strong>John De Goes</strong> for guidance and support</li>
    </ul>
  </div>
</div>

---
transition: slide-left
layout: image-right
image: /computer.png
---

# **Contact me**

<div class="grid grid-cols-8 gap-4 items-center h-4/5 content-center text-2xl">
  <div class="col-span-1"><img src="/twitterReversed.png" class="w-8" /></div> <div class="col-span-7">@jorvasquez2301</div>
  <div class="col-span-1"><img src="/linkedinReversed.png" class="w-8" /></div> <div class="col-span-7">jorge-vasquez-2301</div>
  <div class="col-span-1"><img src="/email2Reversed.png" class="w-8" /></div> <div class="col-span-7">jorge.vasquez@scalac.io</div>
</div>