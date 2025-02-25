<template>
    <div class="flex p-4 space-x-10">
        <div class="flex flex-col w-1/2 space-y-2">
            <p class="w-max bg-gradient-to-tr from-red-700 to-red-500 bg-clip-text text-transparent text-6xl font-bold font-k2d">{{ trueScores.redScore }}</p>
            <LvProgressBar class="!h-4 !bg-red-100 !rounded-md shadow-uniform shadow-red-700" :value="redScore" :showValue="false" color="#c10007" />
        </div>
        <div class="flex flex-col w-1/2 space-y-2">
            <p class="w-max bg-gradient-to-tr from-blue-700 to-blue-500 bg-clip-text text-transparent text-6xl font-bold font-k2d self-end">{{ trueScores.blueScore }}</p>
            <LvProgressBar class="!h-4 !bg-blue-100 !rounded-md rotate-180 shadow-uniform shadow-blue-700" :value="blueScore" :showValue="false" color="#1447e6" />
        </div>
    </div>
</template>

<script>
import LvProgressBar from 'lightvue/progress-bar'

export default {
    data() {
        return {
            redScore: 0,
            blueScore: 0,
            trueScores: {
                redScore: 0,
                blueScore: 0
            }
        }
    },
    interval: null,
    methods: {
        endProgress() {
            clearInterval(this.interval);
            this.interval = null;
        },
        updateValue(newValues){
            console.log(newValues);
            
            this.trueScores.redScore = newValues.score_red;
            this.trueScores.blueScore = newValues.score_blue;
            this.redScore = (newValues.score_red / 300) * 100;
            this.blueScore = (newValues.score_blue / 300) * 100;
        }
    },
    mounted() {
    },
    beforeDestroy() {
        this.endProgress();
    },
    components: {
        LvProgressBar: LvProgressBar,
    }
}
</script>

<style>

</style>