<template>
	<div class="coinoption" >
		<div class="sjx_main">
			<div class="sjx_section">
				<div class="section_title">
					<div class="top-title">
						<!--总资产合计：{{balance}}-->
						<!--&nbsp;&nbsp;USDT<span style="font-size:14px;color:#344252;"> ≈ {{(balance * 6.3).toFixed(2)}} CNY</span>-->
						<div class="quikc-nav">
							<router-link class="quikc-link" to='/property/financialrecords' style="margin-right:26px;">财务记录</router-link>
							<router-link class="quikc-link" to='/property/withdrawaddressmanagement'>提币地址管理</router-link>
						</div>
					</div>
				</div>

				<el-table class="exchange-table coinlist" :data="assetsList" style="width: 100%">
					<el-table-column prop="currency" label="币种">
						<template slot-scope="scope">
							{{scope.row.currency.toUpperCase()}}
						</template>
					</el-table-column>
					<el-table-column prop="available" label="可用">
					</el-table-column>
					<el-table-column prop="frozen" label="冻结">
					</el-table-column>
					<el-table-column prop="options" label="操作" style='position: relative;'>
						<template slot-scope="scope">
							<el-popover popper-class="xm-popover"
							offset=0
							placement="bottom-end"
							ref="popover" width="800" trigger="click">
								<div>
									<p class="popover-titter">充币地址</p>
								</div>
								<button v-if="!rechargeInfo.showAddress"  @click="rechargeCoin">充币地址申请</button>
								<span v-if="rechargeInfo.showAddress">{{rechargeInfo.rechargeAddress}}</span>
								<button v-if="rechargeInfo.showAddress" type="button"
									v-clipboard:copy="rechargeInfo.rechargeAddress"
									v-clipboard:success="onCopy"
									v-clipboard:error="onError">复制</button>
								<p class="popover-hint">温馨提示</p>
								<ul>
									<li>请勿向上述地址重置任何非BTC资产，否则资产将不可找回。</li>
									<li>您充值至上述地址后，需要整个网络节点的确认，1次网络确认后到账，6次网络确认后可提币。</li>
									<li>最小充值金额：0.001BTF，小于最小金额的充值将不会上账。</li>
									<li>您的充值地址不会经常改变，可以重复充值；如有更改，我们会尽量通过网站公告或邮箱通知您。</li>
									<li>请务必确认电脑及浏览器安全，防止信息被篡改或泄露。</li>
								</ul>
							</el-popover>
							<el-popover popper-class="xm-pop" ref="pop" offset=0 width="800" trigger="click" placement="bottom-end" content="提币" style="margin-top:-10px;">
								<div>
									<p class="pop-titter">提币地址</p>
								</div>
								 <!-- <input type="text" class="pop-titter-div" v-model="Mention" /> -->
								 <!-- <div class="pop-titter-div" >{{withdrawInfo.withdrawAddress}}</div> -->
								 <el-select class="pop-titter-div" v-model="withdrawInfo.selectedAddress">
									<el-option
									v-for="item in withdrawInfo.withdrawAddress"
									:key="item.address"
									:label="item.address"
									:value="item.address">
									</el-option>
								</el-select>
								<p style="margin-top: 15px;">数量</p>
								<p class="xm-can-ast">
									<span>可提:{{withdrawInfo.avaliable}}</span>
									<span>限额:{{withdrawInfo.realAmount}}</span>
								</p>
								<span class="combo">
								<input class="combo combo-text" type="number" v-model.number="withdrawInfo.amount" @keyup="calFee">
								<span style="margin-right: 10px;">{{withdrawInfo.currency.toUpperCase()}}</span>
								</span>
								<div style="overflow:hidden;">
								<div id="xm_num">
								<p style="margin-left: 15px;font-size:12px;color: #9ea2f9;">手续费</p>
								<span class="combo">
								<!-- <input class="combo combo-text1" v-model="fee"/>  -->
								<span class="combo combo-text1">{{withdrawInfo.fee}}</span>
								<span  style="padding-right:15px;line-height:35px;">
									{{withdrawInfo.currency.toUpperCase()}}
								</span>

								</span>
								</div>
								<div id="xm_num">
									<p style="margin-left: 15px;font-size:12px;color: #9ea2f9;">到账数量</p>
									<span class="combo" >
										<!-- <input class="combo combo-text2" v-model="readAmount" /> -->
										<span class="combo combo-text2">
											{{withdrawInfo.realAmount}}
										</span>
										<span style="padding-right:15px;line-height:35px;">
											{{withdrawInfo.currency.toUpperCase()}}
										</span>
									</span>

								</div>
								</div>

								<p class="popover-hint">温馨提示</p>
								<ul>
									<li>最小提币数为：200 USDT。</li>
									<li>为保障资金安全，当您账户安全策略变更、密码修改、使用新地址提币，我们会对提币进行人工审核，请耐心等待工作人员电话或邮件联系</li>
									<li>请务必确认电脑及浏览器安全，防止信息被篡改或泄露。</li>
								</ul>
								<button style="cursor: pointer;position: absolute;top: 295px;left: 570px; height: 50px;width: 200px;background:#4cb2f9;border:1px solid #aeb2ff;border-radius: 2px;color: #fcfcf2;font-size:16px;" @click="openVerifyDialog">提币</button>
							</el-popover>
							<el-button v-popover:popover type="text" size="small" class='recharge' @click="openRechargeDialog (scope.row)">充币</el-button>
							<el-button v-popover:pop type="text" size="small" class='tibi' ref='index++' @click='openWithdrawDialog(scope.row)'>提币</el-button>
						</template>
					</el-table-column>
				</el-table>
			</div>
		</div>
		<!-- 模态框 -->
		<el-dialog
			title="安全验证"
			:visible.sync="withdrawInfo.showVarifyModal"
			width="394px"
			@close="resetPop"
			center>
			<div class="login_div_password login_div_password2">
			<el-input class="pwd_input" type="text" v-model="verifyInfo.userEmail" placeholder="输入邮箱地址"></el-input>
			</div>
			<div class="login_div_password">
				<el-input class="pwd_input" type="password" v-model="verifyInfo.mailCode" placeholder="邮箱验证码" style="width:183px;"></el-input>
				<button class="pwd_btn" @click="getTakeCoinVerifyCode">发送验证码</button>
			</div>
			<div class="login_div_password">
				<el-input class="pwd_input" type="password" v-model="verifyInfo.pinCode" placeholder="交易密码填写"></el-input>
			</div>
			<span slot="footer" class="dialog-footer">
					<el-button class="dialog-confirm-btn" @click="withdrawCoin">确 定</el-button>
				</span>
		</el-dialog>
	</div>
</template>

<script>
	import { api } from '@/static/api'
	export default {
		name: 'CoinOption',
		data() {
			return {
				index:0,
				balance: 0.173617413,
				can: 3.9542,
				readAmount:0,
				astrict: 100000.00000000,
				Mention:"123DSA315sa3dAFSAASD",
				verifyInfo:{
					userEmail:'',
					mailCode:'',
					pinCode:''
				},
				rechargeInfo:{
					showAddress:false,
					rechargeAddress:'',
					plate:''
				},
				withdrawInfo:{
					showVarifyModal:false,
					selectedAddress:'',
					withdrawAddress:[],
					avaliable:'',
					fee:0,
					amount:0,
					realAmount:0,
					currency:''
				},
				assetsList: [
					// {
					// 	currency: 'BTC',
					// 	available: 182.000000,
					// 	frozen: 182.00000,
					// }
				],
			}
		},
		computed:{
		},
		created() {
			this.getAssetsList()
		},
		mounted() {

		},
		methods: {
			onCopy: function (e) {
				this.$message({
					message:'复制成功',
					type:'sucess'
				})
			},
			onError(){
				this.$message({
					message:'复制失败，请重试',
					type:'error'
				})
			},
			// 重置弹出框
			resetPop(){
				this.addingAddress = false
				this.verifyInfo.pinCode = ''
				this.verifyInfo.mailCode = ''
				this.verifyInfo.userEmail =''
			},
			// 获取资产列表
			getAssetsList(){
				api.getAssetslist()
				.then(res => {
					if(res.error_code == 1000){
						this.assetsList = res.assets_list
					}
				}).catch(err => {

				})
			},
			// 点击充币按钮
			openRechargeDialog(row){
				this.rechargeInfo.plate = row.currency
			},
			// 发送邮箱验证码
			getTakeCoinVerifyCode(){
				var data = {
					coin_name_en:this.withdrawInfo.currency
				}
				api.sendWithdrawCode(data)
				.then(res => {
					if(res.error_code == 1000){
						console.log(this.assetsList)
						this.$message('发送成功')
					}
				}).catch(err => { })
			},
			// 获取充币地址
			rechargeCoin(){
				console.log(1111111111)
				var data = {
					plate:this.rechargeInfo.plate
				}
				api.getRechargeAddress(data)
				.then(res => {
					if(res.error_code == 1000){
						this.rechargeInfo.rechargeAddress = res.user_address
						this.rechargeInfo.showAddress = true
						console.log(this.assetsList)
					}
				}).catch(err => {console.log(7777777)})
			},
			// 点击提币按钮
			openWithdrawDialog(row){
				this.withdrawInfo.currency = row.currency
				console.log(row,this.withdrawInfo,111222333)
				var data = {
					coin_name:row.currency
				}
				api.getWithdrawAddress(data)
				.then(res => {
					if(res.error_code == 1000){
						console.log(res,2323)
						if(res.error_code == 1000){
							console.log(res.data,999)
							this.withdrawInfo.withdrawAddress = res.data
							if(this.withdrawInfo.withdrawAddress.length > 0){
								this.withdrawInfo.selectedAddress = this.withdrawInfo.withdrawAddress[0].address
							}
						}
						this.withdrawInfo.currency = row.currency
					}
				}).catch(err => {})

				var data ={
					currency:row.currency
				}
				api.userAccount(data)
				.then(res =>{
					console.log(res,123)
					if(res.error_code == 1000){
						this.withdrawInfo.avaliable = res.available
					}
				}).catch(err => {})
			},
			openVerifyDialog(){
				this.withdrawInfo.showVarifyModal = true
			},
			// 提币
			withdrawCoin(){
				var data = {
					count:this.withdrawInfo.amount,
					coin_name_en:this.withdrawInfo.currency,
					password:this.verifyInfo.pinCode,
					code:this.verifyInfo.mailCode,
					address:this.withdrawInfo.selectedAddress
				}
				api.withdrawCoin(data)
				.then(res=>{
					this.withdrawInfo.showVarifyModal = false
					this.resetPop()
					if(res.error_code == 1000){
						this.$message('提取成功')
					}
				}).catch(err => {

				})
			},
			calFee(){
				if(this.withdrawInfo.amount <= 0) return
				var data = {
					count:this.withdrawInfo.amount,
					coin_name_en:this.withdrawInfo.currency
				}
				api.calWithdrawFee(data)
				.then(res=>{
					console.log(res)
					if(res.error_code == 1000){
						this.withdrawInfo.rate = res.data.rate
						this.withdrawInfo.fee = res.data.fee
						this.withdrawInfo.realAmount = res.data.realAmount
					}
				}).catch(err => {

				})
			}
		}
	}
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang='scss' scoped>
    li{list-style: none;}
	.top-title {
		font-size: 18px;
		line-height: 32px;
		height: 32px;
		color: #c2c3ca;
		margin-bottom: 15px;
		position: relative;
		text-align: left;
	}

	// .coinlist {
	// 	text-align: center;
	// }


	.combo {
		background-color: #5056E0;
		border: 0px solid;
		height: 35px;
		display: inline-block;
		white-space: nowrap;
		margin: 15px 15px;
		vertical-align: top;
		// line-height: 30px;
	}

	.combo .combo-text {
		font-size: 18px;
		color: #FFFFFF;
		border: 0px;
		line-height: 20px;
		height: 20px;
		margin: 0;
		padding: 0px 2px;
		vertical-align: baseline;
		border: 0px solid;
		height: 35px;
		width: 675px;
	}

	.combo .combo-text1 {
		font-size: 18px;
		color: #FFFFFF;
		border: 0px;
		margin: 0;
		padding: 0px 2px;
		line-height:35px;
		border: 0px solid;
		width: 300px;
		outline: none;
	}
	.combo .combo-text2 {
		font-size: 18px;
		color: #FFFFFF;
		border: 0px;
		margin: 0;
		line-height:35px;
		padding: 0px 2px;
		border: 0px solid;
		width: 300px;
		outline: none;
	}


	.sjx_main{width: 987px;background: #191f27;overflow:hidden;}
	.sjx_section{width: 890px;margin: 0 auto;}
	.section_title{border-bottom: 1px solid #232935;color: #525d6f;font-size: 12px;padding-top:20px;}
	.section_title span{color: #c8cdd3;}
	.quikc-nav{
		font-size: 14px;
		color: #718198;
		float: right;
		.quikc-link,
		.quikc-link:hover,
		.quikc-link:visited{
			color: #718198;
		}
	}


	/* 充币内容 */
	.charge_title{width:915px;height: 53px;border-bottom: 1px solid #232935;}
	.charge_title li{line-height: 53px;float: left;}
	.charge_title li:first-child{width:222px;;text-indent: 30px;}
	.charge_title li:nth-child(2){width: 217px;text-align: center;}
	.charge_title li:nth-child(3){width: 275px;text-align: center;}
	.charge_title li:nth-child(4){width: 174px;text-align: center;}
	.charge li{color:#dee1e8;}
	/* 提币内容*/
	.charge_title2 li:first-child{width: 190px;}
	.charge_title2 li:nth-child(2){width: 118px;}
	.charge_title2 li:nth-child(3){width: 154px;}
	.charge_title2 li:nth-child(4){width: 130px;}
	.charge_title2 li:nth-child(5){width: 170px;text-align: center;}
	.charge_title2 li:nth-child(6){width: 133px;text-align: center;}

	/* 提币弹窗 */
	.login_div_user,.login_div_password{margin-left: auto;
    margin-right: auto;margin-top:25px;width:310px;height:60px;background:url('~@/assets/img/index8.png') no-repeat left;border-bottom:1px solid #efeff0;padding-left:30px;position:relative;}
  .login_div_user input{background:none;text-align:left;font-size: 14px;color:#363a3f;line-height:60px;width:270px;height:60px;border:none;}
  .login_div_password{background:url('~@/assets/img/index9.png') no-repeat left;}
  .login_div_password2{background:url('~@/assets/img/email.png') no-repeat left;}
  .login_div_password input{background:none;text-align:left;font-size: 14px;color:#363a3f;line-height:60px;width:270px;height:60px;border:none;}
  .login_div_other{margin-left: auto;
    margin-right: auto;width:310px;height:20px;margin-top:30px;}
  .login_div_other_left{width:90px;height:20px;font-size: 12px;float:left;color:#555555;}
  .login_div_other_left a{width:12px;height:12px;border:1px solid #4e56f9;margin-top:4px;display:block;float:left;}
  .login_div_other_left_a_click{background:url('~@/assets/img/index10.png') no-repeat left;border:none !important;}
  .login_div_other_left span{height:20px;line-height:20px;float:left;margin-left:10px}
  .login_div_other_right{width:80px;height:20px;text-align:right;font-size: 12px;float:right;color:#8f94fb}
  .pwd_btn{width: 88px;height: 37px;border: 1px solid #5d5dfb;color: #5d5dfb;font-size: 12px;background: #fff;}
</style>
