<template>
    <my-page title="记录详情" :page="page" backable>
        <div class="common-container">
            <ui-article>
                <h2>{{ record.content }}</h2>
                <p>备注：{{ record.note || '无' }}</p>
                <p>记录值：{{ record.value || 0 }}</p>
                <p>积分：{{ record.score || 0 }}</p>
                <p>开始时间：{{ record.startTime }}</p>
                <p v-if="record.duration">持续时间：{{ record.duration }} 秒</p>
                <p v-if="record.duration">结束时间：{{ record.endTime }}</p>

                <h2>统计</h2>
                <p>今天：{{ stat.today }}</p>
            </ui-article>
        </div>
    </my-page>
</template>

<script>
    /* eslint-disable */
    const moment = window.moment

    export default {
        data () {
            return {
                record: {
                    content: '',
                    note: '',
                },
                stat: {
                    today: 0
                },
                minute: null,
                endTime: '',
                page: {
                    menu: [
                        {
                            type: 'icon',
                            icon: 'edit',
                            click: this.edit,
                            title: '编辑'
                        },
                        {
                            type: 'icon',
                            icon: 'delete',
                            click: this.remove,
                            title: '删除'
                        },
                        {
                            type: 'icon',
                            icon: 'first_page',
                            click: this.prev,
                            title: '上一个记录'
                        },
                                                {
                            type: 'icon',
                            icon: 'last_page',
                            click: this.next,
                            title: '下一个记录'
                        },
                    ]
                }
            }
        },
        mounted() {
            let id = this.$route.params.id
            this.$http.get(`/records/${id}`).then(
                response => {
                    let data = response.data
                    console.log(data)
                    this.record = data
                    this.record.startTime = moment(this.record.startTime).format('YYYY-MM-DD HH:mm:ss')
                    this.record.endTime = moment(this.record.endTime).format('YYYY-MM-DD HH:mm:ss')
                    this.endTime = this.record.endTime

                    // this.$router.go(-1)
                    this.$http.get(`/users/1/records?page_size=9999&date=&content=${this.record.content}`).then(
                        response => {
                            let data = response.data
                            console.log(data)
                            this.stat.today = data.length
                            // this.records = data
                            // if (this.records.length) {
                            //     this.latestRecord = this.records[this.records.length - 1]
                            // }
                        },
                        response => {
                            console.log(response)
                            this.loading = false
                        })
                },
                response => {
                    console.log(response)
                    this.loading = false
                })
            document.addEventListener('keydown', this._onKeyDown = e => {
                console.log(e.keyCode)
                if (e.keyCode === 69) {
                    this.edit()
                }
                switch (e.keyCode) {
                    case 69: // e
                        this.edit()
                        return
                    case 8:
                        this.remove()
                        return
                    case 37:
                        this.prev()
                        return
                    case 39:
                        this.next()
                        return
                }
                // if (e.keyCode === 69) {
                //     this.edit()
                // }
            })
        },
        destroyed() {
            clearInterval(this.timer)
            document.removeEventListener('keydown', this._onKeyDown)
        },
        methods: {
            edit() {
                this.$router.push(`/records/${this.$route.params.id}/edit`)
            },
            finish() {
                if (!this.record.content) {
                    this.$message({
                        type: 'danger',
                        text: '请输入内容'
                    })
                    return
                }
                this.$http.put(`/records/${this.record.id}`, {
                    ...this.record
                    // content: this.content
                }).then(
                    response => {
                        let data = response.data
                        console.log(data)
                        this.$router.go(-1)
                    },
                    response => {
                        console.log(response)
                        this.loading = false
                    })
            },
            endNow() {
                this.record.endTime = moment().format('YYYY-MM-DD HH:mm:ss')
                this.record.duration = (moment(this.record.endTime).toDate().getTime() - moment(this.record.startTime).toDate().getTime()) / 1000
            },
            quickSetting() {
                if (!this.minute) {
                    this.$message({
                        type: 'danger',
                        text: '请输入分钟'
                    })
                    return
                }
                this.record.duration = this.minute * 60
                this.record.endTime = moment(this.record.startTime).add(this.record.duration, 's').format('YYYY-MM-DD HH:mm:ss')
            },
            quickSetting2() {
                if (!this.endTime) {
                    this.$message({
                        type: 'danger',
                        text: '请输入结束时间'
                    })
                    return
                }
                this.record.endTime = this.endTime
                this.record.duration = (moment(this.record.endTime).toDate().getTime() - moment(this.record.startTime).toDate().getTime()) / 1000
                // this.record.endTime = moment(this.record.startTime).add(this.record.duration, 's').format('YYYY-MM-DD HH:mm:ss')
            },
            remove() {
                let ret = window.confirm(`确认删除 ${this.record.content}？`)
                if (!ret) {
                    return
                }
                this.$http.delete(`/records/${this.record.id}`).then(
                    response => {
                        // let data = response.data
                        this.$router.go(-1)
                    },
                    response => {
                        console.log(response)
                        this.loading = false
                    })
            },
            stat() {
                this.$router.push(`/record/stat?content=${this.record.content}`)
            },
            prev() {
                this.$http.get(`/records/${this.record.id}/prev`).then(
                    response => {
                        let data = response.data
                        console.log('上一个', data)
                        if (!data) {
                            this.$message({
                                type: 'danger',
                                text: '没有上一个记录'
                            })
                            return
                        }
                        // // this.$router.go(-1)
                        // this.record.startTime = moment(data.endTime).format('YYYY-MM-DD HH:mm:ss')
                        // this.record.duration = (moment(this.record.endTime).toDate().getTime() - moment(this.record.startTime).toDate().getTime()) / 1000
                        this.$router.replace(`/records/${data.id}`)
                    },
                    response => {
                        console.log(response)
                        this.loading = false
                    })
            },
            next() {
                this.$http.get(`/records/${this.record.id}/next`).then(
                    response => {
                        let data = response.data
                        console.log('下一个', data)
                        if (!data) {
                            this.$message({
                                type: 'danger',
                                text: '没有下一个记录'
                            })
                            return
                        }
                        this.$router.replace(`/records/${data.id}`)
                    },
                    response => {
                        console.log(response)
                        this.loading = false
                    })
            }
        }
    }
</script>

<style lang="scss" scoped>
.record-list {
    .item {
        display: flex;
        justify-content: space-between;
        max-width: 400px;
        margin: 0 auto;
        padding: 16px 0;
        border-bottom: 1px solid rgba(0, 0, 0, .12);
    }
    .title {
        font-size: 16px;
        font-weight: bold;
    }
    .times {
        color: #666;
    }
}
.float-button {
    position: fixed;
    right: 24px;
    bottom: 24px;
}
</style>
