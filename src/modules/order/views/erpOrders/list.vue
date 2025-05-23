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
		{ label: 'ID', prop: 'id', minWidth: 80, hidden: true },
		{ label: '订单编号', prop: 'order_number', minWidth: 150, fixed: 'left' },
		{
			label: '数据时间',
			prop: 'gen_data_time',
			minWidth: 120,
			fixed: 'left',
			formatter: row =>
				row.gen_data_time && typeof row.gen_data_time === 'string'
					? row.gen_data_time.slice(0, 10)
					: '' // 非空判断，提取 YYYY-MM-DD
		},
		{ label: '平台类型', prop: 'platform_type', minWidth: 120 },
		{ label: '店铺名称', prop: 'shop_name', minWidth: 150 },
		{ label: '订单来源', prop: 'order_source', minWidth: 120 },
		{ label: '仓库名称', prop: 'warehouse_name', minWidth: 120 },
		{ label: '仓库类型', prop: 'warehouse_type', minWidth: 120 },
		{ label: '原始单号', prop: 'original_order_no', minWidth: 150 },
		{ label: '订单状态', prop: 'order_status', minWidth: 120 },
		{ label: '发货状态', prop: 'shipping_status', minWidth: 120 },
		{ label: '平台发货状态', prop: 'platform_shipping_status', minWidth: 120 },
		{ label: '订单类型', prop: 'order_type', minWidth: 120 },
		{ label: '发货条件', prop: 'shipping_condition', minWidth: 120 },
		{ label: '冻结原因', prop: 'freeze_reason', minWidth: 150 },
		{ label: '退款状态', prop: 'refund_status', minWidth: 120 },
		{ label: '分销类别', prop: 'distribution_category', minWidth: 120 },
		{ label: '分销商名称', prop: 'distributor_name', minWidth: 120 },
		{ label: '分销商编号', prop: 'distributor_code', minWidth: 120 },
		{ label: '分销原始单号', prop: 'distribution_original_order_no', minWidth: 150 },
		{ label: '下单时间', prop: 'order_time', minWidth: 170 },
		{ label: '付款时间', prop: 'payment_time', minWidth: 170 },
		{ label: '发货倒计时', prop: 'shipping_countdown', minWidth: 120 },
		{ label: '买家付款账号', prop: 'buyer_payment_account', minWidth: 150 },
		{ label: '客户网名', prop: 'customer_nickname', minWidth: 120 },
		{ label: '收件人', prop: 'recipient_name', minWidth: 120 },
		{ label: '省市县', prop: 'province_city_county', minWidth: 120 },
		{ label: '地址', prop: 'address', minWidth: 200 },
		{ label: '手机', prop: 'mobile_phone', minWidth: 150 },
		{ label: '电话', prop: 'telephone', minWidth: 150 },
		{ label: '邮编', prop: 'postcode', minWidth: 100 },
		{ label: '区域', prop: 'area', minWidth: 100 },
		{ label: '大头笔', prop: 'big_pen', minWidth: 120 },
		{ label: '派送时间', prop: 'dispatch_time', minWidth: 170 },
		{ label: '物流公司', prop: 'logistics_company', minWidth: 120 },
		{ label: '物流单号', prop: 'logistics_number', minWidth: 150 },
		{ label: '买家留言', prop: 'buyer_message', minWidth: 200 },
		{ label: '客服备注', prop: 'customer_service_remark', minWidth: 200 },
		{ label: '标旗', prop: 'flag', minWidth: 100 },
		{ label: '打印备注', prop: 'print_remark', minWidth: 200 },
		{ label: '货品种类数', prop: 'product_variety_count', minWidth: 100 },
		{ label: '货品总数', prop: 'product_total_count', minWidth: 100 },
		{ label: '货品总额', prop: 'product_total_amount', minWidth: 120 },
		{ label: '邮资', prop: 'shipping_fee', minWidth: 100 },
		{ label: '其它费用', prop: 'other_fees', minWidth: 100 },
		{ label: '优惠', prop: 'discount', minWidth: 120 },
		{ label: '应收金额', prop: 'receivable_amount', minWidth: 120 },
		{ label: '销项税', prop: 'output_tax', minWidth: 100 },
		{ label: '款到发货金额', prop: 'payment_on_delivery_amount', minWidth: 120 },
		{ label: 'COD金额', prop: 'cod_amount', minWidth: 100 },
		{ label: '买家COD费用', prop: 'buyer_cod_fee', minWidth: 100 },
		{ label: '佣金', prop: 'commission', minWidth: 100 },
		{ label: '货品预估成本', prop: 'product_estimated_cost', minWidth: 120 },
		{ label: '邮资预估成本', prop: 'shipping_estimated_cost', minWidth: 120 },
		{ label: '已付金额', prop: 'paid_amount', minWidth: 120 },
		{ label: '预估重量', prop: 'estimated_weight', minWidth: 100 },
		{ label: '预估毛利', prop: 'estimated_gross_profit', minWidth: 120 },
		{ label: '发票类型', prop: 'invoice_type', minWidth: 120 },
		{ label: '发票抬头', prop: 'invoice_title', minWidth: 150 },
		{ label: '发票内容', prop: 'invoice_content', minWidth: 200 },
		{ label: '业务员', prop: 'salesman', minWidth: 120 },
		{ label: '审核人', prop: 'auditor', minWidth: 120 },
		{ label: '财审人', prop: 'financial_auditor', minWidth: 120 },
		{ label: '签出人', prop: 'sign_out_person', minWidth: 120 },
		{ label: '出库单号', prop: 'outbound_order_no', minWidth: 150 },
		{ label: '标记名称', prop: 'mark_name', minWidth: 120 },
		{ label: '处理天数', prop: 'processing_days', minWidth: 100 },
		{ label: '货品商家编码', prop: 'product_merchant_code', minWidth: 150 },
		{ label: '原始货品数量', prop: 'original_product_quantity', minWidth: 100 },
		{ label: '原始货品种类数', prop: 'original_product_variety_count', minWidth: 100 },
		{ label: '递交时间', prop: 'submission_time', minWidth: 170 },
		{ label: '币种', prop: 'currency', minWidth: 100 },
		{ label: '线上包裹拆分数', prop: 'online_package_split_count', minWidth: 100 },
		{ label: '激活时间', prop: 'activation_time', minWidth: 170 },
		{ label: '已开具发票', prop: 'invoice_issued', minWidth: 100 },
		{ label: '体积', prop: 'volume', minWidth: 100 },
		{ label: '订单标签', prop: 'order_tags', minWidth: 200 },
		{ label: '订单异常', prop: 'order_exception', minWidth: 150 },
		{ label: '便签', prop: 'note', minWidth: 200 },
		{ label: '证件号码', prop: 'id_number', minWidth: 150 },
		{ label: '买家实付', prop: 'buyer_actual_payment', minWidth: 120 },
		{ label: '最晚送达时间', prop: 'latest_delivery_time', minWidth: 170 },
		{ label: '平台标签', prop: 'platform_tags', minWidth: 200 },
		{ label: '操作', type: 'op', buttons: ['delete'], minWidth: 120 }
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
		let filename = 'erp订单表.xlsx';
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
