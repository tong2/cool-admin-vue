<template>
	<cl-crud ref="Crud">
		<cl-row>
			<cl-refresh-btn />
			<cl-multi-delete-btn />
			<el-button type="primary" @click="handleExport">导出</el-button>
			<el-button type="primary" @click="openGenerateDialog">生成日报明细</el-button>
			<cl-flex1 />
			<cl-search-key placeholder="搜索子订单编号" />
		</cl-row>

		<cl-row>
			<cl-table ref="Table" />
		</cl-row>

		<cl-row>
			<cl-flex1 />
			<cl-pagination />
		</cl-row>

		<!-- Generate Completion Table Dialog -->
		<el-dialog
			v-model="generateDialogVisible"
			title="生成日报明细"
			width="500px"
			:close-on-click-modal="false"
			:before-close="handleDialogClose"
		>
			<el-form :model="generateForm" ref="generateFormRef" :rules="generateRules">
				<el-form-item>
					<el-text>达人佣金和团长服务费，非必填</el-text>
				</el-form-item>
				<el-form-item>
					<el-text>已经生成过的数据时间，再次提交会重新更新该天日报明细数据！</el-text>
				</el-form-item>
				<el-form-item label="最新成本的数据日期">
					<el-text>{{ latestDataTime || '' }}</el-text>
				</el-form-item>
				<el-form-item label="数据日期" prop="gen_data_time">
					<el-date-picker
						v-model="generateForm.gen_data_time"
						type="date"
						placeholder="选择数据日期"
						value-format="YYYY-MM-DD"
					/>
				</el-form-item>
			</el-form>
			<template #footer>
				<el-button @click="handleDialogClose">取消</el-button>
				<el-button type="primary" @click="handleGenerateSubmit">提交</el-button>
			</template>
		</el-dialog>
	</cl-crud>
</template>

<script lang="ts" name="finance-accounting" setup>
import { useCrud, useTable } from '@cool-vue/crud';
import { ref, reactive, onMounted } from 'vue';
import { ElMessage, ElLoading, ElMessageBox } from 'element-plus';
import axios from 'axios';
import type { FormInstance, FormRules } from 'element-plus';

const handleExport = async () => {
	const loading = ElLoading.service({
		lock: true,
		text: '正在导出数据...',
		background: 'rgba(0, 0, 0, 0.7)'
	});
	try {
		const params = Crud.value?.search?.params || {};
		const response = await axios.get(`${getDynamicPrefix()}/order/finance/accounting/export`, {
			params,
			responseType: 'blob'
		});

		const url = window.URL.createObjectURL(new Blob([response.data]));
		const link = document.createElement('a');
		link.href = url;
		const contentDisposition = response.headers['content-disposition'];
		let filename = '日报明细表.xlsx';
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

// Fetch latest data time from cost API
const latestDataTime = ref<string | null>(null);
const fetchLatestDataTime = async () => {
	try {
		const response = await axios.get(
			`${getDynamicPrefix()}/order/finance/cost/latest-data-time`
		);
		if (
			response.data &&
			response.data.code === 1000 &&
			response.data.data &&
			response.data.data.code === 1000 &&
			response.data.data.data
		) {
			latestDataTime.value = response.data.data.data; // Sets to "2025-05-19"
		} else {
			latestDataTime.value = null;
		}
	} catch (error: unknown) {
		let message = '未知错误';
		if (axios.isAxiosError(error)) {
			message =
				error.response?.data?.data?.message ||
				error.response?.data?.message ||
				error.message ||
				'网络错误或服务器无响应';
		}
		ElMessage.error(`获取最新数据日期失败: ${message}`);
		latestDataTime.value = null;
	}
};

// Dialog and form handling
const generateDialogVisible = ref(false);
const generateFormRef = ref<FormInstance | null>(null);
const generateForm = reactive({
	gen_data_time: ''
});
const generateRules = reactive<FormRules>({
	gen_data_time: [{ required: true, message: '请选择数据日期', trigger: 'change' }]
});

const openGenerateDialog = async () => {
	await fetchLatestDataTime();
	generateDialogVisible.value = true;
};

const handleDialogClose = () => {
	generateForm.gen_data_time = '';
	generateDialogVisible.value = false;
	generateFormRef.value?.resetFields();
};

const handleGenerateSubmit = async () => {
	if (!generateFormRef.value) return;

	await generateFormRef.value.validate(async valid => {
		if (valid) {
			const genDataTimeRaw = (generateForm.gen_data_time || '').trim();
			if (!genDataTimeRaw) {
				ElMessage.error('数据日期不能为空');
				return;
			}

			// 格式化为 yyyy-MM-dd 00:00:00
			const formatDateToFullString = (dateStr: string) => {
				const dateObj = new Date(dateStr);
				if (isNaN(dateObj.getTime())) return ''; // 非法日期返回空字符串
				const pad = (n: number) => (n < 10 ? '0' + n : n);
				return `${dateObj.getFullYear()}-${pad(dateObj.getMonth() + 1)}-${pad(dateObj.getDate())} 00:00:00`;
			};

			const genDataTime = formatDateToFullString(genDataTimeRaw);
			if (!genDataTime) {
				ElMessage.error('日期格式错误');
				return;
			}

			if (latestDataTime.value && genDataTime > latestDataTime.value) {
				try {
					await ElMessageBox.confirm(
						`您选择的数据日期 ${genDataTime} 晚于最新数据日期 ${latestDataTime.value}，是否继续？`,
						'警告',
						{
							confirmButtonText: '继续',
							cancelButtonText: '取消',
							type: 'warning'
						}
					);
				} catch {
					return;
				}
			}

			const loading = ElLoading.service({
				lock: true,
				text: '正在生成完成表...',
				background: 'rgba(0, 0, 0, 0.7)'
			});

			try {
				const response = await axios.post(
					`${getDynamicPrefix()}/order/finance/finish/generate`,
					{
						gen_data_time: genDataTime
					}
				);
				loading.close();
				if (response.data?.code === 1000) {
					ElMessage.success(response.data?.data);
					handleDialogClose();
					Crud.value?.refresh();
				} else {
					ElMessage.error(`生成失败: ${response.data?.message || '响应格式错误'}`);
				}
			} catch (error: unknown) {
				loading.close();
				const message = axios.isAxiosError(error)
					? error.response?.data?.message || error.message || '网络错误'
					: '未知错误';
				ElMessage.error(`生成失败: ${message}`);
			}
		}
	});
};

const Table = useTable({
	columns: [
		{ type: 'selection', width: 60 },
		{ label: 'ID', prop: 'id', minWidth: 80, hidden: true },
		{ label: '子订单编号', prop: 'sub_order_no', minWidth: 120, fixed: 'left' },
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
		{ label: '商家编码', prop: 'merchant_code', minWidth: 120, fixed: 'left' },
		{ label: '状态', prop: 'status', minWidth: 100 },
		{ label: '仓库', prop: 'warehouse', minWidth: 120 },
		{ label: '交易日期', prop: 'trade_date', minWidth: 150 },
		{ label: '商品ID', prop: 'product_id', minWidth: 120 },
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

const Crud = ref();
useCrud(
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
		Crud.value = app;
		app.refresh();
	}
);

// Trigger the list query when the component is mounted
onMounted(() => {
	Crud.value?.refresh();
	fetchLatestDataTime();
});
</script>

<style scoped></style>
