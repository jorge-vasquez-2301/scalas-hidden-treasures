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
</template>