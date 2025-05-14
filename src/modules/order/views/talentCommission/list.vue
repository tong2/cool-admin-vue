<template>
	<cl-crud ref="Crud">
		<cl-row>
			<cl-refresh-btn />
			<cl-multi-delete-btn />
			<el-button type="primary" @click="handleExport">导出</el-button>
			<el-button type="success" @click="handleImport">导入</el-button>
			<cl-flex1 />
			<cl-search-key placeholder="搜索订单ID" />
		</cl-row>

		<cl-row>
			<cl-table ref="Table" />
		</cl-row>

		<cl-row>
			<cl-flex1 />
			<cl-pagination />
		</cl-row>

		<cl-upsert ref="Upsert" />

		<el-dialog title="导入达人佣金记录" v-model="importDialogVisible" width="30%">
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

<script lang="ts" name="talent-commission-list" setup>
import { useCrud, useTable, useUpsert } from '@cool-vue/crud';
import { ref } from 'vue';
import { ElMessage, ElLoading } from 'element-plus';
import axios from 'axios';

const Table = useTable({
	columns: [
		{ type: 'selection', width: 60 },
		{ label: 'ID', prop: 'id', minWidth: 80 },
		{ label: '订单ID', prop: 'order_id', minWidth: 150 },
		{ label: '商品ID', prop: 'product_id', minWidth: 120 },
		{ label: '商品名称', prop: 'product_name', minWidth: 150 },
		{ label: '作者账号', prop: 'author_account', minWidth: 120 },
		{ label: '抖音/火山号', prop: 'douyin_huoshan_id', minWidth: 120 },
		{ label: '支付金额', prop: 'payment_amount', minWidth: 120 },
		{ label: '佣金率', prop: 'commission_rate', minWidth: 100 },
		{ label: '预估佣金支出', prop: 'estimated_commission', minWidth: 150 },
		{ label: '结算金额', prop: 'settled_amount', minWidth: 120 },
		{ label: '实际佣金支出', prop: 'actual_commission', minWidth: 150 },
		{ label: '订单状态', prop: 'order_status', minWidth: 120 },
		{ label: '付款时间', prop: 'payment_time', minWidth: 170 },
		{ label: '收货时间', prop: 'receipt_time', minWidth: 170 },
		{ label: '订单结算时间', prop: 'order_settlement_time', minWidth: 170 },
		{ label: '尾款支付时间', prop: 'final_payment_time', minWidth: 170 },
		{ label: '定金金额', prop: 'deposit_amount', minWidth: 120 },
		{ label: '店铺ID', prop: 'shop_id', minWidth: 120 },
		{ label: '店铺名称', prop: 'shop_name', minWidth: 150 },
		{ label: '商品数量', prop: 'product_quantity', minWidth: 100 },
		{ label: '佣金发票', prop: 'commission_invoice', minWidth: 120 },
		{ label: '冻结比例', prop: 'freeze_ratio', minWidth: 100 },
		{ label: '是否阶梯佣金', prop: 'is_tiered_commission', minWidth: 120 },
		{ label: '门槛销量', prop: 'threshold_sales', minWidth: 100 },
		{ label: '基础佣金率', prop: 'base_commission_rate', minWidth: 120 },
		{ label: '升佣佣金率', prop: 'increased_commission_rate', minWidth: 120 },
		{ label: '预估奖励佣金支出', prop: 'estimated_bonus_commission', minWidth: 150 },
		{ label: '结算奖励佣金支出', prop: 'settled_bonus_commission', minWidth: 150 },
		{ label: '阶梯计划ID', prop: 'tiered_plan_id', minWidth: 120 },
		{ label: '支付补贴', prop: 'payment_subsidy', minWidth: 120 },
		{ label: '平台补贴', prop: 'platform_subsidy', minWidth: 120 },
		{ label: '达人补贴', prop: 'talent_subsidy', minWidth: 120 },
		{ label: '运费', prop: 'shipping_fee', minWidth: 100 },
		{ label: '税费', prop: 'tax_fee', minWidth: 100 },
		{ label: '运费补贴', prop: 'shipping_subsidy', minWidth: 120 },
		{ label: '分销来源', prop: 'distribution_source', minWidth: 120 },
		{ label: '营销活动ID', prop: 'marketing_activity_id', minWidth: 120 },
		{ label: '推广费率', prop: 'promotion_rate', minWidth: 100 },
		{ label: '推广技术服务费', prop: 'promotion_tech_service_fee', minWidth: 150 },
		{ label: '预估推广费支出', prop: 'estimated_promotion_fee', minWidth: 150 },
		{ label: '结算推广费支出', prop: 'settled_promotion_fee', minWidth: 150 },
		{ label: '计划类型', prop: 'plan_type', minWidth: 120 },
		{ label: '订单来源', prop: 'order_source', minWidth: 120 },
		{ label: '流量细分来源', prop: 'traffic_segment_source', minWidth: 120 },
		{ label: '流量来源', prop: 'traffic_source', minWidth: 120 },
		{ label: '订单类型', prop: 'order_type', minWidth: 120 },
		{ label: '生效时间', prop: 'effect_time', minWidth: 170 },
		{ label: '操作', type: 'op', buttons: ['delete'], minWidth: 120 }
	]
});

const Upsert = useUpsert({
	dialog: { width: '800px' },
	items: [
		{ prop: 'order_id', label: '订单ID', component: { name: 'el-input' }, required: true },
		{ prop: 'product_id', label: '商品ID', component: { name: 'el-input' } },
		{ prop: 'product_name', label: '商品名称', component: { name: 'el-input' } },
		{ prop: 'author_account', label: '作者账号', component: { name: 'el-input' } },
		{ prop: 'douyin_huoshan_id', label: '抖音/火山号', component: { name: 'el-input' } },
		{
			prop: 'payment_amount',
			label: '支付金额',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{
			prop: 'commission_rate',
			label: '佣金率',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{
			prop: 'estimated_commission',
			label: '预估佣金支出',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{
			prop: 'settled_amount',
			label: '结算金额',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{
			prop: 'actual_commission',
			label: '实际佣金支出',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{ prop: 'order_status', label: '订单状态', component: { name: 'el-input' } },
		{
			prop: 'unsettled_reason',
			label: '超时未结算原因',
			component: { name: 'el-input', props: { type: 'textarea' } }
		},
		{
			prop: 'payment_time',
			label: '付款时间',
			component: { name: 'el-date-picker', props: { type: 'datetime' } }
		},
		{
			prop: 'receipt_time',
			label: '收货时间',
			component: { name: 'el-date-picker', props: { type: 'datetime' } }
		},
		{
			prop: 'order_settlement_time',
			label: '订单结算时间',
			component: { name: 'el-date-picker', props: { type: 'datetime' } }
		},
		{
			prop: 'product_source',
			label: '商品来源',
			component: { name: 'el-input', props: { type: 'textarea' } }
		},
		{
			prop: 'final_payment_time',
			label: '尾款支付时间',
			component: { name: 'el-date-picker', props: { type: 'datetime' } }
		},
		{
			prop: 'deposit_amount',
			label: '定金金额',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{ prop: 'shop_id', label: '店铺ID', component: { name: 'el-input' } },
		{ prop: 'shop_name', label: '店铺名称', component: { name: 'el-input' } },
		{
			prop: 'product_quantity',
			label: '商品数量',
			component: { name: 'el-input-number', props: { min: 0 } }
		},
		{ prop: 'commission_invoice', label: '佣金发票', component: { name: 'el-input' } },
		{
			prop: 'freeze_ratio',
			label: '冻结比例',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{ prop: 'is_tiered_commission', label: '是否阶梯佣金', component: { name: 'el-input' } },
		{
			prop: 'threshold_sales',
			label: '门槛销量',
			component: { name: 'el-input-number', props: { min: 0 } }
		},
		{
			prop: 'base_commission_rate',
			label: '基础佣金率',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{
			prop: 'increased_commission_rate',
			label: '升佣佣金率',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{
			prop: 'estimated_bonus_commission',
			label: '预估奖励佣金支出',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{
			prop: 'settled_bonus_commission',
			label: '结算奖励佣金支出',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{ prop: 'tiered_plan_id', label: '阶梯计划ID', component: { name: 'el-input' } },
		{
			prop: 'payment_subsidy',
			label: '支付补贴',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{
			prop: 'platform_subsidy',
			label: '平台补贴',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{
			prop: 'talent_subsidy',
			label: '达人补贴',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{
			prop: 'shipping_fee',
			label: '运费',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{
			prop: 'tax_fee',
			label: '税费',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{
			prop: 'shipping_subsidy',
			label: '运费补贴',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{ prop: 'distribution_source', label: '分销来源', component: { name: 'el-input' } },
		{ prop: 'marketing_activity_id', label: '营销活动ID', component: { name: 'el-input' } },
		{
			prop: 'promotion_rate',
			label: '推广费率',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{
			prop: 'promotion_tech_service_fee',
			label: '推广技术服务费',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{
			prop: 'estimated_promotion_fee',
			label: '预估推广费支出',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{
			prop: 'settled_promotion_fee',
			label: '结算推广费支出',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{ prop: 'plan_type', label: '计划类型', component: { name: 'el-input' } },
		{ prop: 'order_source', label: '订单来源', component: { name: 'el-input' } },
		{ prop: 'traffic_segment_source', label: '流量细分来源', component: { name: 'el-input' } },
		{ prop: 'traffic_source', label: '流量来源', component: { name: 'el-input' } },
		{ prop: 'order_type', label: '订单类型', component: { name: 'el-input' } }
	]
});

const getDynamicPrefix = () => {
	return import.meta.env.MODE === 'development' ? '/dev' : '/api';
};

const importDialogVisible = ref(false);
const importAction = `${getDynamicPrefix()}/order/finance/talentCommission/import`;

const fetchTalentCommissionList = async (params: any) => {
	const loading = ElLoading.service({
		lock: true,
		text: '正在加载数据...',
		background: 'rgba(0, 0, 0, 0.7)'
	});
	try {
		const response = await axios.post(
			`${getDynamicPrefix()}/order/finance/talentCommission/page2`,
			{
				...params
			}
		);
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
			ElMessage.error(`获取达人佣金列表失败: ${response.data?.message || '响应格式错误'}`);
			return { list: [], pagination: { total: 0, page: 1, size: 10 } };
		}
	} catch (error: unknown) {
		loading.close();
		let message = '未知错误';
		if (axios.isAxiosError(error)) {
			message = error.response?.data?.message || error.message || '网络错误或服务器无响应';
		}
		ElMessage.error(`获取达人佣金列表失败: ${message}`);
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
		const response = await axios.get(
			`${getDynamicPrefix()}/order/finance/talentCommission/export`,
			{
				params,
				responseType: 'blob'
			}
		);

		const url = window.URL.createObjectURL(new Blob([response.data]));
		const link = document.createElement('a');
		link.href = url;
		const contentDisposition = response.headers['content-disposition'];
		let filename = 'finance-talent-commission.xlsx';
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
			page: fetchTalentCommissionList,
			list: (params: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/talentCommission/list`, params)
					.then(res => res.data.data),
			add: (data: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/talentCommission/add`, data)
					.then(res => res.data),
			delete: (data: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/talentCommission/delete`, data)
					.then(res => res.data),
			update: (data: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/talentCommission/update`, data)
					.then(res => res.data),
			info: (params: any) =>
				axios
					.get(`${getDynamicPrefix()}/order/finance/talentCommission/info/${params.id}`)
					.then(res => res.data.data)
		}
	},
	(app: any) => {
		app.refresh();
	}
);
</script>

<style scoped></style>
