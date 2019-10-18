<template>
    <div class="ubtrip-calendar" v-show="initData.showCalendar">

        <div class="custom-header">
            <span @click="back">返回</span>
            <h3 class="header-title">选择日期</h3>
        </div>

        <div v-if="initData.tip" class="ubtrip-calendar-tip">{{initData.tip}}</div>

        <div class="ubtrip-calendar-week clearfix">
            <div class="week-item">日</div>
            <div class="week-item">一</div>
            <div class="week-item">二</div>
            <div class="week-item">三</div>
            <div class="week-item">四</div>
            <div class="week-item">五</div>
            <div class="week-item">六</div>
        </div>
        <div class="ubtrip-calendar-content scroll-view" v-if="dateArray.length">
            <div class="ubtrip-calendar-month clearfix" v-for="item in dateArray">
                <div class="calendar-month-tit">
                    {{item.month}}
                </div>
                <div class="ubtrip-calendar-day" v-for="day in calendarDaysByMonth(item.month)"
                    :class="setCalendarDayClass(day.changeStatus)" @click="handleClickDay(day)">
                    <div class="ubtrip-calendar-number">{{day.isNow ? '今天' : day.dayNum}}</div>
                    <span class="ubtrip-calendar-text"
                        v-if="day.changeStatus == 1 || day.changeStatus == 2">{{day.text}}</span>
                </div>
            </div>
        </div>
    </div>
</template>
<script type="text/javascript">
    //import lunar from '@/assets/js/lunar'
    const changeStatus = {
        Unable: -1, //今天以前(无法操作)
        Normal: 0, // 正常可选状态
        StartSelected: 1, //出发日期选中状态
        ReturnSelected: 2, //返程日期选中状态
        Selected: 3, //日期选中状态
    };

    export default {
        data() {
            return {
                dateArray: [], //日期数组
                isFirstClick: true, //是否是第一次点击
                startDate: null, //选中的起始日期
                returnDate: null, //选中的结束日期
                delayTime: 300, //关闭日历延时时间

            }
        },
        props: {
            initData: {
                default: {
                    showCalendar: false, ////展示日历控件(目的是每次展示日历时清除已选择的状态以及样式)
                    monthNum: 2, //多少个月
                    needReturn: true, //单程还是往返
                    type: "airplane", //飞机票，酒店，火车票，差旅计划(默认飞机票)
                    startDate: "", //可点击的开始时间
                    returnDate: "", //可点击的结束日期
                    tip: "", //日历顶部提示
                }
            }

        },
        computed: {
            startText() {
                return this.initData.type == 'hotel' ? '入住' : this.initData.type == 'plan' ? '开始' : '出发';
            },
            returnText() {
                return this.initData.type == 'hotel' ? '离店' : this.initData.type == 'plan' ? '截止' : '返程';
            },

        },

        methods: {
            init() {
                if (!this.dateArray.length) {
                    this.initDateArray();
                }
                this.isFirstClick = true;
                this.dateArray.forEach((monthVal, monthIndex) => {
                    monthVal.days.forEach((dayVal, dayIndex) => {
                        if (dayVal.changeStatus != changeStatus.Unable) {
                            dayVal.changeStatus = changeStatus.Normal;
                        }
                    });
                });

            },
            //生成日历
            initDateArray() {
                console.log(this.initData);
                let _this = this;
                let currentDateMomentFormat = moment().format("YYYY-MM-DD");
                let beginDate = moment(`${currentDateMomentFormat.substr(0,currentDateMomentFormat.length-2)}01`);
                let endDate = moment(beginDate).add(this.initData.monthNum, 'month');
                for (let i = beginDate; i < endDate; i = i.add({
                        days: 1
                    })) {
                    let m = i.format("YYYY年MM月");
                    let fullDate = i.format("YYYY-MM-DD");

                    let dayItem = {
                        day: i.format('YYYY-MM-DD'),
                        dayNum: i.format("DD"),
                        dayFormat: i.format("MM月DD日"),
                        weekday: i.format("dddd"),
                        unix: i.valueOf(),
                        changeStatus: _getChangeStatus(fullDate),
                        isNow: _isNow(fullDate),
                    };
                    //判断是否是新的月份，如果是的话，则另外创建数组
                    let hasDay = this.dateArray.some(d => d.month == m);
                    if (!hasDay) {
                        let firstWeek = i.format("dddd");
                        this.dateArray.push({
                            month: m,
                            days: [dayItem],
                            firstWeek,
                        });

                    } else {
                        let temp = this.dateArray.find(d => d.month == m);
                        temp.days.push(dayItem);
                    }

                }

                // console.log('生成的日历===========', this.dateArray);

                function _isNow(fullDate) {
                    if (currentDateMomentFormat == fullDate) return true;
                    return false;
                }

                function _getChangeStatus(fullDate) {
                    if ((_this.initData.startDate && fullDate < _this.initData.startDate) || (_this.initData
                            .returnDate && fullDate > _this.initData.returnDate)) {
                        return changeStatus.Unable;
                    } else if (!_this.initData.startDate && fullDate < currentDateMomentFormat) {
                        return changeStatus.Unable;
                    }
                    return changeStatus.Normal;
                }
            },
            calendarDaysByMonth(month) {
                let tempMonth = this.dateArray.find(m => m.month == month);
                if (!tempMonth) return [];
                let spaceDays = _spaceDayArray(tempMonth.firstWeek);

                return [...spaceDays, ...tempMonth.days];

                function _spaceDayArray(week) {
                    let spaceDays = [];
                    switch (week) {
                        case "周六":
                            spaceDays.push('');
                        case "周五":
                            spaceDays.push('');
                        case "周四":
                            spaceDays.push('');
                        case "周三":
                            spaceDays.push('');
                        case "周二":
                            spaceDays.push('');
                        case "周一":
                            spaceDays.push('');
                    }
                    return spaceDays;
                }

            },
            handleClickDay(dayItem) {
                //无用日期和置灰日期不能点击
                if (dayItem == '' || dayItem.changeStatus == changeStatus.Unable) {
                    return;
                }

                console.log(dayItem);
                if (this.initData.needReturn) {

                    if (this.isFirstClick) {
                        this.isFirstClick = false;
                        this.setStartDate(dayItem)
                        console.log(`请选择${this.returnText}日期`);
                    } else {
                        //如果存在出发日期时，选择返程日期需要判断
                        if (this.startDate.unix > dayItem.unix) {
                            console.log(`${this.returnText}日期不能早于${this.startText}日期`);
                        } else if (this.startDate.unix < dayItem.unix) {
                            this.setReturnDate(dayItem);
                            //设置起始,结束日期之间日期的样式
                            this.setDuringDate(this.startDate, dayItem);
                            this.closeCalendar();

                        }

                    }

                } else {
                    this.setStartDate(dayItem);
                    this.closeCalendar();
                }

            },
            //延时关闭日期
            closeCalendar() {
                setTimeout(() => {
                    console.log(2222222);
                    if (this._events["closeCalendar"]) {
                        this.initData.showCalendar = false;
                        console.log(111111);
                        this.$emit("closeCalendar", {
                            startDate: this.getDayItem(this.startDate),
                            returnDate: this.getDayItem(this.returnDate),
                        });
                    }
                }, this.delayTime);
            },
            getDayItem(item) {
                if (!item) {
                    return;
                }
                return {
                    day: item.day,
                    dayFormat: item.dayFormat,
                    weekday: item.weekday,
                    unix: item.unix
                }
            },
            setStartDate(newItem) {
                newItem.changeStatus = changeStatus.StartSelected;
                newItem.text = this.startText;
                this.startDate = newItem;
            },
            setReturnDate(newItem) {
                newItem.changeStatus = changeStatus.ReturnSelected;
                newItem.text = this.returnText;
                this.returnDate = newItem;
            },

            setDuringDate() {

                this.dateArray.forEach((monthVal, monthIndex) => {
                    monthVal.days.forEach((dayVal, dayIndex) => {
                        if (dayVal.unix > this.startDate.unix && dayVal.unix < this.returnDate.unix) {
                            dayVal.changeStatus = changeStatus.Selected;
                        }
                    });
                });

            },

            setCalendarDayClass(status) {
                return status == changeStatus.Unable ? 'before-status' : status == changeStatus.StartSelected ?
                    'start' : status == changeStatus.ReturnSelected ? 'return' : status == changeStatus.Selected ?
                    'selected' : '';

            },
            back() {
                this.initData.showCalendar = false;

            },

        },

        watch: {
            'initData.showCalendar': function () {
                if (this.initData.showCalendar) {
                    this.init();
                }
            }
        }
    }
</script>

<style lang="scss" scoped>
    $daySize:0.88rem;
    $dayHeight:1.4rem;

    .custom-header {
        height: 1.1481rem;
        align-items: center;
        background-color: #26a2ff;
        box-sizing: border-box;
        color: #fff;
        display: flex;
        font-size: 14px;
        line-height: 1;
        padding: 0 10px;
        position: relative;
        text-align: center;
        white-space: nowrap;

        .header-title {
            flex: 2.5;
        }
    }

    .scroll-view {
        overflow-x: hidden;
        overflow-y: auto;
    }


    .ubtrip-calendar {
        position: fixed;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        background-color: #fff;
        z-index: 999;

        .ubtrip-calendar-tip {
            line-height: $daySize;
            background-color: #ffffcc;
            color: #666;
            padding: 0 0.3rem;
        }

        .ubtrip-calendar-week {
            width: 100%;
            color: #333;
            background-color: #F5F5F5;

            .week-item {
                float: left;
                width: (100%/7);
                height: $daySize;
                line-height: $daySize;
                font-size: 14px;
                text-align: center;
            }
        }

        .ubtrip-calendar-content {
            position: absolute;
            top: $daySize * 2 + 1.1481rem;
            width: 100%;
            bottom: 0;
            background-color: #f5f5f5;
        }

        .ubtrip-calendar-content .ubtrip-calendar-month {
            background-color: #fff;

            &.hide {
                visibility: hidden;
            }

            .calendar-month-tit {
                position: relative;
                width: 100%;
                height: 1.2407rem;
                line-height: 1.2407rem;
                text-align: center;
                font-size: 14px;
                color: #333;
            }
        }

        .ubtrip-calendar-content .ubtrip-calendar-day {
            position: relative;
            float: left;
            width: (100%/7);
            height: $dayHeight;
            font-size: 14px;
            text-align: center;
            color: #221714;

            .ubtrip-calendar-number {
                position: absolute;
                top: 0;
                bottom: 0;
                left: 0;
                right: 0;
                margin: auto;
                width: $daySize;
                height: $daySize;
                line-height: $daySize;
            }

            &.before-status {
                color: #999;
            }

            &.selected {
                background-color: #b5caf8;
                color: #fff;
            }

            &.start,
            &.return {
                background-color: #4b7ff5;
                color: #fff;

                .ubtrip-calendar-number {
                    margin: 0 auto;
                    height: $dayHeight/2;
                    line-height: $dayHeight/2;
                }

                span {
                    display: block;
                    position: absolute;
                    top: $dayHeight/2;
                    width: 100%;
                    font-size: 12px;
                }
            }
        }
    }
</style>