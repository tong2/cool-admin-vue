<template>
	<cl-crud ref="Crud">
		<cl-row>
			<cl-refresh-btn />
			<cl-multi-delete-btn />
			<cl-flex1 />
			<cl-search-key placeholder="搜索子订单编号或选购商品" />
		</cl-row>

		<cl-row>
			<cl-table ref="Table" />
		</cl-row>

		<cl-row>
			<cl-flex1 />
			<cl-pagination />
		</cl-row>
	</cl-crud>
</template>

<script lang="ts" name="finance-finish" setup>
import { useCrud, useTable } from '@cool-vue/crud';
import { ref } from 'vue';
import { ElMessage, ElLoading } from 'element-plus';
import axios from 'axios';

const Table = useTable({
	columns: [
		{ type: 'selection', width: 60 },
		{ label: 'ID', prop: 'id', minWidth: 80 },
		{ label: '主订单编号', prop: 'main_order_number', minWidth: 120 },
		{ label: '子订单编号', prop: 'sub_order_number', minWidth: 120 },
		{ label: '选购商品', prop: 'selected_product', minWidth: 150 },
		{ label: '商品规格', prop: 'product_specification', minWidth: 150 },
		{ label: '商品数量', prop: 'product_quantity', minWidth: 100 },
		{ label: '商品ID', prop: 'product_id', minWidth: 120 },
		{ label: '商家编码', prop: 'merchant_code', minWidth: 120 },
		{ label: '商品单价', prop: 'product_unit_price', minWidth: 100 },
		{ label: '订单应付金额', prop: 'order_payable_amount', minWidth: 120 },
		{ label: '运费', prop: 'shipping_fee', minWidth: 100 },
		{ label: '优惠总金额', prop: 'total_discount_amount', minWidth: 120 },
		{ label: '平台优惠', prop: 'platform_discount', minWidth: 100 },
		{ label: '商家优惠', prop: 'merchant_discount', minWidth: 100 },
		{ label: '达人优惠', prop: 'talent_discount', minWidth: 100 },
		{ label: '商家改价', prop: 'merchant_price_adjustment', minWidth: 120 },
		{ label: '支付优惠', prop: 'payment_discount', minWidth: 100 },
		{ label: '红包抵扣', prop: 'red_packet_deduction', minWidth: 100 },
		{ label: '支付方式', prop: 'payment_method', minWidth: 120 },
		{ label: '手续费', prop: 'transaction_fee', minWidth: 100 },
		{ label: '收件人', prop: 'recipient_name', minWidth: 120 },
		{ label: '收件人手机号', prop: 'recipient_phone', minWidth: 120 },
		{ label: '省', prop: 'province', minWidth: 100 },
		{ label: '市', prop: 'city', minWidth: 100 },
		{ label: '区', prop: 'district', minWidth: 100 },
		{ label: '街道', prop: 'street', minWidth: 150 },
		{ label: '详细地址', prop: 'detailed_address', minWidth: 200 },
		{ label: '订单提交时间', prop: 'order_submission_time', minWidth: 150 },
		{ label: '支付完成时间', prop: 'payment_completion_time', minWidth: 150 },
		{ label: 'APP渠道', prop: 'app_channel', minWidth: 120 },
		{ label: '流量来源', prop: 'traffic_source', minWidth: 120 },
		{ label: '订单状态', prop: 'order_status', minWidth: 120 },
		{ label: '承诺发货时间', prop: 'promised_shipping_time', minWidth: 150 },
		{ label: '订单类型', prop: 'order_type', minWidth: 120 },
		{ label: '鲁班落地页ID', prop: 'luban_landing_page_id', minWidth: 120 },
		{ label: '达人ID', prop: 'talent_id', minWidth: 120 },
		{ label: '达人昵称', prop: 'talent_nickname', minWidth: 120 },
		{ label: '所属门店ID', prop: 'store_id', minWidth: 120 },
		{ label: '售后状态', prop: 'after_sales_status', minWidth: 120 },
		{ label: '取消原因', prop: 'cancellation_reason', minWidth: 150 },
		{ label: '预约发货时间', prop: 'scheduled_shipping_time', minWidth: 150 },
		{ label: '仓库ID', prop: 'warehouse_id', minWidth: 120 },
		{ label: '仓库名称', prop: 'warehouse_name', minWidth: 150 },
		{ label: '是否安心购', prop: 'is_secure_purchase', minWidth: 120 },
		{ label: '广告渠道', prop: 'ad_channel', minWidth: 120 },
		{ label: '流量类型', prop: 'traffic_type', minWidth: 120 },
		{ label: '流量体裁', prop: 'traffic_format', minWidth: 120 },
		{ label: '流量渠道', prop: 'traffic_channel', minWidth: 120 },
		{ label: '发货主体', prop: 'shipping_entity', minWidth: 120 },
		{ label: '发货主体明细', prop: 'shipping_entity_details', minWidth: 150 },
		{ label: '发货时间', prop: 'shipping_time', minWidth: 150 },
		{ label: '降价类优惠', prop: 'price_reduction_discount', minWidth: 120 },
		{ label: '平台实际承担优惠金额', prop: 'platform_actual_discount', minWidth: 150 },
		{ label: '商家实际承担优惠金额', prop: 'merchant_actual_discount', minWidth: 150 },
		{ label: '达人实际承担优惠金额', prop: 'talent_actual_discount', minWidth: 150 },
		{ label: '预计送达时间', prop: 'estimated_delivery_time', minWidth: 150 },
		{ label: '是否平台仓自流转', prop: 'is_platform_warehouse_transfer', minWidth: 120 },
		{ label: '车型', prop: 'vehicle_type', minWidth: 120 },
		{ label: '商品69码', prop: 'product69_code', minWidth: 120 },
		{ label: '发货SN码', prop: 'shipping_sn_code', minWidth: 120 },
		{ label: '发货IMEI码1', prop: 'shipping_imei_code_1', minWidth: 120 },
		{ label: '发货IMEI码2', prop: 'shipping_imei_code_2', minWidth: 120 },
		{ label: '预约送达时间', prop: 'scheduled_delivery_time', minWidth: 150 },
		{ label: '建议发货时间（起）', prop: 'suggested_shipping_start', minWidth: 150 },
		{ label: '建议发货时间（止）', prop: 'suggested_shipping_end', minWidth: 150 },
		{ label: '物流SN码', prop: 'logistics_sn_code', minWidth: 120 },
		{ label: '物流IMEI码1', prop: 'logistics_imei_code_1', minWidth: 120 },
		{ label: '物流IMEI码2', prop: 'logistics_imei_code_2', minWidth: 120 },
		{ label: '交易时间', prop: 'transaction_time', minWidth: 150 },
		{ label: '单位成本', prop: 'unit_cost', minWidth: 100 },
		{ label: '订单数量', prop: 'order_quantity', minWidth: 100 },
		{ label: '成本', prop: 'total_cost', minWidth: 100 },
		{ label: '重量', prop: 'weight', minWidth: 100 },
		{ label: '省份', prop: 'province_2', minWidth: 100 },
		{ label: '快递费', prop: 'express_fee', minWidth: 100 },
		{ label: '操作费', prop: 'operation_fee', minWidth: 100 },
		{ label: '平台服务费', prop: 'platform_service_fee', minWidth: 120 },
		{ label: '仓库', prop: 'warehouse_2', minWidth: 150 },
		{ label: '状态', prop: 'status', minWidth: 120 },
		{ label: 'ERP成本', prop: 'erp_cost', minWidth: 100 },
		{ label: 'ERP快递费', prop: 'erp_express_fee', minWidth: 100 },
		{ label: '操作', type: 'op', buttons: ['delete'], minWidth: 120 }
	]
});

const getDynamicPrefix = () => {
	return import.meta.env.MODE === 'development' ? '/dev' : '/api';
};

const fetchFinishList = async (params: any) => {
	const loading = ElLoading.service({
		lock: true,
		text: '正在加载数据...',
		background: 'rgba(0, 0, 0, 0.7)'
	});
	try {
		const response = await axios.post(`${getDynamicPrefix()}/order/finance/finish/page2`, {
			...params
		});
		loading.close();
		if (response.data && response.data.code === 1000 && response.data.data) {
			return {
				list: response.data.data.list || [],
				pagination: {
					total: response.data.data.pagination.total || 0,
					page: response.data.data.pagination.page || 1,
					size: response.data.data.pagination.size || 10
				}
			};
		} else {
			ElMessage.error(`获取完成列表失败: ${response.data?.message || '响应格式错误'}`);
			return { list: [], pagination: { total: 0, page: 1, size: 10 } };
		}
	} catch (error: unknown) {
		loading.close();
		let message = '未知错误';
		if (axios.isAxiosError(error)) {
			message = error.response?.data?.message || error.message || '网络错误或服务器无响应';
		}
		ElMessage.error(`获取完成列表失败: ${message}`);
		return { list: [], pagination: { total: 0, page: 1, size: 10 } };
	}
};

const Crud = useCrud(
	{
		service: {
			page: fetchFinishList,
			list: (params: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/finish/list`, params)
					.then(res => res.data.data),
			add: (data: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/finish/add`, data)
					.then(res => res.data),
			delete: (data: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/finish/delete`, data)
					.then(res => res.data),
			update: (data: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/finish/update`, data)
					.then(res => res.data),
			info: (params: any) =>
				axios
					.get(`${getDynamicPrefix()}/order/finance/finish/info/${params.id}`)
					.then(res => res.data.data)
		}
	},
	(app: any) => {
		app.refresh();
	}
);
</script>

<style scoped></style>
