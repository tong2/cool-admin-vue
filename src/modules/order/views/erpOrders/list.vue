<template>
	<cl-crud ref="Crud">
		<cl-row>
			<cl-refresh-btn />
			<cl-multi-delete-btn />
			<el-button type="primary" @click="handleExport">导出</el-button>
			<el-button type="success" @click="handleImport">导入</el-button>
			<cl-flex1 />
			<cl-search-key placeholder="搜索订单编号" />
		</cl-row>

		<cl-row>
			<cl-table ref="Table" />
		</cl-row>

		<cl-row>
			<cl-flex1 />
			<cl-pagination />
		</cl-row>

		<cl-upsert ref="Upsert" />

		<el-dialog title="导入ERP订单" v-model="importDialogVisible" width="30%">
			<el-upload
				:action="importAction"
				:before-upload="beforeUpload"
				:on-success="onImportSuccess"
				:on-error="onImportError"
				accept=".xlsx"
			>
				<el-button type="primary">选择 Excel 文件</el-button>
			</el-upload>
		</el-dialog>
	</cl-crud>
</template>

<script lang="ts" name="erp-order-list" setup>
import { useCrud, useTable, useUpsert } from '@cool-vue/crud';
import { ref } from 'vue';
import { ElMessage, ElLoading } from 'element-plus';
import axios from 'axios';

const Table = useTable({
	columns: [
		{ type: 'selection', width: 60 },
		{ label: 'ID', prop: 'id', minWidth: 80 },
		{ label: '订单编号', prop: 'order_number', minWidth: 150 },
		{ label: '店铺', prop: 'shop', minWidth: 150 },
		{ label: '订单来源', prop: 'order_source', minWidth: 120 },
		{ label: '仓库', prop: 'warehouse', minWidth: 120 },
		{ label: '子单原始单号', prop: 'sub_order_original_number', minWidth: 150 },
		{ label: '订单状态', prop: 'order_status', minWidth: 120 },
		{ label: '订单类型', prop: 'order_type', minWidth: 120 },
		{ label: '货到付款', prop: 'cash_on_delivery', minWidth: 100 },
		{ label: '订单退款状态', prop: 'order_refund_status', minWidth: 120 },
		{ label: '交易时间', prop: 'transaction_time', minWidth: 170 },
		{ label: '付款时间', prop: 'payment_time', minWidth: 170 },
		{ label: '发货时间', prop: 'delivery_time', minWidth: 170 },
		{ label: '客户网名', prop: 'customer_nickname', minWidth: 120 },
		{ label: '收件人', prop: 'recipient_name', minWidth: 120 },
		{ label: '收货地区', prop: 'recipient_region', minWidth: 120 },
		{ label: '收货地址', prop: 'recipient_address', minWidth: 200 },
		{ label: '收件人手机', prop: 'recipient_phone', minWidth: 150 },
		{ label: '分销商', prop: 'distributor', minWidth: 120 },
		{ label: '客户编号', prop: 'customer_number', minWidth: 120 },
		{ label: '邮编', prop: 'postcode', minWidth: 100 },
		{ label: '区域', prop: 'area', minWidth: 100 },
		{ label: '物流公司', prop: 'logistics_company', minWidth: 120 },
		{ label: '物流单号', prop: 'logistics_number', minWidth: 150 },
		{ label: '买家留言', prop: 'buyer_message', minWidth: 200 },
		{ label: '客服备注', prop: 'customer_service_remark', minWidth: 200 },
		{ label: '订单支付金额', prop: 'order_payment_amount', minWidth: 120 },
		{ label: '邮费', prop: 'postage', minWidth: 100 },
		{ label: '订单总优惠', prop: 'order_total_discount', minWidth: 120 },
		{ label: '应收金额', prop: 'receivable_amount', minWidth: 120 },
		{ label: '货品名称', prop: 'product_name', minWidth: 150 },
		{ label: '规格名称', prop: 'specification_name', minWidth: 120 },
		{ label: '下单数量', prop: 'order_quantity', minWidth: 100 },
		{ label: '成交价', prop: 'transaction_price', minWidth: 100 },
		{ label: '操作', type: 'op', buttons: ['delete'], minWidth: 120 }
	]
});

const Upsert = useUpsert({
	dialog: { width: '800px' },
	items: [
		{
			prop: 'order_number',
			label: '订单编号',
			component: { name: 'el-input' },
			required: true
		},
		{ prop: 'shop', label: '店铺', component: { name: 'el-input' } },
		{ prop: 'order_source', label: '订单来源', component: { name: 'el-input' } },
		{ prop: 'warehouse', label: '仓库', component: { name: 'el-input' } },
		{
			prop: 'sub_order_original_number',
			label: '子单原始单号',
			component: { name: 'el-input' }
		},
		{ prop: 'order_status', label: '订单状态', component: { name: 'el-input' } },
		{ prop: 'order_type', label: '订单类型', component: { name: 'el-input' } },
		{ prop: 'cash_on_delivery', label: '货到付款', component: { name: 'el-input' } },
		{ prop: 'order_refund_status', label: '订单退款状态', component: { name: 'el-input' } },
		{
			prop: 'transaction_time',
			label: '交易时间',
			component: { name: 'el-date-picker', props: { type: 'datetime' } }
		},
		{
			prop: 'payment_time',
			label: '付款时间',
			component: { name: 'el-date-picker', props: { type: 'datetime' } }
		},
		{
			prop: 'delivery_time',
			label: '发货时间',
			component: { name: 'el-date-picker', props: { type: 'datetime' } }
		},
		{ prop: 'customer_nickname', label: '客户网名', component: { name: 'el-input' } },
		{
			prop: 'recipient_name',
			label: '收件人',
			component: { name: 'el-input' },
			required: true
		},
		{ prop: 'recipient_region', label: '收货地区', component: { name: 'el-input' } },
		{ prop: 'recipient_address', label: '收货地址', component: { name: 'el-input' } },
		{ prop: 'recipient_phone', label: '收件人手机', component: { name: 'el-input' } },
		{ prop: 'distributor', label: '分销商', component: { name: 'el-input' } },
		{ prop: 'customer_number', label: '客户编号', component: { name: 'el-input' } },
		{ prop: 'postcode', label: '邮编', component: { name: 'el-input' } },
		{ prop: 'area', label: '区域', component: { name: 'el-input' } },
		{ prop: 'logistics_company', label: '物流公司', component: { name: 'el-input' } },
		{ prop: 'logistics_number', label: '物流单号', component: { name: 'el-input' } },
		{
			prop: 'buyer_message',
			label: '买家留言',
			component: { name: 'el-input', props: { type: 'textarea' } }
		},
		{
			prop: 'customer_service_remark',
			label: '客服备注',
			component: { name: 'el-input', props: { type: 'textarea' } }
		},
		{
			prop: 'order_payment_amount',
			label: '订单支付金额',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{
			prop: 'postage',
			label: '邮费',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{
			prop: 'order_total_discount',
			label: '订单总优惠',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{
			prop: 'receivable_amount',
			label: '应收金额',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{ prop: 'product_name', label: '货品名称', component: { name: 'el-input' } },
		{ prop: 'specification_name', label: '规格名称', component: { name: 'el-input' } },
		{
			prop: 'order_quantity',
			label: '下单数量',
			component: { name: 'el-input-number', props: { min: 0 } }
		},
		{
			prop: 'transaction_price',
			label: '成交价',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		}
	]
});

const getDynamicPrefix = () => {
	return import.meta.env.MODE === 'development' ? '/dev' : '/api';
};

const importDialogVisible = ref(false);
const importAction = `${getDynamicPrefix()}/order/finance/erpOrders/import`;

const fetchErpOrderList = async (params: any) => {
	const loading = ElLoading.service({
		lock: true,
		text: '正在加载数据...',
		background: 'rgba(0, 0, 0, 0.7)'
	});
	try {
		const response = await axios.post(`${getDynamicPrefix()}/order/finance/erpOrders/page2`, {
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
			ElMessage.error(`获取ERP订单列表失败: ${response.data?.message || '响应格式错误'}`);
			return { list: [], pagination: { total: 0, page: 1, size: 10 } };
		}
	} catch (error: unknown) {
		loading.close();
		let message = '未知错误';
		if (axios.isAxiosError(error)) {
			message = error.response?.data?.message || error.message || '网络错误或服务器无响应';
		}
		ElMessage.error(`获取ERP订单列表失败: ${message}`);
		return { list: [], pagination: { total: 0, page: 1, size: 10 } };
	}
};

const handleExport = async () => {
	const loading = ElLoading.service({
		lock: true,
		text: '正在导出数据...',
		background: 'rgba(0, 0, 0, 0.7)'
	});
	try {
		const params = Crud.value?.search?.params || {};
		const response = await axios.get(`${getDynamicPrefix()}/order/finance/erpOrders/export`, {
			params,
			responseType: 'blob'
		});

		const url = window.URL.createObjectURL(new Blob([response.data]));
		const link = document.createElement('a');
		link.href = url;
		const contentDisposition = response.headers['content-disposition'];
		let filename = 'finance-erp-orders.xlsx';
		if (contentDisposition) {
			const filenameMatch = contentDisposition.match(/filename="?(.+)"?/i);
			if (filenameMatch && filenameMatch.length > 1) {
				filename = filenameMatch[1];
			}
		}
		link.setAttribute('download', filename);
		document.body.appendChild(link);
		link.click();
		document.body.removeChild(link);
		window.URL.revokeObjectURL(url);

		ElMessage.success('导出成功');
		loading.close();
	} catch (error: unknown) {
		loading.close();
		let message = '未知错误';
		if (axios.isAxiosError(error)) {
			if (
				error.response &&
				error.response.data instanceof Blob &&
				error.response.data.type.includes('json')
			) {
				try {
					const errorJson = JSON.parse(await error.response.data.text());
					message = errorJson.message || '导出失败';
				} catch (parseError) {
					message = '未知导出错误';
				}
			} else {
				message =
					error.response?.data?.message || error.message || '网络错误或服务器无响应';
			}
		}
		ElMessage.error(`导出失败: ${message}`);
	}
};

const handleImport = () => {
	importDialogVisible.value = true;
};

const beforeUpload = (file: File): boolean => {
	const isXlsx =
		file.type === 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet';
	const isXls = file.type === 'application/vnd.ms-excel';
	if (!isXlsx && !isXls) {
		ElMessage.error('请上传 .xlsx 或 .xls 文件');
		return false;
	}
	return true;
};

const onImportSuccess = (response: any) => {
	if (response && response.code === 1000) {
		ElMessage.success(
			`导入成功${response.data?.count ? '，处理 ' + response.data.count + ' 条记录' : ''}`
		);
		importDialogVisible.value = false;
		Crud.value?.refresh();
	} else {
		ElMessage.error(`导入失败: ${response?.message || '服务器返回错误'}`);
	}
};

const onImportError = (error: any) => {
	let message = '上传失败';
	try {
		const errorResponse = JSON.parse(error.message || '{}');
		message = errorResponse.message || message;
	} catch (e) {
		message = error.message || '未知上传错误';
	}
	ElMessage.error(`导入失败: ${message}`);
};

const Crud = useCrud(
	{
		service: {
			page: fetchErpOrderList,
			list: (params: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/erpOrders/list`, params)
					.then(res => res.data.data),
			add: (data: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/erpOrders/add`, data)
					.then(res => res.data),
			delete: (data: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/erpOrders/delete`, data)
					.then(res => res.data),
			update: (data: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/erpOrders/update`, data)
					.then(res => res.data),
			info: (params: any) =>
				axios
					.get(`${getDynamicPrefix()}/order/finance/erpOrders/info/${params.id}`)
					.then(res => res.data.data)
		}
	},
	(app: any) => {
		app.refresh();
	}
);
</script>

<style scoped></style>
