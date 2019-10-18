<template>
    <div id="app">
        <div class="open" @click="changeDate">打开日历</div>
        <!-- 日期选择组件 -->
        <v-calendar type="airplane" :initData="initData" @closeCalendar="closeCalendar"></v-calendar>
    </div>
</template>

<script>
    import vCalendar from '@/components/Calendar';
    export default {
        name: 'app',
        data() {
            return {

                initData: {
                    showCalendar: false, //展示日历控件
                    monthNum: 13, //多少个月
                    needReturn: true //单程还是往返
                }

            }
        },
        methods: {
            changeDate() {
                this.initData.showCalendar = true;
            },
            closeCalendar(item) {
                console.log("选中的日期", item);
            },
        },
        components: {
            vCalendar
        },
    }
</script>

<style lang="scss">
    .open {
        width: 2rem;
        height: 1rem;
        line-height: 1rem;
        font-size: 15px;
        margin: auto;
        background-color: aquamarine;
    }
</style>