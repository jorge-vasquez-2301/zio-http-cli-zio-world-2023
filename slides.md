---
info: Slides for presentation at ZIO World 2023
theme: 'default'
presenter: true
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

```scala {1|2-5|7-10|12|13-14|16|17-21|23|24-28} {maxHeight:'400px'}
// Domain models
final case class User(id: Int, name: String, email: String)
object User {
  implicit val schema = DeriveSchema.gen[User]
}

final case class Post(userId: Int, postId: Int, contents: String)
object Post {
  implicit val schema = DeriveSchema.gen[Post]
}

// Endpoint to get a User by ID
val getUser =
  Endpoint.get("users" / int("userId")).header(HeaderCodec.location).out[User]

// Endpoint to get a User's posts by userId and postId
val getUserPosts =
  Endpoint
    .get("users" / int("userId") / "posts" / int("postId"))
    .query(query("name"))
    .out[List[Post]]

// Endpoint to create a new User
val createUser =
  Endpoint
    .post("users")
    .in[User]
    .out[String]
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
    _        <- executor(getUser(42, Location.toLocation("some-location"))).debug("result1") 
    _        <- executor(getUserPosts(42, 200, "adam")).debug("result2")
    _        <- executor(createUser(User(2, "john", "john@test.com"))).debug("result3")
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

```bash {1-4|5|7-10|11-22|19|24|25-29|28|30} {maxHeight:'400px'}
$ curl --request POST 'http://localhost:8080/users' --header 'Content-Type: application/json' --data-raw '{
    "id": 2,
    "name": "Test"
}'
# Got nothing 🤔

$ curl --request POST 'http://localhost:8080/users' --header 'Content-Type: application/json' --data-raw '{
    "id": 2,
    "name": "Test"
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

# **Conclusion**

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
layout: image
image: /laptop.jpg
class: "flex h-screen justify-end items-center"
---

# Solution

<style>
h1 {
  @apply text-6xl !important
}
</style>

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
image: /thanks.jpg
---

# **Special Thanks**

<v-clicks>

* **Ziverge** for organizing this conference
* **John De Goes** for guidance and support

</v-clicks>

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