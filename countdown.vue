<template>
	<div class='m-countdown flex'>
        <div class="countdown-item flex" v-for="(item,index) in timeItems" :key="index">
            <div class="num">{{item.num}}</div>
            <div class="symbol">{{item.symbol}}</div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'count-down',
    props: {
        countdown: Number,
        format: String,
    },
	data () {
		return {
            _timer: null,
            endTime: null,
            timeItems: null
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
            this._timer = clearTimeout(this._timer)
        },
        /**
         * 恢复倒计时
         */
        resume () {
            if (this.format && this.endTime && !this._timer) {
                let endTime = this.endTime
                let format = this.format
                this.initCountdown(endTime, format)
            }
        },

        /**
         * 重置倒计时
         */
        reset() {
            if (!this.countdown&&!this.format) {
                return
            }
            this.setUp()
        },

        /**
         * 启动倒计时
         */
        setUp() {
            // 单个倒计时
            let { countdown, format } = this
            // 时间戳，单位毫秒
            let timeStamp = countdown * 1000
            // 缓存结束时间
            let now = new Date().getTime()
            let endTime = now + timeStamp
            this.endTime = endTime
            this.format = format
            this.initCountdown(endTime, format)
        },
        /**
         * 初始化倒计时
         * @param  {Number} endTime 结束时间
         * @param  {String} format 选项
         */
        initCountdown(endTime, format) {
            clearTimeout(this._timer)
            let now = new Date().getTime()
            let timeStamp = endTime - now
            let timeout = timeStamp % 1000 || 0
            this._timer = setTimeout(() => {
                this.initCountdown(endTime, format)
            }, timeout)
            this.setCountdownTimeItems(timeStamp, format)
        },
        /**
         * 设置倒计时渲染数据
         */
        setCountdownTimeItems(timeStamp, format) {
            let showTime = parseInt(Math.ceil(timeStamp / 1000));
            if (showTime <= 0) {
                clearTimeout(this._timer)
            }
            let timeItems = this.getTimeItems(showTime, format)
            this.timeItems = timeItems
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
