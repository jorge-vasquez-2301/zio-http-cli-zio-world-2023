<template>
    <footer class="absolute bottom-0 left-10 right-0 p-2 z-50">
        <LightOrDark>
            <template #dark>
                <img src="/scalacLogo.png" class="h-10" />
            </template>
            <template #light>
                <img src="/scalacLogo2.png" class="h-10" />
            </template>
        </LightOrDark>
    </footer>

    <footer class="absolute bottom-0 right-0 p-2 z-50">
        <SlideCurrentNo /> /
        <SlidesTotal />
    </footer>
</template>