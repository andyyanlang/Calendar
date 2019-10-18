<template>
    <div class="scroll-view main-view index-view">
        <mt-tabbar v-model="selected" fixed>
            <mt-tab-item id="home">
                <i class="iconfont icon-home" slot="icon"></i>首页
            </mt-tab-item>
            <mt-tab-item id="pay" v-alert>
                <i class="iconfont icon-weixinpay" slot="icon"></i>微信付款
            </mt-tab-item>
            <mt-tab-item @click.native="goMyInfo">
                <i class="iconfont icon-people1" slot="icon"></i>我的商旅
            </mt-tab-item>
        </mt-tabbar>
        <mt-tab-container v-model="selected">
            <mt-tab-container-item class="home-view" id="home">
                <img class="home-bg" id="home-bg"  alt="" v-if="locationVersion">
                <img class="home-bg" id="home-bg" :src="require('@/assets/img/yun.png')" alt="" v-else>
                <div class="home-container">
                    <img class="home-logo" id="home-logo" alt="" @click="showIndexAddress" v-if="locationVersion" />
                    <img class="home-logo" id="home-logo" :src="require('@/assets/img/index_logo.png')" alt="" @click="showIndexAddress" v-else />
                    <p class="welcome-tip" v-show="userEmployee.Name">欢迎您，{{userEmployee.Name}}</p>
                    <transition name="fade">
                        <div class="recent-trip" v-show="complete">
                            <div class="common-details-view index-trip-view" v-if="LatestTripType === 'HomeTicket'">
                                <div class="trip-top">
                                    <span>
                                        <i class="iconfont icon-clock"></i>{{LatestTrip.FlightDate}} {{LatestTrip.Week}}</span>
                                    <span class="flight-name">{{LatestTrip.AirCorpName}} {{LatestTrip.FlightCode}}</span>
                                </div>
                                <div class="details-center" @click="goAirplaneDetails">
                                    <div class="details-side-content">
                                        <div class="details-time">{{LatestTrip.STime}}</div>
                                        <div class="details-small">{{LatestTrip.SCityName}}{{LatestTrip.STerminal}}</div>
                                    </div>

                                    <div class="details-center-content">
                                        <!-- <p>{{LatestTrip.TravelTimeSpan}}</p> -->
                                        <i class="iconfont icon-feiji8"></i>
                                        <p>{{LatestTrip.StopCity ? `经停 ${LatestTrip.StopCity}` : "直飞"}}</p>
                                    </div>

                                    <div class="details-side-content">
                                        <div class="details-time">{{LatestTrip.ETime}}</div>
                                        <div class="details-small">{{LatestTrip.ECityName}}{{LatestTrip.ETerminal}}</div>
                                    </div>
                                </div>

                                <div class="trip-bottom">
                                    <!-- <img :src="this.LatestTrip.AirCorpCode | toFlightLogo" alt="" class="flight-logo" /> -->
                                    <div class="check-in" v-if="LatestTrip.ShowNetCheckButton" @click="checkIn">
                                        <i class="iconfont icon-cangwei"></i>
                                        <span>代办值机</span>
                                    </div>
                                </div>
                            </div>

                            <div class="common-details-view index-trip-view" v-if="LatestTripType == 'Train'">
                                <div class="trip-top">
                                    <i class="iconfont icon-clock"></i>
                                    {{LatestTrip.SDate}} {{LatestTrip.Week}} {{LatestTrip.SeatName}}
                                </div>
                                <div class="details-center" @click="goTrainDetails">
                                    <div class="details-side-content">
                                        <div class="details-time">{{LatestTrip.STime}}</div>
                                        <div class="details-small">{{LatestTrip.SStationName}}</div>
                                    </div>

                                    <div class="details-center-content">
                                        <p>{{LatestTrip.TravelTimeSpan}}</p>
                                        <img :src="require('@/assets/img/train-line1.png')" class="details-line-bg" />
                                        <p>{{LatestTrip.TrainNumber}}</p>
                                    </div>

                                    <div class="details-side-content">
                                        <div class="details-time">
                                            <span class="end-time">{{LatestTrip.ETime}}
                                                <i class="end-time-tag" v-if="LatestTrip.AddTravelDays > 0">+{{LatestTrip.AddTravelDays}}天</i>
                                            </span>
                                        </div>
                                        <div class="details-small">{{LatestTrip.EStationName}}</div>
                                    </div>
                                </div>
                                <div class="trip-bottom"></div>
                            </div>
                        </div>
                    </transition>

                    <div class="entry-container">
                        <div class="entry-item">
                            <router-link class="entry-item-wrapper" to="/airline/index">
                                <img :src="require('@/assets/img/airplane-circle.png')">
                                <span>国内机票</span>
                            </router-link>
                        </div>
                        <div class="entry-item">
                            <router-link class="entry-item-wrapper" to="/interAirline/index">
                                <img :src="require('@/assets/img/interAirplane-circle.png')">
                                <span>国际机票</span>
                            </router-link>
                        </div>
                        <div class="entry-item">
                            <router-link class="entry-item-wrapper" to="/wineshop/index">
                                <img :src="require('@/assets/img/hotel-circle.png')" />
                                <span>国内酒店</span>
                            </router-link>
                        </div>
                        <div class="entry-item">
                            <a class="entry-item-wrapper" @click="goTrain">
                                <img :src="require('@/assets/img/train-circle.png')" />
                                <span>火车票</span>
                            </a>
                        </div>
                        <div class="entry-item" v-if="showTripPlan">
                            <a class="entry-item-wrapper" @click="goPlan">
                                <img :src="require('@/assets/img/plan-circle.png')" />
                                <span>差旅计划</span>
                            </a>
                        </div>
                        <div class="entry-item">
                            <a class="entry-item-wrapper" @click="goApprove">
                                <img :src="require('@/assets/img/approve-circle.png')" />
                                <span>审批授权</span>
                                <em v-if="approveNum">{{approveNum}}</em>
                            </a>
                        </div>
                    </div>

                </div>

            </mt-tab-container-item>
            <mt-tab-container-item class="weixin-pay-view" id="pay">
                <img id="wx-pay" v-if="locationVersion" />
                <img id="wx-pay" :src="require('@/assets/img/weixin-pay.jpg')" v-else />
            </mt-tab-container-item>
        </mt-tab-container>
        <i class="mintui mintui-back back-history" @click="goHistory" v-if="backurl !== ''"></i>
    </div>
</template>

<script>
    window.getHomeImgUrls = function (res){
        var HomeImgUrls = sessionStorage.getItem('Ubtrip_homeImgUrls');
        if (!HomeImgUrls || HomeImgUrls === '{}') {
           sessionStorage.setItem('Ubtrip_homeImgUrls', JSON.stringify(res));
        }
        const {HomeLogo, WxPay, Yun} = res;
        $('#home-logo').attr('src', HomeLogo);
        $('#wx-pay').attr('src', WxPay);
        $('#home-bg').attr('src', Yun);
        // document.getElementById("img_id").src = res
        // document.getElementById("img_id").width = 200

    }
    import { Base64 } from 'js-base64';
    import { mapMutations, mapState, mapActions } from 'vuex';
    export default {
        name: 'index',
        data() {
            return {
                selected: "home",
                complete: false,
                LatestTrip: {}, //行程提醒对象
                LatestTripType: "", //当前展示的行程类型
                swtichCount: 0,
                approveNum: 0, //待审批授权提醒数量

            }
        },
        computed: {
            ...mapState('base', ['userInfo', 'userEmployee', 'backurl', 'IsOfferTrainServer', 'showTripPlan', 'isFromApp', 'appInfo']),
            locationVersion() {
                return this.appInfo.version >= 7.6;
            },
        },
        methods: {
            ...mapMutations('base', {
                baseChange: 'changeStaticState'
            }),
            ...mapMutations('airline', {
                airlineChange: 'changeStaticState'
            }),
            ...mapMutations('train', {
                trainChange: 'changeStaticState'
            }),
            ...mapActions('base', ['wxSignUp', 'autoSignUp']),
            refreshIsApprovalLogin() {
                this.baseChange({
                    name: 'isApprovalLogin',
                    newVal: false
                })
            },
            async loginWx(OpenId) {
                const params_LoginForOpenId = {
                    OpenId,
                    showLoading: false
                }
                this.$indicator.open('加载中...');
                await this.wxSignUp(params_LoginForOpenId).then(() => {
                    this.$indicator.close();
                    return Promise.resolve();
                }).catch(() => {
                    this.$indicator.close();
                    return Promise.reject();
                });
            },

            goMyInfo() {
                this.$router.push({
                    name: "profile.index"
                });
            },
            goHistory() {
                let backurl = decodeURIComponent(decodeURIComponent(this.backurl));
                window.location.href = backurl.indexOf('//:') > -1 ? backurl : 'http://' + backurl;
            },
            goTrain() {
                if (this.userInfo.Id === undefined) {
                    let history = this.$route.name;
                    this.$toast('请先登录');
                    return this.$router.replace({ name: 'login', params: { showTransition: 'go', history } });
                }
                if (this.IsOfferTrainServer) {
                    return this.$router.push({ name: 'train.index' });
                }
                this.$alert('暂不提供火车票预订服务，如有疑问，请联系商旅管理员');
            },
            goPlan() {
                if (this.userInfo.Id === undefined) {
                    let history = this.$route.name;
                    this.$toast('请先登录');
                    return this.$router.replace({ name: 'login', params: { showTransition: 'go', history } });
                }
                this.$store.commit('approve/changeStaticState', {
                    name: "travelPlanIndex",
                    newVal: 0
                });
                return this.$router.push({ name: 'travelPlan.index' });
            },
            goApprove() {
                this.$store.commit('approve/changeStaticState', {
                    name: "approveIndex",
                    newVal: 0
                });
                this.$router.push({ name: 'approve.list' });
            },
            getRecentTrip(res) {
                // this.$axios({
                //     url: "/GetLatestTrip",
                // }).then(res => {
                    this.complete = true;
                    this.LatestTripType = res.data.LatestTripResponse.LatestTripType;
                    // this.LatestTripType = res.data.LatestTripType;
                    if (!this.LatestTripType) {
                        return;
                    }
                    if (this.LatestTripType == "HomeTicket") {
                        this.LatestTrip = res.data.LatestTripResponse.TicketTrip;
                        // this.LatestTrip = res.data.TicketTrip;
                        this.LatestTrip.FlightDate = this.LatestTrip.FlightDate ? this.LatestTrip.FlightDate.split("T")[0] : '';
                        let tempTime = "";
                        if (this.LatestTrip.ETime && this.LatestTrip.STime) {
                            let lim = this.$moment(this.LatestTrip.ETime, "HH:mm") - this.$moment(this.LatestTrip.STime, "HH:mm");
                            tempTime = lim > 0 ? lim : lim + 24 * 60 * 60 * 1000;
                        }
                    } else if (this.LatestTripType == "Train") {
                        this.LatestTrip = res.data.LatestTripResponse.TrainTrip;
                        // this.LatestTrip = res.data.TicketTrip;
                        this.LatestTrip.SDate = this.LatestTrip.SDate ? this.LatestTrip.SDate.split("T")[0] : '';
                    }
                // });
            },
            goAirplaneDetails() {
                this.airlineChange({
                    name: "OrderId",
                    newVal: this.LatestTrip.OrderId
                });
                this.$router.push({
                    name: "profile.airplane.orderDetails"
                });
            },
            goTrainDetails() {
                this.trainChange({
                    name: "OrderId",
                    newVal: this.LatestTrip.OrderId
                });
                this.$router.push({
                    name: "profile.train.orderDetails"
                });
            },
            getWaitApprovalCount(res) {
                // this.$axios({
                //     url: "/GetWaitApprovalCount",
                // }).then(res => {
                    this.approveNum = res.data.WaitApprovalCountResponse.ApprovalCount;
                    // this.approveNum = res.data.ApprovalCount;

                //});
            },
            GetLatestTripAndWaitingApprovalNum() {
                this.$axios({'url': '/GetLatestTripAndWaitingApprovalNum'}).then(res => {
                    this.getRecentTrip(res)
                    this.getWaitApprovalCount(res);
                })
            },
            checkIn() {
                //值机页面显示所需字段
                this.airlineChange({
                    name: "orderItemInfo",
                    newVal: [{
                        OrderId: this.LatestTrip.OrderId,
                        PassengerName: this.userEmployee.Name,
                    }]
                });
                this.$router.push({
                    name: "profile.airplane.checkIn",
                    params: { showTransition: 'go' }
                });
            },
            showIndexAddress() {
                if (++this.swtichCount >= 10) {
                    this.$alert(window.location.href + (window.location.href.indexOf('?') > -1 ? '&' : '?') +'H5Version='+process.env.version);
                }
            }

        },
        mounted() {
            var HomeImgUrls = sessionStorage.getItem('Ubtrip_homeImgUrls');
            if (HomeImgUrls && HomeImgUrls !== '{}') {
                getHomeImgUrls(JSON.parse(HomeImgUrls));
            }
            //已登录
            if (typeof this.userInfo.Id != "undefined") {
                this.GetLatestTripAndWaitingApprovalNum()
            }
            //微信内首次进入获取openid
            const { opcode } = this.$route.query;
            if (!!opcode && !sessionStorage.getItem('Ubtrip_openid')) {
                const params_GetAccessToken = {
                    code: Base64.decode(opcode)
                };
                this.$axios({ url: '/GetAccessToken', params: params_GetAccessToken }).then(res => {
                    sessionStorage.setItem('Ubtrip_openid', res.data.accessToken.openid);
                    this.loginWx(res.data.accessToken.openid).then(() => {
                        //手机号与openid绑定过，才会登录成功
                        this.GetLatestTripAndWaitingApprovalNum()
                    });
                });
            }
            //getHomeImgUrls();
            this.refreshIsApprovalLogin();
        },
        beforeRouteEnter(to, from, next) {
            let OpenId = sessionStorage.getItem('Ubtrip_openid');
            next(vm => {
                if (typeof vm.userInfo.Id === "undefined") {
                    //微信内也可以进入登录页面，所以优先判断是否是自动登录。
                    if (localStorage["Ubtrip_LoginCookie"] && localStorage["Ubtrip_LoginEmployeeId"]) {
                        //自动登录
                        vm.autoSignUp().then(() => {
                            vm.GetLatestTripAndWaitingApprovalNum()
                            //vm.$axios({url: '/GetHomeInfo'})
                        });
                    } else if (OpenId) {
                        //微信内手动刷新或者微信内登录超时，因为重新reload了，所以需要自动登录
                        vm.loginWx(OpenId).then(() => {
                            vm.GetLatestTripAndWaitingApprovalNum()
                        });
                    }

                }
                return true;
            });
        },
        components: {}
    }
</script>

<style scoped lang="scss">
    $tabBarHeight:1.6rem;

    .mint-tabbar {
        z-index: 1;
        background: rgba(51, 51, 51, .8);

        .mint-tab-item {
            height: $tabBarHeight;
            padding: 0;
            align-items: center;
            color: #ccc;

            &.is-selected {
                color: #459cfd;
                background-color: inherit;
            }

            .iconfont {
                font-size: 20px;
            }
        }
    }

    .mint-tab-container {
        width: 100%;
        height: 100%;
        background: linear-gradient(rgb(53, 163, 229), rgb(107, 187, 250), rgb(224, 241, 252), white);
    }

    .mint-tab-container-item {
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
    }

    .home-view {
        .home-bg {
            width: 100%;
            position: relative;
            top: -2px;
        }

        .home-container {
            position: absolute;
            top: 0.5rem;
            left: 0;
            right: 0;
            bottom: 0;
        }

        .home-logo {
            display: block;
            width: 5.9rem;
            height: 1.466rem;
            margin: 0 auto;
        }

        .welcome-tip {
            color: #fff;
            text-align: center;
            margin: 0 auto 0.5rem;
        }

        .recent-trip {
            margin: 0 0.3rem;
            background: rgba(0, 0, 0, .4);
            border-radius: 6px;
        }

        .index-trip-view {
            color: #fff;

            .trip-top,
            .trip-bottom {
                padding: 0 0.4rem;
                height: 0.7rem;
                line-height: 0.7rem;
                font-size: 12px;

                .iconfont {
                    position: relative;
                    top: 1.5px;
                    color: #ff6600;
                }
            }

            .flight-name {
                float: right;
            }

            .details-center {
                border-top: 1px solid #5f87a5;
                border-bottom: 1px solid #5f87a5;
                background-color: transparent;

                .iconfont {
                    display: inline-block;
                    margin: 0.1rem 0;
                    font-size: 20px;
                }

                .details-side-content {
                    padding: 0.3rem 0;
                }

                .details-center-content {
                    margin-top: 0.35rem;
                }

                .details-time {
                    .end-time {
                        position: relative;
                    }

                    .end-time-tag {
                        position: absolute;
                        right: -0.8rem;
                        top: -0.05rem;
                        font-size: 10px;
                    }
                }

                .details-line-bg {
                    margin: -0.05rem 0;
                }

                .details-small {
                    color: #fff;
                }
            }

            .check-in {
                float: right;
            }
        }

        .entry-container {
            position: absolute;
            bottom: 2rem;
            left: 0;
            right: 0;
            padding: 0 0.333rem;
        }


        .entry-item {
            width: 33.33%;
            float: left;
            text-align: center;
            margin-bottom: 0.5rem;
            font-size: 12px;

            .entry-item-wrapper {
                width: 1.8rem;
                display: block;
                position: relative;
                margin: 0 auto;
                color: #444;

                img {
                    display: block;
                    width: 100%;
                    height: 1.8rem;
                    margin-bottom: 0.2592rem;
                }

                em {
                    position: absolute;
                    top: 0;
                    right: 0;
                    padding: 2px 6px;
                    text-align: center;
                    background-color: #ff4242;
                    color: #fff;
                    border-radius: 10px;
                }
            }
        }
    }

    .weixin-pay-view {
        bottom: $tabBarHeight;

        img {
            width: 100%;
            height: 100%;
        }
    }

    .back-history {
        position: absolute;
        top: .8rem;
        left: .5rem;
        color: #fff;
        font-size: 22px;
    }
</style>
