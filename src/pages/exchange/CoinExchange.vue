<template>
    <div>
        <el-container class="fs-container">
            <el-aside class="leftside" style="width:370px;">
                <div class="inner-wrapper">
                    <el-container class="dash">
                        <el-aside class="left-pic"  style="width:88px;">
                            <img class="left-pic"  src="@icon64/eth.png" alt=""/>
                        </el-aside>
                        <el-main class="right-word">
                            <div class="right-l1" v-if="price.usdt[0]" >ETH/USDT  {{price.usdt[0].order_price}}</div>
                            <div class="right-l2" v-if="price.usdt[0]">≈  {{(price.usdt[0].order_price*6.3).toFixed(2)}}CNY</div>
                            <div class="right-l3"  v-if="price.usdt[0]" :class="price.usdt[0].p>0?'red':'green'">{{curPrice}}%</div>
                            <div class="right-l4"  v-if="price.usdt[0]" >高：{{price.usdt[0].high}} 低：{{price.usdt[0].low}}</div>
                        </el-main>
                        
                    </el-container>
                    <!-- 市场 -->
                    <div class="cur-price">
                        <div class="tabs-header">
                            <i class="arrow-right el-icon-arrow-right"></i>
                            市场
                        </div>
                        <el-tabs type="border-card" class="market-table">
                            
                            <el-tab-pane label="USDT">
                                <div class="market-list-header" >
                                    <span class="rel1"></span>
                                    <span class="rel2">币种</span>
                                    <span class="rel3">最新价</span>
                                    <span class="rel4">24h涨幅</span>
                                </div>
                                <div class="vuebar-element" v-bar="{preventParentScroll:true,scrollThrottle:50}"> <!-- el1 -->
                                    <div>
                                        <div class="market-list" v-for="(item,index) in marketListUSDT" :key="index">
                                            <span class="rel1">
                                                {{item.icon}}
                                            </span>
                                            <span class="rel2">
                                                {{item.name.toUpperCase()}}
                                            </span>
                                            <span class="rel3">{{item.order_price}}</span>
                                            <span class="rel4" :class="item.positive?'rise':'fall'">{{Math.abs(item.p)}}%</span>
                                        </div>
                                    </div>
                                </div>
                            </el-tab-pane>
                            <!-- <el-tab-pane label="UT">
                                <div class="market-list-header" >
                                    <span class="rel1"></span>
                                    <span class="rel2">币种</span>
                                    <span class="rel3">最新价</span>
                                    <span class="rel4">24h涨幅</span>
                                </div>
                                <div class="vuebar-element" v-bar="{preventParentScroll:true,scrollThrottle:50}">
                                    <div> 
                                        
                                        <div class="market-list" v-for="(item,index) in marketListUT"  :key="index">
                                            <span class="rel1">{{item.icon}}</span>
                                            <span class="rel2">{{item.name.toUpperCase()}}</span>
                                            <span class="rel3">{{item.order_price}}</span>
                                            <span class="rel4" :class="item.positive?'rise':'fall'">{{Math.abs(item.p)}}%</span>
                                        </div>
                                    </div>
                                </div>
                            </el-tab-pane> -->
                        </el-tabs> 
                   </div>

                    <!-- 交易 -->
                    <div class="exchange">
                        <div class="tabs-header">
                            <i class="arrow-right el-icon-arrow-right"></i>
                            交易
                        </div>
                        <el-tabs type="border-card" v-model="exchange.orderType" class="market-table"> 
                            <el-tab-pane name="limitprice" label="限价">
                                <div class="tips">
                                    可用：{{exchange.balance}} USDT
                                </div>
                                <div class="deal-form">
                                    <el-row :gutter="20">
                                        <el-col :span="12">
                                            <div class="amount-label">
                                                价格
                                            </div>
                                            <el-input class="amount-input" type="number" v-model.number="exchange.limitPriceDeal.price">
                                                <template slot="append">USDT</template>
                                            </el-input>
                                        </el-col>
                                        <el-col :span="12">
                                            <div class="amount-label">
                                                数量
                                            </div>
                                            <el-input class="amount-input" type="number" v-model.number="exchange.limitPriceDeal.amount">
                                                <template slot="append">ETH</template>
                                            </el-input>
                                        </el-col>
                                    </el-row>
                                    <el-slider 
                                    v-model="exchange.limitPriceDeal.percent" 
                                    :show-tooltip="false"
                                    :show-stops="true"
                                    min.number="0"
                                    step.number="5"
                                    @change="computeLimitAmount"
                                    ></el-slider>
                                    <div class="amount-tips">
                                    交易额:{{allprice}} USDT
                                    </div>
                                    <div class="btn-wrapper">
                                        <el-button 
                                        type="purchase" 
                                        value="purchase" 
                                        @click="addDelegate($event)">买入</el-button>
                                        <el-button 
                                        type="sell" 
                                        value="sell"  
                                        @click="addDelegate($event)">卖出</el-button>
                                    </div>
                                </div>
                            </el-tab-pane>
                            <!-- <el-tab-pane  name="marketprice" label="市价">
                                <div class="tips">
                                    可用：{{exchange.balance}} USDT
                                </div>
                                <div class="tips-big">
                                    *市场最优价格（买卖）
                                </div>
                                
                                <div class="deal-form">
                                    <el-row :gutter="20">
                                        <el-col :span="12">
                                            <div class="amount-label">
                                                数量
                                            </div>
                                            <el-input class="amount-input" type="number" 
                                            name="buy"  v-model.number="exchange.marketPriceDeal.price" 
                                            :disabled="exchange.marketPriceDeal.buyDisable"
                                            @focus="getFocus"
                                            @keyup.native="getBuyKeyup"
                                            placeholder="买入交易额" >
                                                <template slot="append">USDT</template>
                                            </el-input>
                                        </el-col>
                                        <el-col :span="12">
                                            <div class="amount-label">
                                                数量
                                            </div>
                                            <el-input class="amount-input" type="number"
                                            name="sell"  
                                            placeholder="卖出数量" 
                                            :disabled="exchange.marketPriceDeal.sellDisable"
                                            @focus="getFocus"
                                            @keyup.native="getSellKeyup"
                                            v-model.number="exchange.marketPriceDeal.amount">
                                                <template slot="append">ETH</template>
                                            </el-input>
                                        </el-col>
                                    </el-row>
                                    <el-slider 
                                    v-model="exchange.marketPriceDeal.percent"
                                    :show-tooltip="false"
                                    :show-stops="true"
                                    min.number="0"
                                    step.number="5"
                                    @change="computeMarketAmount"
                                    ></el-slider>
                                    <div class="btn-wrapper">
                                        <el-button type="purchase" 
                                        value="purchase" 
                                        @click="addDelegate($event)"
                                        :disabled="exchange.marketPriceDeal.buyDisable">买入</el-button>
                                        <el-button type="sell" 
                                        value="sell"  
                                        @click="addDelegate($event)"
                                        :disabled="exchange.marketPriceDeal.sellDisable">卖出</el-button>
                                    </div>
                                </div>
                            </el-tab-pane> -->
                        </el-tabs> 
                    </div>


                    <!-- 持有 -->
                    <div class="mycoins">
                        <div class="normal-header">
                            <i class="arrow-right el-icon-arrow-right"></i>
                            持有
                        </div>
                        <div class="market-list-header pad15" >
                            <span class="rel1">币种</span>
                            <span class="rel2">可用</span>
                            <span class="rel3">冻结</span>
                            <span class="rel4">操作</span>
                        </div>
                        <div class="mycoins-table">
                            <div class="vuebar-element" 
                            v-bar="{preventParentScroll:true,scrollThrottle:50}">
                                <div >
                                    <div class="mycoin-list" v-for="(item,index) in mycoins" :key="index">
                                        <span class="rel1">{{item.currency.toUpperCase()}}</span>
                                        <span class="rel2">{{item.available}}</span>
                                        <span class="rel3">{{item.frozen}}</span>
                                        <span class="rel4">
                                             <el-button class="get-coin" @click="getCoin(item,index)" type="text" size="small">提币</el-button>
                                             <el-button  class="add-coin" @click="addCoin(item,index)" type="text" size="small">充币</el-button>
                                        </span>
                                    </div>
                                </div>
                            </div> 
                        </div>
                    </div>


                    <div class="boardcast">
                        <div class="normal-header">
                            <i class="arrow-right el-icon-arrow-right"></i>
                            公告
                        </div>
                        <div class="vuebar-element" v-bar="{preventParentScroll:true,scrollThrottle:50}">
                            <div >
                                <ul class="notice-list">
                                    <li class="notice-li" v-for="(item,index) in noticeList" :key="index">
                                        <span class="notice-squre">·</span>{{item.notice_title}}
                                    </li>
                                </ul>
                            </div>
                        </div> 
                    </div>
                </div>
            </el-aside>
            <el-main class="rightside">
                <div class="kline">
                    <div class="kline-header">
                         <i class="arrow-right el-icon-arrow-right"></i>
                            图表
                            <el-select v-model="curChart" style="width:140px;">
                                <el-option
                                v-for="item in chartOptions"
                                :key="item.value"
                                :label="item.label"
                                :value="item.value">
                                </el-option>
                            </el-select>
                    </div>
                    <div style="height:400px;">
                       <iframe id="show-iframe"  frameborder=0 name="showHere" scrolling=auto src="http://frontend.sy.sxurl.cn/kline/versiontwo?plate_id=usdt_to_eth&1524061966"></iframe> 
                    </div>
                </div>
                <div class="exchange-table">
                    <div class="exchange-table-header cur-delegation">
                        <i class="arrow-right el-icon-arrow-right"></i>
                            当前委托
                    </div>
                    <el-table class="center-table"
                    :data="curDelegation"
                    style="width: 100%">
                        <el-table-column
                            prop="time"
                            label="时间"
                            width="180"
                            >
                        </el-table-column>
                        <el-table-column
                            prop="duad"
                            label="交易对"
                            >
                            <template slot-scope="scope">
                                {{scope.row.duad.toUpperCase()}}
                            </template>
                        </el-table-column>
                        <el-table-column
                            prop="direction"
                            label="方向">
                            <template slot-scope="scope">
                                <span 
                                :class="scope.row.direction == 1?'buy-direction':'sell-direction'">
                                    {{scope.row.direction == 1?"买入":"卖出"}}
                                </span>
                            </template>
                        </el-table-column>
                        <el-table-column
                            prop="price"
                            label="价格(USDT)">
                        </el-table-column>
                        <el-table-column
                            prop="number"
                            label="数量(ETH)">
                        </el-table-column>
                        <el-table-column
                            prop="total"
                            label="委托总额">
                        </el-table-column>
                        <el-table-column
                            prop="deal"
                            label="已成交">
                        </el-table-column>
                        <el-table-column
                            prop="untreated"
                            label="未成交">
                        </el-table-column>
                        <el-table-column
                            label="操作">
                            <template slot-scope="scope">
                                <el-button @click="cancelDelegate(scope.row)" type="text" size="small">撤销</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </div> 
                <div class="exchange-table">
                    <div class="exchange-table-header cur-delegation">
                        <i class="arrow-right el-icon-arrow-right"></i>
                            历史委托
                    </div>
                    <el-table class="center-table"
                    :data="hisDelegation"
                    style="width: 100%">
                        <el-table-column
                            prop="time"
                            label="时间"
                            width="180"
                            >
                        </el-table-column>
                        <el-table-column
                            prop="duad"
                            label="交易对"
                            >
                            <template slot-scope="scope">
                                {{scope.row.duad.toUpperCase()}}
                            </template>
                        </el-table-column>
                        <el-table-column
                            prop="direction"
                            label="方向">
                            <template slot-scope="scope">
                                <span 
                                :class="scope.row.direction == 1?'buy-direction':'sell-direction'">
                                    {{scope.row.direction == 1?"买入":"卖出"}}
                                </span>
                            </template>
                        </el-table-column>
                        <el-table-column
                            prop="price"
                            label="价格(USDT)">
                        </el-table-column>
                        <el-table-column
                            prop="number"
                            label="数量(ETH)">
                        </el-table-column>
                        <el-table-column
                            prop="total"
                            label="委托总额">
                        </el-table-column>
                        <el-table-column
                            prop="deal"
                            label="已成交">
                        </el-table-column>
                        <el-table-column
                            prop="Untreated"
                            label="未成交">
                        </el-table-column>
                        <!-- <el-table-column
                            label="操作">
                            <template slot-scope="scope">
                                <el-button @click="delHistory(scope.row)" type="text" size="small">删除</el-button>
                            </template>
                        </el-table-column> -->
                    </el-table>
                </div>  
                <el-row :gutter="20">
                    <el-col :span="16">
                        <div class="exchange-table">
                            <div class="exchange-table-header cur-delegation">
                                <i class="arrow-right el-icon-arrow-right"></i>
                                    深度图
                            </div>
                            <el-row>
                                <el-col :span="12">
                                    <el-table class="center-table"
                                    :data="realTimeDeal.buy"
                                    style="width: 100%">
                                        <el-table-column
                                            prop="role"
                                            label="买盘"
                                            >
                                        </el-table-column>
                                        <el-table-column
                                            prop="amount"
                                            label="数量"
                                            >
                                        </el-table-column>
                                        <el-table-column
                                            prop="allamount"
                                            label="累计">
                                        </el-table-column>
                                        <el-table-column
                                            prop="order_price"
                                            label="价格">
                                        </el-table-column>
                                    </el-table>
                                </el-col>
                                <el-col :span="12">
                                    <el-table class="center-table"
                                    :data="realTimeDeal.sell"
                                    style="width: 100%">
                                        <el-table-column
                                            prop="order_price"
                                            label="价格"
                                            >
                                        </el-table-column>
                                        <el-table-column
                                            prop="allamount"
                                            label="累计">
                                        </el-table-column>
                                        <el-table-column
                                            prop="amount"
                                            label="数量"
                                            >
                                        </el-table-column>
                                        <el-table-column
                                            prop="role"
                                            label="卖盘">
                                        </el-table-column>
                                    </el-table>
                                </el-col>
                            </el-row>  
                        </div> 
                    </el-col>
                    <el-col :span="8">
                        <div class="exchange-table">
                            <div class="exchange-table-header cur-delegation">
                                <i class="arrow-right el-icon-arrow-right"></i>
                                    实时成交
                            </div>
                            <el-table class="exchange-table"
                            :data="realTimeDealList"
                            style="width: 100%">
                                <el-table-column
                                    prop="order_time"
                                    label="时间"
                                    >
                                </el-table-column>
                                <el-table-column
                                    prop="order_price"
                                    label="价格">
                                </el-table-column>
                                <el-table-column
                                    prop="processed_amount"
                                    label="数量">
                                </el-table-column>
                                
                            </el-table>
                        </div> 
                    </el-col>
                </el-row>
            </el-main>
        </el-container>
    </div>
</template>
<script>
// import Vue from 'vue'
// import VueWebsocket from "vue-websocket";
// Vue.use(VueWebsocket, "ws://54.65.108.119:9541")
import ScrollBar from 'vue2-scrollbar'
import { api } from '@/static/api'
import { functionDeclaration } from 'babel-types';
import { coinTypeIcon } from '@/static/dataConfig';

// import ReconnectingWebSocket from 'reconnecting-websocket'
export default {
    name:'CoinExchange',
    props:{
        
    },
    data(){
        return{
            
            // socket_1:new WebSocket('ws://54.65.108.119:9541'),
            socket_1:new WebSocket('ws://54.65.108.119:9541'),
            socket_2:new WebSocket('ws://54.65.108.119:9542'),
            barContainerStyle:{
                width:'6px',
                backgroundColor:'#202832'
            },
            barStyle:{
                width:'6px',
                backgroundColor:'#344253'
            },
            price:{
                // isPositive:true,
                // btc:{
                //     high:0,
                //     low:0,
                //     order_price:0,
                //     p:0
                // }
                usdt:[],
                ut:[]
            },
            marketListUSDT:[
                // {
                //     icon:'$',
                //     type:'BTC',
                //     price:'11499.975',
                //     rise:'2.28',
                //     positive:true
                // }
            ],
            marketListUT:[
                // {
                //     icon:'',
                //     type:'BTC',
                //     price:'23499.975',
                //     rise:'2.28'
                // }
            ],
            mycoins:[
                // {
                //     plate_en:'btc',
                //     available:199,
                //     frozen:19,
                //     id:1003
                // }
            ],
            exchange:{
                orderType:'limitprice',
                focusChoose:'buy',
                balance:0,
                percent:0,
                // price:0,
                marketPriceDeal:{
                    percent:0,
                    amount:0,
                    price:0
                },
                limitPriceDeal:{
                    percent:0,
                    amount:0,
                    price:0,
                    buyDisable:false,
                    sellDisable:false
                }
            },
            noticeList:[
                {
                    time:'2018-03-09',
                    notice_title:'2323',
                    notice_content:"2323"
                },
                {
                    time:'2018-03-09',
                    notice_title:'2323',
                    notice_content:"2323"
                },
                {
                    time:'2018-03-09',
                    notice_title:'2323',
                    notice_content:"2323"
                },
                {
                    time:'2018-03-09',
                    notice_title:'2323',
                    notice_content:"2323"
                }
            ],
            curChart:'ETH/USDT',
            chartOptions:[
                {
                    value:'ETH/USDT',
                    label:'ETH/USDT'
                }
                // {
                //     value:'BTC/USDT',
                //     label:'BTC/USDT'
                // }
            ],
            curDelegation:[],
            hisDelegation:[],
            realTimeDeal:{
                buy:[],
                sell:[]
            },
            realTimeDealList:[]
        }
    },
    components:{
      ScrollBar
    },
    computed:{
        allprice(){
            return (this.exchange.limitPriceDeal.price * this.exchange.limitPriceDeal.amount).toFixed(4)
        },
        allcurprice(){

        },
        curPrice(){
            if(this.price.usdt && this.price.usdt[0] && this.price.usdt[0].p){
                return Math.abs(this.price.usdt[0].p)
            }else{
                return 0
            }
        }
    },
    created(){
        this.getCurDelegate()
        this.getHisDelegate()
        this.getNotice()
        this.getAssetslist()
        this.userAccount()
    },
    mounted () {
        var self = this
        this.socket_1.onmessage = function(data){
            var res = JSON.parse(data.data)
            console.log(res)
            self.normalizeCurPrice(res.price)
            self.normalizeRealTime(res.sb)
        }
        this.socket_2.onmessage = function(data){
            var res = JSON.parse(data.data)
            console.log(res)
            self.realTimeDealList = res
        }
    },
    watch:{
        
    },
    methods:{
        normalizeRealTime(res){
            var self = this
            self.realTimeDeal.buy = []
            self.realTimeDeal.sell = []
            for(var i =0,imax = res.buy.length;i<imax;i++){
                var item = {}
                    item.allamount = res.buy[i].allamount
                    item.amount = res.buy[i].amount
                    item.order_price = res.buy[i].order_price
                    item.role = '买' + (i+1).toString()
                self.realTimeDeal.buy.push(item)
            }
            for(var i =0,imax = res.sell.length;i<imax;i++){
                var item = {}        
                    item.allamount = res.sell[i].allamount
                    item.amount = res.sell[i].amount
                    item.order_price = res.sell[i].order_price
                    item.role = '卖' + (i+1).toString()
                self.realTimeDeal.sell.push(item)
            }
        },
        normalizeCurPrice(res){
            var self = this
            this.price = res
            this.marketListUSDT = res.usdt
            this.marketListUT = res.ut
            for(var i in this.marketListUSDT){
                self.marketListUSDT[i].icon = '$'
                self.marketListUSDT[i].positive = self.marketListUSDT[i].p >=0?true:false
            }
            for(var i in this.marketListUT){
                self.marketListUT[i].icon = '$'
                self.marketListUT[i].positive = self.marketListUSDT[i].p >=0?true:false
            }
        },
        computeLimitAmount(){
            console.log(this.exchange.limitPriceDeal.price,this.exchange.limitPriceDeal.percent)
            if(this.exchange.limitPriceDeal.price == 0){
                this.exchange.limitPriceDeal.amount = 0 
                return
            }else{                
                this.exchange.limitPriceDeal.amount = (Math.floor(this.exchange.balance/this.exchange.limitPriceDeal.price * this.exchange.limitPriceDeal.percent/100))
            }
            console.log('限价交易：',this.exchange.limitPriceDeal.amount)
        },
        getFocus(e){
            this.exchange.focusChoose = e.target.name
        },
        getBuyKeyup(e){
            if(Number(e.target.value) > 0){
                this.exchange.marketPriceDeal.buyDisable = false
                this.exchange.marketPriceDeal.sellDisable = true
            }else{
                this.exchange.marketPriceDeal.sellDisable = false
            }
        },
        getSellKeyup(e){
            if(Number(e.target.value) > 0){
                this.exchange.marketPriceDeal.buyDisable = true
                this.exchange.marketPriceDeal.sellDisable = false
            }else{
                this.exchange.marketPriceDeal.buyDisable = false
            }
        },
        computeMarketAmount(){
            if(this.exchange.focusChoose == 'sell'){
                // 市价卖
                this.exchange.marketPriceDeal.amount = (this.exchange.marketPriceDeal.percent * this.exchange.balance).toFixed(2)
                if(this.exchange.marketPriceDeal.amount > 0){
                    this.exchange.marketPriceDeal.buyDisable = true
                }else{
                    this.exchange.marketPriceDeal.buyDisable = false
                }
                return
            }else if(this.exchange.focusChoose == 'buy'){
                // 市价买         
                this.exchange.marketPriceDeal.price = (this.exchange.marketPriceDeal.percent * this.exchange.balance).toFixed(2)
                if(this.exchange.marketPriceDeal.price > 0){
                    this.exchange.marketPriceDeal.sellDisable = true
                }else{
                    this.exchange.marketPriceDeal.sellDisable = false
                } 
            }
            console.log('市价交易：',this.exchange.marketPriceDeal.amount == 0,this.exchange.marketPriceDeal.price == 0)
        },
        computePercent(){
            this.exchange.percent = Math.floor(this.exchange.amount/this.exchange.balance*100)
        },
        // 获取当前委托
        getCurDelegate(){
            var data = {
                page:1
            }
            api.curDelegate(data)
            .then(res => {
                console.log("当前委托",res)
                if(res.error_code == 1000){
                    this.curDelegation = res.entrusts
                }
            }).catch(err => {

            })
        },
        // 获取历史委托
        getHisDelegate(){
            var data = {
                page:1
            }
            api.hisDelegate(data)
            .then(res => {
                console.log("历史委托",res)
                this.hisDelegation = res.entrusts
            }).catch(err => {

            })
        },
        // 新增委托
        addDelegate($event){
            console.log($event.currentTarget.value,this.exchange.orderType)
            const trade_type = $event.currentTarget.value == 'purchase'?1:2

            let reg = /\d*/
            // if(this.exchange.amount < 0 || !reg.test(this.exchange.amount)){
            //     this.$message('交易数量大于等于0')
            //     this.exchange.amount = 0
            //     return
            // }
            
            
            var deal = this.exchange.orderType == 'limitprice'?this.exchange.limitPriceDeal:this.exchange.marketPriceDeal

            var orderType = this.exchange.orderType == 'limitprice'?0:1
            if(this.exchange.orderType == 0 && this.exchange.limitPriceDeal.amount * this.exchange.limitPriceDeal.price  > this.exchange.balance){
                this.$message('交易额超过当前账户余额')
                return
            }
            var data = {
                currency:'USDT',                     //基础货币，货币符号 CNY,BTC，ETH，UT等
                order_amount:deal.amount,  //订单数量
                order_price:Number(deal.price),          //订单价格
                order_type:orderType,                    //订单类型，0 限价单，1市价单
                trade_currency:'ETH',               //交易货币，货币符号 CNY,BTC，ETH，UT等
                trade_type:trade_type               //交易类型，0买单，1买单
            }
            api.addDelegate(data)
            .then(res => {
                if(res.error_code == 1000){
                    this.userAccount()
                    this.getCurDelegate()
                }
                this.$message(res.error_desc)
            }).catch(err => {

            })
        },
        // 取消委托
        cancelDelegate(row){
            console.log(row)
            var data = {
                order_id:row.id,
                currency:row.order_type
            }
            api.cancelDelegate(data)
            .then(res => {
                if(res.error_code == 1000){
                    this.$message(res.error_desc)
                    this.getCurDelegate()
                }
            }).catch(err => {
                this.$message('网络失败请重试')
            })
        },
        getNotice(){
            var data = {
                type:1
            }
            api.getNotice(data)
            .then(res => {
                console.log(res)
                if(res.error_code == 1000){
                    this.noticeList = res.data
                }
            }).catch(err => {

            })
        },
        getAssetslist(){
            var data = {}
            api.getAssetslist(data)
            .then(res=>{
                if(res.error_code == 1000){
                    this.mycoins = res.assets_list
                }
            }).catch(err => {

            })
        },
        // 提币
        getCoin(item){
            console.log(item)
            this.$router.replace({name:'coinoption',params: { dealId: item.id,type:2}})
        },
        // 冲币
        addCoin(item){
            console.log(item)
            this.$router.replace({name:'coinoption',params: { dealId: item.id,type:1}})
        },
        userAccount(){
            var data = {
                currency:'usdt'
            }
            api.userAccount(data)
            .then(res => {
                if(res.error_code == 1000){
                    this.exchange.balance = res.available
                }
            }).catch(err => {

            })
        }
    }
}
</script>
<style lang='scss' scoped>
.coin-icon{
    width: 18px;
    height: 18px;
    display: block;
}
.market-table{
    background-color: #191f27;
    font-size: 14px;
    color: #617997;
    position: relative;
}
.vuebar-element {
  height: 150px;
  width: 100%;
  background: #191f27;
  transform: rotate3d(0,0,0,0);
}
.market-list-header,
.market-list{
    width: 100%;
    background: #191f27;
    // overflow: hidden;
    white-space:nowrap; 
    color:#c2c3ca;
    line-height: 1.5;
    
    .rel1{
        width: 36px;
        display: inline-block;
        text-align: center;
        white-space:nowrap; 
        overflow: hidden;
    }
    .rel2{
        width: 70px;
        display: inline-block;
        text-align: center;
        white-space:nowrap; 
        overflow: hidden;
    }
    .rel3{
        width: 100px;
        display: inline-block;
        text-align: center;
        white-space:nowrap; 
        overflow: hidden;
    }
    .rel4{
        width: 80px;
        display: inline-block;
        text-align: center;
        white-space:nowrap; 
        overflow: hidden;
    }
   
    .rel4.rise{
        width: 80px;
        display: inline-block;
        color:#a4454b;
        text-align: center;
    }
    .rel4.fall{
        width: 80px;
        display: inline-block;
        color:#5ead6f;
        text-align: center;
    }
}

.mycoin-list{
    width: 100%;
    background: #191f27;
    color:#c2c3ca;
    line-height: 1.5;
    font-size: 10px;
    overflow: hidden;
    white-space:nowrap; 

    .rel1{
        width: 36px;
        display: inline-block;
        text-align: center;
        overflow: hidden;
        white-space:nowrap; 
    }
    .rel2{
        width: 72px;
        display: inline-block;
        text-align: center;
        overflow: hidden;
        white-space:nowrap; 
    }
    .rel3{
        width: 72px;
        display: inline-block;
        text-align: center;
        overflow: hidden;
        white-space:nowrap; 
    }
    .rel4{
        width: 80px;
        display: inline-block;
        text-align: center;
        overflow: hidden;
        white-space:nowrap; 
    }
    .get-coin{
        background-color:#344253;
        padding: 5px;
        color: #8e9aa9;
        border-radius: 0;
    }
    .add-coin{
        background-color:#495d75;
        padding: 5px;
        color: #8e9aa9;
        border-radius: 0;
        margin-left: 5px;
    }
}
.market-list-header{
    font-size: 12px;
    color:#384456;
    line-height: 1.5;
    height: 28px;
}
.pad15{
    padding: 0 15px;
}
.market-list{
    text-align: center;
    font-size:12px;
}

.mycoins-table{
    padding:0 15px;
}
.notice-list{
    width: 100%;
    background: #191f27;
    overflow: hidden;
    white-space:nowrap; 
    text-overflow: ellipsis;
    color:#c2c3ca;
    list-style-type: none;
    padding-left:12px;
    padding-right: 12px;
    .notice-squre{
        width: 20px;
        display: inline-block;
        text-align: center;
        padding-top: 3px;
        padding-bottom: 3px;
    }
    .notice-li{
        line-height: 1.5;
        font-size: 12px;
        padding-top: 3px;
        padding-bottom: 3px;
        overflow: hidden;
        white-space:nowrap; 
        text-overflow: ellipsis;
    }
}

.exchange-table{
    font-size: 12px;
}
.center-table{
    text-align: center;
}

#show-iframe{
    width: 100%;
    height: 100%;
}
</style>