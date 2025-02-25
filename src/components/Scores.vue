<template>
    <div style="width: 100%">
        <LvProgressBar :value="redScore" :showValue="false" color="red" />
        <br />
        <LvProgressBar :value="blueScore" :showValue="false" color="blue" />
    </div>
</template>

<script>
import LvProgressBar from 'lightvue/progress-bar';

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