<template>
	<cl-crud ref="Crud">
		<cl-row>
			<cl-refresh-btn />
			<cl-multi-delete-btn />
			<el-button type="primary" @click="handleExport">导出</el-button>
			<el-button type="success" @click="handleImport">导入</el-button>
			<cl-flex1 />
			<cl-search-key placeholder="搜索子订单号" />
		</cl-row>

		<cl-row>
			<cl-table ref="Table" />
		</cl-row>

		<cl-row>
			<cl-flex1 />
			<cl-pagination />
		</cl-row>

		<cl-upsert ref="Upsert" />

		<el-dialog title="导入订单" v-model="importDialogVisible" width="30%">
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

<script lang="ts" name="order-list" setup>
import { useCrud, useTable, useUpsert } from '@cool-vue/crud';
import { useCool } from '/@/cool';
import { ref } from 'vue';
import { ElMessage, ElLoading } from 'element-plus';
import axios from 'axios'; // Make sure axios is installed and imported

// cool-vue services (if used alongside axios, ensure consistency)
const { service } = useCool(); // This seems unused if using direct axios calls below

// cl-table configuration
const Table = useTable({
	columns: [
		{ type: 'selection', width: 60 },
		{ label: 'ID', prop: 'id', minWidth: 80 },
		{ label: '主订单号', prop: 'main_order_number', minWidth: 150 },
		{ label: '子订单号', prop: 'sub_order_number', minWidth: 150 },
		{ label: '选购商品', prop: 'selected_goods', minWidth: 200 },
		{ label: '商品规格', prop: 'product_specification', minWidth: 120 },
		{ label: '商品数量', prop: 'product_quantity', minWidth: 100 },
		{ label: '商品ID', prop: 'product_id', minWidth: 120 },
		{ label: '商家编码', prop: 'merchant_code', minWidth: 120 },
		{ label: '商品单价', prop: 'product_price', minWidth: 100 },
		{ label: '订单应付金额', prop: 'order_payable_amount', minWidth: 120 },
		{ label: '运费', prop: 'shipping_fee', minWidth: 100 },
		{ label: '优惠总金额', prop: 'total_discount_amount', minWidth: 120 },
		{ label: '平台优惠', prop: 'platform_discount', minWidth: 150 },
		{ label: '商家优惠', prop: 'merchant_discount', minWidth: 100 },
		{ label: '达人优惠', prop: 'influencer_discount', minWidth: 100 },
		{ label: '商家改价', prop: 'merchant_price_adjustment', minWidth: 120 },
		{ label: '支付优惠', prop: 'payment_discount', minWidth: 100 },
		{ label: '红包抵扣', prop: 'red_envelope_deduction', minWidth: 100 },
		{ label: '支付方式', prop: 'payment_method', minWidth: 120 },
		{ label: '手续费', prop: 'transaction_fee', minWidth: 100 },
		{ label: '收件人', prop: 'recipient_name', minWidth: 120 },
		{ label: '收件人手机号', prop: 'recipient_phone_number', minWidth: 150 },
		{ label: '省', prop: 'province', minWidth: 100 },
		{ label: '市', prop: 'city', minWidth: 100 },
		{ label: '区', prop: 'district', minWidth: 100 },
		{ label: '街道', prop: 'street', minWidth: 120 },
		{ label: '详细地址', prop: 'detailed_address', minWidth: 200 },
		{ label: '是否修改过地址', prop: 'is_address_modified', minWidth: 120 },
		{ label: '买家留言', prop: 'buyer_message', minWidth: 200 },
		{ label: '订单提交时间', prop: 'order_submission_time', minWidth: 170 },
		{ label: '旗帜颜色', prop: 'flag_color', minWidth: 100 },
		{ label: '商家备注', prop: 'merchant_remark', minWidth: 200 },
		{ label: '订单完成时间', prop: 'order_completion_time', minWidth: 170 },
		{ label: '支付完成时间', prop: 'payment_completion_time', minWidth: 170 },
		{ label: 'APP渠道', prop: 'app_channel', minWidth: 120 },
		{ label: '流量来源', prop: 'traffic_source', minWidth: 120 },
		{ label: '订单状态', prop: 'order_status', minWidth: 120 },
		{ label: '承诺发货时间', prop: 'promised_delivery_time', minWidth: 170 },
		{ label: '订单类型', prop: 'order_type', minWidth: 120 },
		{ label: '鲁班落地页ID', prop: 'luban_page_id', minWidth: 120 },
		{ label: '达人ID', prop: 'influencer_id', minWidth: 120 },
		{ label: '达人昵称', prop: 'influencer_nickname', minWidth: 120 },
		{ label: '所属门店ID', prop: 'store_id', minWidth: 120 },
		{ label: '售后状态', prop: 'after_sales_status', minWidth: 120 },
		{ label: '取消原因', prop: 'cancellation_reason', minWidth: 200 },
		{ label: '预约发货时间', prop: 'scheduled_delivery_time', minWidth: 170 },
		{ label: '仓库ID', prop: 'warehouse_id', minWidth: 120 },
		{ label: '仓库名称', prop: 'warehouse_name', minWidth: 120 },
		{ label: '是否安心购', prop: 'is_safe_purchase', minWidth: 120 },
		{ label: '广告渠道', prop: 'ad_channel', minWidth: 120 },
		{ label: '流量类型', prop: 'traffic_type', minWidth: 120 },
		{ label: '流量体裁', prop: 'traffic_format', minWidth: 120 },
		{ label: '流量渠道', prop: 'traffic_channel', minWidth: 120 },
		{ label: '发货主体', prop: 'delivery_entity', minWidth: 120 },
		{ label: '发货主体明细', prop: 'delivery_entity_details', minWidth: 200 },
		{ label: '发货时间', prop: 'delivery_time', minWidth: 170 },
		{ label: '降价类优惠', prop: 'price_reduction_discount', minWidth: 120 },
		{ label: '平台实际优惠', prop: 'platform_actual_discount', minWidth: 120 },
		{ label: '商家实际优惠', prop: 'merchant_actual_discount', minWidth: 120 },
		{ label: '达人实际优惠', prop: 'influencer_actual_discount', minWidth: 120 },
		{ label: '预计送达时间', prop: 'estimated_delivery_time', minWidth: 170 },
		{ label: '是否平台仓自流转', prop: 'is_platform_warehouse_auto_transfer', minWidth: 120 },
		{ label: '车型', prop: 'vehicle_type', minWidth: 120 },
		{ label: '预约送达时间', prop: 'scheduled_delivery_arrival_time', minWidth: 170 },
		{ label: '建议发货时间（起）', prop: 'suggested_delivery_start_time', minWidth: 170 },
		{ label: '建议发货时间（止）', prop: 'suggested_delivery_end_time', minWidth: 170 },
		{ label: '操作', type: 'op', buttons: ['delete'], minWidth: 120 }
	]
});

// cl-upsert configuration
const Upsert = useUpsert({
	dialog: { width: '800px' },
	items: [
		{
			prop: 'main_order_number',
			label: '主订单号',
			component: { name: 'el-input' },
			required: true
		},
		{ prop: 'sub_order_number', label: '子订单号', component: { name: 'el-input' } },
		{
			prop: 'selected_goods',
			label: '选购商品',
			component: { name: 'el-input', props: { type: 'textarea' } }
		},
		{ prop: 'product_specification', label: '商品规格', component: { name: 'el-input' } },
		{
			prop: 'product_quantity',
			label: '商品数量',
			component: { name: 'el-input-number', props: { min: 0 } }
		},
		{ prop: 'product_id', label: '商品ID', component: { name: 'el-input' } },
		{ prop: 'merchant_code', label: '商家编码', component: { name: 'el-input' } },
		{
			prop: 'product_price',
			label: '商品单价',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{
			prop: 'order_payable_amount',
			label: '订单应付金额',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } },
			required: true
		},
		{
			prop: 'shipping_fee',
			label: '运费',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{
			prop: 'total_discount_amount',
			label: '优惠总金额',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{ prop: 'platform_discount', label: '平台优惠', component: { name: 'el-input' } },
		{
			prop: 'merchant_discount',
			label: '商家优惠',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{
			prop: 'influencer_discount',
			label: '达人优惠',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{
			prop: 'merchant_price_adjustment',
			label: '商家改价',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{
			prop: 'payment_discount',
			label: '支付优惠',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{
			prop: 'red_envelope_deduction',
			label: '红包抵扣',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{ prop: 'payment_method', label: '支付方式', component: { name: 'el-input' } },
		{
			prop: 'transaction_fee',
			label: '手续费',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{
			prop: 'recipient_name',
			label: '收件人',
			component: { name: 'el-input' },
			required: true
		},
		{ prop: 'recipient_phone_number', label: '收件人手机号', component: { name: 'el-input' } },
		{ prop: 'province', label: '省', component: { name: 'el-input' } },
		{ prop: 'city', label: '市', component: { name: 'el-input' } },
		{ prop: 'district', label: '区', component: { name: 'el-input' } },
		{ prop: 'street', label: '街道', component: { name: 'el-input' } },
		{ prop: 'detailed_address', label: '详细地址', component: { name: 'el-input' } },
		{ prop: 'is_address_modified', label: '是否修改过地址', component: { name: 'el-input' } },
		{
			prop: 'buyer_message',
			label: '买家留言',
			component: { name: 'el-input', props: { type: 'textarea' } }
		},
		{
			prop: 'order_submission_time',
			label: '订单提交时间',
			component: { name: 'el-date-picker', props: { type: 'datetime' } }
		},
		{ prop: 'flag_color', label: '旗帜颜色', component: { name: 'el-input' } },
		{
			prop: 'merchant_remark',
			label: '商家备注',
			component: { name: 'el-input', props: { type: 'textarea' } }
		},
		{
			prop: 'order_completion_time',
			label: '订单完成时间',
			component: { name: 'el-date-picker', props: { type: 'datetime' } }
		},
		{
			prop: 'payment_completion_time',
			label: '支付完成时间',
			component: { name: 'el-date-picker', props: { type: 'datetime' } }
		},
		{ prop: 'app_channel', label: 'APP渠道', component: { name: 'el-input' } },
		{ prop: 'traffic_source', label: '流量来源', component: { name: 'el-input' } },
		{ prop: 'order_status', label: '订单状态', component: { name: 'el-input' } },
		{
			prop: 'promised_delivery_time',
			label: '承诺发货时间',
			component: { name: 'el-date-picker', props: { type: 'datetime' } }
		},
		{ prop: 'order_type', label: '订单类型', component: { name: 'el-input' } },
		{ prop: 'luban_page_id', label: '鲁班落地页ID', component: { name: 'el-input' } },
		{ prop: 'influencer_id', label: '达人ID', component: { name: 'el-input' } },
		{ prop: 'influencer_nickname', label: '达人昵称', component: { name: 'el-input' } },
		{ prop: 'store_id', label: '所属门店ID', component: { name: 'el-input' } },
		{ prop: 'after_sales_status', label: '售后状态', component: { name: 'el-input' } },
		{
			prop: 'cancellation_reason',
			label: '取消原因',
			component: { name: 'el-input', props: { type: 'textarea' } }
		},
		{
			prop: 'scheduled_delivery_time',
			label: '预约发货时间',
			component: { name: 'el-date-picker', props: { type: 'datetime' } }
		},
		{ prop: 'warehouse_id', label: '仓库ID', component: { name: 'el-input' } },
		{ prop: 'warehouse_name', label: '仓库名称', component: { name: 'el-input' } },
		{ prop: 'is_safe_purchase', label: '是否安心购', component: { name: 'el-input' } },
		{ prop: 'ad_channel', label: '广告渠道', component: { name: 'el-input' } },
		{ prop: 'traffic_type', label: '流量类型', component: { name: 'el-input' } },
		{ prop: 'traffic_format', label: '流量体裁', component: { name: 'el-input' } },
		{ prop: 'traffic_channel', label: '流量渠道', component: { name: 'el-input' } },
		{ prop: 'delivery_entity', label: '发货主体', component: { name: 'el-input' } },
		{
			prop: 'delivery_entity_details',
			label: '发货主体明细',
			component: { name: 'el-input', props: { type: 'textarea' } }
		},
		{
			prop: 'delivery_time',
			label: '发货时间',
			component: { name: 'el-date-picker', props: { type: 'datetime' } }
		},
		{
			prop: 'price_reduction_discount',
			label: '降价类优惠',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{
			prop: 'platform_actual_discount',
			label: '平台实际优惠',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{
			prop: 'merchant_actual_discount',
			label: '商家实际优惠',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{
			prop: 'influencer_actual_discount',
			label: '达人实际优惠',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{
			prop: 'estimated_delivery_time',
			label: '预计送达时间',
			component: { name: 'el-date-picker', props: { type: 'datetime' } }
		},
		{
			prop: 'is_platform_warehouse_auto_transfer',
			label: '是否平台仓自流转',
			component: { name: 'el-input' }
		},
		{ prop: 'vehicle_type', label: '车型', component: { name: 'el-input' } },
		{
			prop: 'scheduled_delivery_arrival_time',
			label: '预约送达时间',
			component: { name: 'el-date-picker', props: { type: 'datetime' } }
		},
		{
			prop: 'suggested_delivery_start_time',
			label: '建议发货时间（起）',
			component: { name: 'el-date-picker', props: { type: 'datetime' } }
		},
		{
			prop: 'suggested_delivery_end_time',
			label: '建议发货时间（止）',
			component: { name: 'el-date-picker', props: { type: 'datetime' } }
		}
	]
});

// 动态路由前缀
const getDynamicPrefix = () => {
	return import.meta.env.MODE === 'development' ? '/dev' : '/api';
};

// Import dialog state
const importDialogVisible = ref(false);
// Ensure the base URL is correct or use environment variables
const importAction = `${getDynamicPrefix()}/order/finance/orders/import`; // Construct URL properly if base URL is set

// --- Custom Request Functions using Axios ---

// Custom function for fetching paginated order list (used by cl-crud for page/search)
const fetchOrderList = async (params: any) => {
	const loading = ElLoading.service({
		lock: true,
		text: '正在加载数据...',
		background: 'rgba(0, 0, 0, 0.7)'
	});
	try {
		const response = await axios.post(`${getDynamicPrefix()}/order/finance/orders/page2`, {
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
			ElMessage.error(`获取订单列表失败: ${response.data?.message || '响应格式错误'}`);
			return { list: [], pagination: { total: 0, page: 1, size: 10 } };
		}
	} catch (error: unknown) {
		loading.close();
		let message = '未知错误';
		if (axios.isAxiosError(error)) {
			message = error.response?.data?.message || error.message || '网络错误或服务器无响应';
		}
		ElMessage.error(`获取订单列表失败: ${message}`);
		return { list: [], pagination: { total: 0, page: 1, size: 10 } };
	}
};

// --- Export Handling ---
const handleExport = async () => {
	const loading = ElLoading.service({
		lock: true,
		text: '正在导出数据...',
		background: 'rgba(0, 0, 0, 0.7)'
	});
	try {
		// Get current search/filter parameters from cl-crud instance
		const params = Crud.value?.search?.params || {};
		const response = await axios.get(`${getDynamicPrefix()}/order/finance/orders/export`, {
			params, // Send search parameters as query params for GET export
			responseType: 'blob' // Important for file download
		});

		// Create a link to download the file
		const url = window.URL.createObjectURL(new Blob([response.data]));
		const link = document.createElement('a');
		link.href = url;
		// Try to get filename from headers, fallback to default
		const contentDisposition = response.headers['content-disposition'];
		let filename = 'finance-orders.xlsx';
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
		window.URL.revokeObjectURL(url); // Clean up blob URL

		ElMessage.success('导出成功');
		loading.close();
	} catch (error: unknown) {
		loading.close();
		let message = '未知错误';
		if (axios.isAxiosError(error)) {
			// Try to read error message from blob response if backend sends JSON error for blob request
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

// --- Import Handling ---
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
	// Add size limit if needed
	// const isLt2M = file.size / 1024 / 1024 < 2;
	// if (!isLt2M) {
	//   ElMessage.error('上传文件大小不能超过 2MB!');
	//   return false;
	// }
	return true;
};

const onImportSuccess = (response: any, _uploadFile: any, _uploadFiles: any) => {
	// Check response structure based on your backend API
	if (response && response.code === 1000) {
		// Assuming code 1000 means success
		ElMessage.success(
			`导入成功${response.data?.count ? '，处理 ' + response.data.count + ' 条记录' : ''}`
		);
		importDialogVisible.value = false;
		Crud.value?.refresh(); // Refresh the table data
	} else {
		// Use message from response if available
		ElMessage.error(`导入失败: ${response?.message || '服务器返回错误'}`);
	}
};

const onImportError = (error: any) => {
	// Error handling for upload failure (network error, server error 5xx, etc.)
	let message = '上传失败';
	// Try to parse error message if it's JSON
	try {
		const errorResponse = JSON.parse(error.message || '{}');
		message = errorResponse.message || message;
	} catch (e) {
		// If parsing fails, use the raw message or a default
		message = error.message || '未知上传错误';
	}
	ElMessage.error(`导入失败: ${message}`);
};

// --- cl-crud Setup ---
const Crud = useCrud(
	{
		// Service configuration using direct axios calls
		service: {
			// Page uses the custom fetchOrderList which handles POST and loading/error states
			page: fetchOrderList,
			// Configure other operations to use axios directly as well
			list: (params: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/orders/list`, params)
					.then(res => res.data.data), // Adjust based on actual API response structure
			add: (data: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/orders/add`, data)
					.then(res => res.data), // Adjust response handling if needed
			delete: (data: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/orders/delete`, data)
					.then(res => res.data), // Adjust response handling if needed
			update: (data: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/orders/update`, data)
					.then(res => res.data), // Adjust response handling if needed
			info: (
				params: any // 'info' usually takes ID, often as path param or query param -> adjust if needed
			) =>
				// Assuming ID is in params.id - GET request common for info
				axios
					.get(
						`${getDynamicPrefix()}/order/finance/orders/info/${params.id}` /* or { params } if ID is query param */
					)
					.then(res => res.data.data) // Adjust based on actual API
		}
	},
	(app: any) => {
		// This callback runs once after crud setup, refresh data initially
		app.refresh();
	}
);
</script>

<style scoped>
/* Add any specific styles if needed */
</style>
