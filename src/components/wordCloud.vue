<template>
    <h2 class="text-2xl mb-5 font-bold text-[#40513B]">{{ title }}</h2>
    <section class="border-[#9DC08B] border-8 p-5 my-4">
        <vue-word-cloud style="height: 600px;width: 100%;" :words="words" color="#40513B" :animationDuration="5000"
            animationEasing="ease-in" :animationOverlap="0.5" font-family="Noto Sans TC" :spacing="1"
            enterAnimation="animate__animated animate__fadeIn" leaveAnimation="animate__animated animate__fadeOut" />
    </section>
</template>

<script>
import VueWordCloud from 'vuewordcloud';
export default {
    name: 'wordCloud',
    components: {
        [VueWordCloud.name]: VueWordCloud,
    },
    props: {
        title: '',
        words: []
    },
}
</script>