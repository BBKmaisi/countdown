<template>
    <div class="m-countdown-list flex" v-if="countdownList&&countdownList.length">
        <div class="countdown-item flex" v-for="(item, cIndex) in timeItemsList[index]" :key="cIndex" >
            <div>{{item.num}}</div>
            <div>{{item.symbol}}</div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'count-down-list',
    props: {
        countdownList: Array,
        index: Number
    },
	data () {
		return {
            _listTimer: null,
            maxEndTime: null,
            showTimeList: [],
            timeItemsList: []
        }
    },
    /**
     * 监听页面显示
     */
    onShow() {
        this.resume();
    },

    /**
     * 监听页面隐藏
     */
    onHide() {
        this.stop();
    },

    /**
     * 监听页面卸载
     */
    onUnload() {
        this.stop();
    },

    mounted(){
        this.setUp();
    },

	methods: {
        /**
         * 暂停倒计时
         */
        stop() {
            this._listTimer = clearTimeout(this._listTimer)
        },
        /**
         * 恢复倒计时
         */
        resume () {
            if (this.showTimeList && this.showTimeList.length > 0 && !this._listTimer) {
                let maxEndTime = this.maxEndTime
                this.initCountdown(maxEndTime)
            }
        },

        /**
         * 重置倒计时
         */
        reset() {
            if (!this.countdownList&&!this.countdownList.length) {
                return
            }
            this.setUp()
        },

        /**
         * 启动倒计时
         */
        setUp() {
            // 缓存结束时间
            let maxEndTime = 0;
            let showTimeList = []
            this.countdownList&&this.countdownList.map((item, index) => {
                let countdown = Number(item.endTime) || 0;
                let format = item.format;
                let endTime = countdown * 1000;
                if (endTime > maxEndTime) {
                    maxEndTime = endTime;
                }
                showTimeList.push({
                    format,
                    endTime
                })
            })
            this.showTimeList = showTimeList
            this.maxEndTime = maxEndTime
            this.initCountdown(maxEndTime)
        },
        /**
         * 初始化倒计时
         * @param  {Number} endTime 结束时间
         * @param  {String} format 选项
         */
        initCountdown(maxEndTime) {
            clearTimeout(this._listTimer);
            let now = new Date().getTime();
            let timeStamp = maxEndTime - now;
            let timeout = timeStamp % 1000 || 0;
            this._listTimer = setTimeout(() => {
                this.initCountdown(maxEndTime)
            }, timeout);
            this.setCountdownTime();
        },
        /**
         * 设置倒计时渲染数据
         */
        setCountdownTime() {
            let timeItemsList = [];
            let zeroCount = 0;
            let now = new Date().getTime();
            this.showTimeList&&this.showTimeList.forEach((showTimeValue) => {
                let format = showTimeValue.format;
                let endTime = showTimeValue.endTime;
                let showTime = parseInt(Math.ceil((endTime - now) / 1000));
                if (showTime <= 0) {
                    showTime = 0;
                    zeroCount ++;
                }
                let timeItems = this.getTimeItems(showTime, format);
                timeItemsList.push(timeItems);
            })
            if (this.showTimeList.length === zeroCount) {
                clearTimeout(this._listTimer);
            }
            this.timeItemsList = timeItemsList
        },
        /**
         * 获取渲染所需数据
         * @param  {Number} showTime 时间戳
         * @param  {String} format   格式
         * @return {Array}          渲染所需数组
         */
        getTimeItems(showTime, format) {
            if (showTime < 0) {
                showTime = 0
            }
            let data = []
            let arr = format.match(/[a-zA-Z]{1,3}/g)
            let symbolArr = format.match(/[\u4e00-\u9fa5]+|[^a-zA-Z]/g) || []
            let time = this.getTime(showTime, format)
            arr.map((item, i) => {
                data.push({
                    num: time[item],
                    symbol: symbolArr[i]
                })
            })
            return data;
        },
        /**
         * 获取格式化时间
         * @param  {Number} leftseconds 剩余时间
         * @param  {String} format      格式
         * @return {Object}             格式化映射对象
         */
        getTime(leftseconds, format) {
            let d = Math.floor(leftseconds / (60 * 60 * 24))
            let h = Math.floor((leftseconds - d * 24 * 60 * 60) / 3600)
            let m = Math.floor((leftseconds - (d * 24 * 60 + h * 60) * 60) / 60)
            let s = Math.floor(leftseconds - (d * 24 * 60 + h * 60 + m) * 60)
            if (leftseconds > 86400 && format.indexOf('d') === -1) {
                h += d * 24;
            }
            if (leftseconds > 3600 && format.indexOf('h') === -1) {
                m += h * 60;
            }
            if (leftseconds > 60 && format.indexOf('m') === -1) {
                s += m * 60;
            }
            return {
                dd: this.formatTime(d),
                hh: this.formatTime(h),
                mm: this.formatTime(m),
                ss: this.formatTime(s),
                d,
                h,
                m,
                s
            }
        },
        /**
         * 补全数字
         * @param  {Number} val 数字
         * @return {String}     数字字符串
         */
        formatTime(val) {
            return val < 10 ? `0${val}` : val
        }
    }
}
</script>

<style lang="less" scoped>
.num{
    font-weight: 500;
}
.symbol{
    margin: 0 4px;
}
</style>
