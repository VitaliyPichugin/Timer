<template>
    <div class="notification">
        <div class="d-flex flex-row justify-content-center align-items-center notification-wrap" v-if="!showModal">
            <img :src=time_out_img >
            <div class="notification-message">
                Complete your payment in
                <span class="font-weight-bold">{{ timer_out | formatted_time }}</span>
                <span class="d-none d-md-inline">to avoid session time out</span>
            </div>
        </div>
        <modal-payment v-if="showModal" @close="closeModal()">
            <div slot="header">
                <img class="img_close img-responsive " @click="closeModal()" :src=close_img />
            </div>
            <div slot="body">
                <div class="d-flex flex-column justify-content-center align-items-center modal-wrap">
                    <div class="d-flex flex-row justify-content-center align-items-center session-expired">
                        <img class="timer" :src=timer_img />
                        <span>Your session expired :(</span>
                    </div>
                    <div class="redirect-message">You will be redirected to the Event Page to select your tickets again.</div>
                    <button type="button" class="btn btn-redirect" @click="closeModal()">Okay</button>
                </div>
            </div>
        </modal-payment>
    </div>
</template>

<script>
    import modalPayment from  './Modal';

    export default {
        components: {
            modalPayment,
        },
        props: ['flag'],
        data() {
            return {
                showModal: false,
                minutes: 0,
                seconds: 0,
                time: 0,
                timer: null,
                t_out: false,
                time_out_img: '/images/time_out.svg',
                close_img: '/images/close.svg',
                timer_img: '/images/timer.svg',
                data: {},
            }
        },
        computed: {
            timer_out() {
                let time = this.time / 60;
                let minutes = parseInt(time);
                let seconds = Math.round((time - minutes) * 60);
                return `${minutes}:${seconds}`;
            }
        },
        methods: {
            start() {
                if (!this.timer) {
                    this.timer = setInterval(() => {
                        if (!this.t_out && this.time > 0) {
                            this.time--;
                        } else {
                            this.stop();
                            this.showModal = true;
                        }
                    }, 1000 );
                }
            },
            stop() {
                clearInterval(this.timer);
                this.timer = null;
                this.time = 0;
                this.seconds = 0;
                this.minutes = 0;
            },
            isAllEvents() {
                if((window.location.href).indexOf('rugby-world-cup-2019') === -1) {
                    return true;
                } else {
                    return false;
                }
            },
            gettingTime(){
                axios.post('/payment/get-time').then((response) => {
                    this.data = response.data;
                    this.t_out = this.data.time.time_out;
                    this.time = (this.data.time.diff_min * 60 + this.data.time.diff_sec);
                    this.start();
                });
            },
            getTime() {
                if (this.flag) {
                    axios.post('/payment/reset-simple', {})
                        .then(() => {
                           this.gettingTime();
                        })
                        .catch((error) => {
                            console.error(error);
                        });
                }else this.gettingTime();
            },
            closeModal() {
                this.showModal = false;
                if (this.isAllEvents()) {
                    axios.post('/payment/reset-time', {})
                        .then(() => {
                            window.location = `/tickets/${window.localStorage.getItem('slug')}`;
                        })
                        .catch((error) => {
                            console.error(error);
                        });
                }
                else {
                    window.location = '/rugby-world-cup-2019';
                }
            }
        },
        filters: {
            formatted_time : (value) => {
                let data = value.split(':');
                let minutes = data[0];
                let seconds = data[1];
                if (minutes < 10) {
                    minutes = `0${minutes}`;
                }
                if (seconds < 10) {
                    seconds = `0${seconds}`;
                }
                return `00:${minutes}:${seconds}`;
            },
        },
        mounted () {
            this.getTime();
        }
    };
</script>
