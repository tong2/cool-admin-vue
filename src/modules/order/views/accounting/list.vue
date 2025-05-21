<template>
	<cl-crud ref="Crud">
		<cl-row>
			<cl-refresh-btn />
			<cl-multi-delete-btn />
			<cl-flex1 />
			<cl-search-key placeholder="搜索子订单编号或商品ID" />
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

<script lang="ts" name="finance-accounting" setup>
import { useCrud, useTable } from '@cool-vue/crud';
import { ref } from 'vue';
import { ElMessage, ElLoading } from 'element-plus';
import axios from 'axios';

const Table = useTable({
	columns: [
		{ type: 'selection', width: 60 },
		{ label: 'ID', prop: 'id', minWidth: 80 },
		{ label: '数据时间', prop: 'gen_data_time', minWidth: 150 },
		{ label: '子订单编号', prop: 'sub_order_no', minWidth: 120 },
		{ label: '状态', prop: 'status', minWidth: 100 },
		{ label: '仓库', prop: 'warehouse', minWidth: 120 },
		{ label: '交易日期', prop: 'trade_date', minWidth: 150 },
		{ label: '商品ID', prop: 'product_id', minWidth: 120 },
		{ label: '商家编码', prop: 'merchant_code', minWidth: 120 },
		{ label: '订单数量', prop: 'order_quantity', minWidth: 100 },
		{ label: '订单应付金额', prop: 'order_payable_amount', minWidth: 120 },
		{ label: '实际平台补贴', prop: 'actual_platform_subsidy', minWidth: 120 },
		{ label: '达人实际承担优惠金额', prop: 'influencer_discount_amount', minWidth: 150 },
		{ label: '实销', prop: 'actual_sales', minWidth: 100 },
		{ label: '平台补贴扣费（2%）', prop: 'platform_subsidy_fee', minWidth: 120 },
		{ label: '平台服务费', prop: 'platform_service_fee', minWidth: 120 },
		{ label: '达人佣金', prop: 'influencer_commission', minWidth: 120 },
		{ label: '团长服务费', prop: 'group_leader_service_fee', minWidth: 120 },
		{ label: '成本', prop: 'cost', minWidth: 100 },
		{ label: '快递费', prop: 'shipping_fee', minWidth: 100 },
		{ label: '操作费', prop: 'operation_fee', minWidth: 100 },
		{ label: '利润', prop: 'profit', minWidth: 100 },
		{ label: '毛利率', prop: 'gross_margin', minWidth: 100 },
		{ label: '操作', type: 'op', buttons: ['delete'], minWidth: 120 }
	]
});

const getDynamicPrefix = () => {
	return import.meta.env.MODE === 'development' ? '/dev' : '/api';
};

const fetchAccountingList = async (params: any) => {
	const loading = ElLoading.service({
		lock: true,
		text: '正在加载数据...',
		background: 'rgba(0, 0, 0, 0.7)'
	});
	try {
		const response = await axios.post(`${getDynamicPrefix()}/order/finance/accounting/page2`, {
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
			ElMessage.error(`获取核算列表失败: ${response.data?.message || '响应格式错误'}`);
			return { list: [], pagination: { total: 0, page: 1, size: 10 } };
		}
	} catch (error: unknown) {
		loading.close();
		let message = '未知错误';
		if (axios.isAxiosError(error)) {
			message = error.response?.data?.message || error.message || '网络错误或服务器无响应';
		}
		ElMessage.error(`获取核算列表失败: ${message}`);
		return { list: [], pagination: { total: 0, page: 1, size: 10 } };
	}
};

const Crud = useCrud(
	{
		service: {
			page: fetchAccountingList,
			list: (params: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/accounting/list`, params)
					.then(res => res.data.data),
			add: (data: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/accounting/add`, data)
					.then(res => res.data),
			delete: (data: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/accounting/delete`, data)
					.then(res => res.data),
			update: (data: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/accounting/update`, data)
					.then(res => res.data),
			info: (params: any) =>
				axios
					.get(`${getDynamicPrefix()}/order/finance/accounting/info/${params.id}`)
					.then(res => res.data.data)
		}
	},
	(app: any) => {
		app.refresh();
	}
);
</script>

<style scoped></style>
