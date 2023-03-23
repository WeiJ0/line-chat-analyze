<template>
    <div class="w-full md:w-1/2 lg:w-1/4 p-3">
        <div class="bg-white border rounded shadow shadow-[#9DC08B] p-5">
            <div class="flex justify-center flex-row items-center">
                <div class="flex-shrink pr-3">
                    <div class="rounded p-3 bg-[#609966]">
                        <i :class="icon" class="fas fa-2x fa-fw fa-inverse"></i>
                    </div>
                </div>
                <div class="text-center">
                    <h5 class="font-bold uppercase text-gray-500">{{ title }}</h5>
                    <h3 class="font-bold text-4xl">{{ result }} </h3>
                </div>

            </div>
            <ul class=" py-2 px-5">
                <li class="text-left flex items-center my-2" v-for="(item, index) in top">
                    <i :class="{
                        'text-yellow-300': index === 0,
                        'text-gray-300': index === 1,
                        ' text-yellow-700': index === 2
                    }" class="fas fa-2x fa-award mr-3"></i>
                    <span class="text-lg">{{ item.join(": ") }}</span>
                </li>
            </ul>
        </div>
    </div>
</template>

<script>
export default {
    name: 'count',
    props: {
        title: '',
        icon: '',
        result: 0,
        top: [],
    },
}
</script>