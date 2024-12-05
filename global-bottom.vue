<template>
    <footer v-if="$nav.currentPage !== 1 && $nav.hasNext" class="absolute bottom-0 left-6 right-0 p-2 z-50">
        <SlideCurrentNo /> /
        <SlidesTotal />
    </footer>

    <footer v-if="$nav.currentPage !== 1 && $nav.hasNext && $nav.currentLayout !== 'image-right'" class="absolute bottom-0 right-6 p-2 z-50">
        <LightOrDark>
            <template #dark>
                <img src="/functionalScalaIconWhite.png" class="h-10" />
            </template>
            <template #light>
                <img src="/functionalScalaIcon.png" class="h-10" />
            </template>
        </LightOrDark>
    </footer>

    <footer v-if="$nav.currentPage !== 1 && $nav.hasNext && $nav.currentLayout !== 'image-right'" class="absolute bottom-0 z-50 flex w-full justify-center items-start text-[8px] mb-4">
        <p>Scala's Hidden Treasures: Five ZIO-Compatible Libraries you didn't know you needed! for Functional Scala 2024</p>
    </footer>
</template>