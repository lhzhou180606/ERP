<template>
<div class="willorderInfo-box">
	<h3 class="willorderInfo-title">{{quotationDetailObject.productcode}}</h3>
	<div class="btn-box">
		<el-button type="primary" class="btn-w80" @click="save">保存</el-button>
		<el-button class="btn-w80" @click="close">取消</el-button>
	</div>
	<div class="info-item">
		<h4>询价信息</h4>
		<div class="left">
			<ul class="info">
				<li>产品编码：<span class="font-color-5">{{quotationDetailObject.productcode}}</span></li>
				<li>产品型号/SKU：<span class="font-color-5">{{quotationDetailObject.productmodel}}</span></li>
				<li>附件名/图纸号：<span class="font-color-5">{{quotationDetailObject.attachmentindex | checkNull}}</span></li>
				<li>询价数量：<span class="font-color-5">{{quotationDetailObject.orderamount | checkNull}}</span></li>
			</ul>
		</div>
		<div class="right">
			<ul class="info">
				<li>品牌：<span class="font-color-5">{{quotationDetailObject.productbrand | checkNull}}</span></li>
				<li>产品名称：<span class="font-color-5">{{quotationDetailObject.productname | checkNull}}</span></li>
				<li>询价单位：<span class="font-color-5">{{quotationDetailObject.unit | checkNull}}</span></li>
			</ul>
		</div>
	</div>
	<div class="info-item supplier-box">
		<el-form :model="quotationDetailObject" label-position="left">
			<h4>报价信息</h4>
			<div class="left">
				<el-form-item label="是否能做" label-width="104px" class="required-icon">
					<el-checkbox class="info-item-checkbox" v-model="quotationDetailObject.iscando" true-label="1" false-label="0">能做</el-checkbox>
				</el-form-item>
				<el-form-item label="货期（天）" label-width="104px" class="required-icon">
					<el-input-number class="info-item-input-number" :min="0" v-model="quotationDetailObject.deliverytime" :debounce="50" :max="99999"></el-input-number>
				</el-form-item>
				<el-form-item label="未税单价" label-width="104px">
					<div class="price-warpper">
						<el-input class="info-item-input" :disabled="true" v-model="price"></el-input>
						<p class="price-tip">系统默认未税单价=材料费+加工费+表面处理</p>
					</div>
				</el-form-item>
			</div>
			<div class="right">
				<el-form-item label="最小订购量" label-width="97px">
					<el-input-number class="info-item-input-number" :min="0" v-model="quotationDetailObject.minamount" :debounce="50" :max="99999999"></el-input-number>
				</el-form-item>
				<el-form-item label="是否包含运费" label-width="97px">
					<el-checkbox class="info-item-checkbox" v-model="quotationDetailObject.issendcost" true-label="1" false-label="0">包含</el-checkbox>
				</el-form-item>
			</div>
			<div class="clear"></div>
			<div class="pricenotax-box">
				<div class="left">
					<el-form-item label="材料费" label-width="97px">
						<el-input class="info-item-input" placeholder="请输入单个产品的材料费" v-model="quotationDetailObject.materialcost" @blur="blur($event, 'materialcost')" @keyup.native="checkValue($event, 'materialcost')" :maxlength="maxlength8"></el-input>
					</el-form-item>
					<el-form-item label="表面处理费" label-width="97px">
						<el-input class="info-item-input" placeholder="请输入单个产品的表面处理费" v-model="quotationDetailObject.dealwithcost" @blur="blur($event, 'dealwithcost')" @keyup.native="checkValue($event, 'dealwithcost')" :maxlength="maxlength8"></el-input>
					</el-form-item>
				</div>
				<div class="right">
					<el-form-item label="加工费" label-width="104px">
						<el-input class="info-item-input" placeholder="请输入单个产品的加工费" v-model="quotationDetailObject.workcost" @blur="blur($event, 'workcost')" @keyup.native="checkValue($event, 'workcost')" :maxlength="maxlength8"></el-input>
					</el-form-item>
				</div>
			</div>
			<div class="left">
				<el-form-item label="税率" label-width="97px">
					<el-select v-model="quotationDetailObject.taxrate" class="info-item-input" placeholder="请选择" @change="handleChange">
						<el-option v-for="item in taxrates" :key="item.name" :label="item.text" :value="item.name">
						</el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="含税金额" label-width="97px">
					<el-input class="info-item-input" v-model="quotationDetailObject.taxamount" :disabled="true"></el-input>
				</el-form-item>
			</div>
			<div class="right">
				<el-form-item label="含税单价" label-width="104px" class="required-icon">
					<el-input class="info-item-input" :disabled="price>0" v-model="quotationDetailObject.pricewithtax" @blur="blur($event, 'taxprice')" @keyup.native="checkValue($event, 'taxprice')" @change="computTaxamount" :maxlength="maxlength11"></el-input>
				</el-form-item>
				<el-form-item label="备注" label-width="104px">
					<el-input class="info-item-input" type="textarea" autosize v-model="quotationDetailObject.remark"></el-input>
				</el-form-item>
			</div>
		</el-form>
	</div>
</div>
</template>
<script type="text/javascript">
import api from 'api/request'
export default {
	name: 'addquotationDetail',
	props: {
		quotationDetailObject: {
			type: Object
		}
	},
	watch: {
		price(val) {
			let price = val * (1 + this.quotationDetailObject.taxrate / 100)
			this.quotationDetailObject.taxprice = price.toFixed(2)
		}
	},
	data() {
		return {
			taxrates: [],
      supplierShow: false,
			maxlength8: 8,
			maxlength11: 11
		}
	},
	computed: {
		price() {
			let price = (this.quotationDetailObject.materialcost * 10000 + this.quotationDetailObject.dealwithcost * 10000 + this.quotationDetailObject.workcost * 10000) / 10000
			price = isNaN(price) ? '0' : price
			return price
		}
	},
	methods: {
		checkValue(e, id) {
			let value = e.target.value
			if (isNaN(value) || value != '') {
				this.quotationDetailObject[id] = value.replace(/[^\d|.]/g, '')
			}
		},
		blur(e, id) {
			let value = e.target.value
			if (value == '' || isNaN(value)) {
				return
			}
			if (value > 99999999) {
				value = 99999999
			}
			this.quotationDetailObject[id] = parseFloat(value).toFixed(2)
		},
		handleChange(val) {
			if (this.price > 0) {
				let taxprice = this.price * (1 + this.quotationDetailObject.taxrate / 100)
				this.quotationDetailObject.taxprice = taxprice.toFixed(2)
			}
			let mount = this.quotationDetailObject.taxprice * this.quotationDetailObject.orderamount
			this.quotationDetailObject.taxamount = mount.toFixed(2)
		},
		// 计算含税金额
		computTaxamount(val) {
			if (!isNaN(val)) {
				let mount = val * this.quotationDetailObject.orderamount
				this.quotationDetailObject.taxamount = mount.toFixed(2)
			}
		},
		// 保存
		save() {
			this.$emit('listenquotationDetail', '')
		},
		// 取消
		close() {
			this.$emit('listenquotationDetail', '')
		}
	},
	filters: {
		checkNull(val) {
			val = val == null || val == '' ? '--' : val
			return val
		}
	},
	created() {
		let parms = { typenumbers: 'TaxRate' }
		api.supplier.getDictByTypeNumbers(parms).then(res => {
			if (res.data.code == 200) {
				for (var i = 0; i < res.data.data.TaxRate.length; i++) {
					res.data.data.TaxRate[i].name = parseInt(res.data.data.TaxRate[i].name)
					res.data.data.TaxRate[i].text = res.data.data.TaxRate[i].name + '%'
				}
				this.taxrates = res.data.data.TaxRate
			}
		})
	}
}
</script>
<style lang="stylus">
  @import '~common/style/supplierDetail.styl'
</style>
