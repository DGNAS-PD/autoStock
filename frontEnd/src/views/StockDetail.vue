<template>
    <div>
        <el-row :gutter="20">
            <el-col :span="6">
                <el-button class="button_info" plain @click="prev" icon="el-icon-d-arrow-left" circle></el-button>
            </el-col>
            <el-col :span="6" :offset="3">
                <stock-info :stock="stock" align="center"></stock-info>
            </el-col>
        </el-row>

        <el-tabs v-model="activeName" type="card" style="margin-bottom: 30px;">
            <el-tab-pane name="first" :key="'first'">
                <span slot="label"><i class="el-icon-date"></i> 每日指标</span>
                <daily-list
                        ref="daily_list"
                        :total="daily_total"
                        :daily_basics="daily_basics"
                        @pageChange="daily_pageChange"
                        @sortChange="daily_sortChange"
                ></daily-list>

            </el-tab-pane>
            <el-tab-pane name="second" :key="'second'">
                <span slot="label"><i class="el-icon-date"></i> 北向资金</span>
                <hkhold-list
                        ref="hkhold_list"
                        :total="hkhold_total"
                        :hkholds="hkholds"
                        @pageChange="hkhold_pageChange"
                        @sortChange="hkhold_sortChange"
                ></hkhold-list>


            </el-tab-pane>
            <el-tab-pane name="third" :key="'third'">
                <span slot="label"><i class="el-icon-s-marketing"></i> 分红配股</span>
                <share-list
                        ref="share_list"
                        :total="share_total"
                        :shares="shares"
                        @filterChange="share_filterChange"
                        @pageChange="share_pageChange"
                        @sortChange="share_sortChange"
                        :state="true"
                ></share-list>
            </el-tab-pane>
        </el-tabs>

    </div>
</template>

<script>
    import StockInfo from "../components/StockInfo";
    import DailyList from "../components/DailyList";
    import ShareList from "../components/ShareList";
    import HkholdList from "../components/HkholdList";

    import {getShareList, getStock, getStockDaily, getHkholdList} from "@/network/stock_detail";

    export default {
        name: "StockDetail",
        components: {ShareList, DailyList, StockInfo,HkholdList},
        props: {},
        data() {
            return {
                stock: {},
                activeName: "first",
                daily_total: 0,
                daily_basics: [],
                daily_context: {
                    ts_code: '',
                    prop: 'trade_date',
                    order: 'descending',
                    offset: 0,
                    page_size: 15,
                },

                hkhold_total: 0,
                hkholds: [],
                hkhold_context: {
                    ts_code: '',
                    prop: 'trade_date',
                    order: 'descending',
                    offset: 0,
                    page_size: 15,
                    proc_filter: [],
                },

                share_total: 0,
                shares: [],
                share_context: {
                    ts_code: '',
                    prop: 'ann_date',
                    order: 'descending',
                    offset: 0,
                    page_size: 15,
                    proc_filter: [],
                },

            };
        },
        mounted() {
            this.activeName = this.$route.query.myTab;
        },
        methods: {
            prev() {
                this.$router.go(-1)
                // this.$router.back()
            },
            daily_sortChange(prop, order) {
                this.daily_context.prop = prop
                this.daily_context.order = order
            },
            //仅在pageChange中进行请求
            daily_pageChange(offset, PageSize) {
                this.daily_context['offset'] = offset
                this.daily_context['page_size'] = PageSize
                this.$refs.daily_list.loading = true
                getStockDaily(this.daily_context).then(res => {
                    this.daily_total = res.data['total']
                    this.daily_basics = res.data['daily_basics']
                    this.$refs.daily_list.loading = false
                })
            },

            hkhold_sortChange(prop, order) {
                this.hkhold_context.prop = prop
                this.hkhold_context.order = order
            },
            //仅在pageChange中进行请求
            hkhold_pageChange(offset, PageSize) {
                this.hkhold_context['offset'] = offset
                this.hkhold_context['page_size'] = PageSize
                this.$refs.hkhold_list.loading = true
                getHkholdList(this.hkhold_context).then(res => {
                    this.hkhold_total = res.data['total']
                    this.hkholds = res.data['hk_holds']
                    this.$refs.hkhold_list.loading = false
                })
            },

            share_filterChange(proc_filter) {
                this.share_context.proc_filter = proc_filter
            },
            share_sortChange(prop, order) {
                this.share_context.prop = prop
                this.share_context.order = order
            },
            //仅在pageChange中进行请求
            share_pageChange(offset, PageSize) {
                this.share_context['offset'] = offset
                this.share_context['page_size'] = PageSize
                this.$refs.share_list.loading = true
                getShareList(this.share_context).then(res => {
                    this.share_total = res.data['total']
                    this.shares = res.data['shares']
                    this.$refs.share_list.loading = false
                })
            }
        },

        created() {

            this.daily_context.ts_code = this.$route.query.ts_code
            getStock({ts_code: this.daily_context.ts_code}).then(res => {
                this.stock = res.data['stock']
            })
            //并发
            getStockDaily(this.daily_context).then(res => {
                this.daily_total = res.data['total']
                this.daily_basics = res.data['daily_basics']
                this.$refs.daily_list.loading = false
            })

            this.hkhold_context.ts_code = this.$route.query.ts_code
            getHkholdList(this.hkhold_context).then(res => {
                this.hkhold_total = res.data['total']
                this.hkholds = res.data['hk_holds']
                console.log(this.$refs.hkhold_list.loading);
                this.$refs.hkhold_list.loading = false
            })

            this.share_context.ts_code = this.$route.query.ts_code
            // getStock({ts_code: this.share_context.ts_code}).then(res => {
            //   this.stock = res.data['stock']
            // })
            getShareList(this.share_context).then(res => {
                this.share_total = res.data['total']
                this.shares = res.data['shares']
                this.$refs.share_list.loading = false
            })
        }
    };
</script>

<style scoped>
    .el-row {
        margin-bottom: 20px;

    }

    .el-col {
        border-radius: 4px;
    }

    .button_info {
        margin-top: 5%;
        margin-left: 2%;
        font-size: 15px;
        /*color: black;*/
        /*border-color: #545c64;*/
    }


    .el-button.button_info:hover {
        color: #545c64;
        border-color: #dadbdc;
        background-color: #eff0f0;
    }

    .el-button.button_info:active {
        color: #545c64;
        border-color: #dadbdc;
        background-color: #eff0f0;
    }


</style>
<style>
    /*标签页样式*/
    .el-tabs__item {
        padding: 0 20px;
        height: 40px;
        -webkit-box-sizing: border-box;
        box-sizing: border-box;
        line-height: 40px;
        display: inline-block;
        list-style: none;
        font-size: 13px;
        font-weight: 600;
        color: #545c64;
        position: relative;
    }

    .el-tabs--card > .el-tabs__header {
        /*border-bottom: 1px solid #545c64;*/
        background-color: #fff;
    }

    .el-tabs__item.is-active {
        color: #EEB422;
    }

    .el-tabs__item:hover {
        color: #EEB422;
        cursor: pointer;
    }
</style>
