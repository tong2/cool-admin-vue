<template>
	<cl-crud ref="Crud">
		<cl-row>
			<cl-refresh-btn />
			<cl-multi-delete-btn />
			<el-button type="primary" @click="handleExport">导出</el-button>
			<el-button type="success" @click="handleImport">导入</el-button>
			<cl-flex1 />
			<cl-search-key placeholder="搜索商品编码" />
		</cl-row>

		<cl-row>
			<cl-table ref="Table" />
		</cl-row>

		<cl-row>
			<cl-flex1 />
			<cl-pagination />
		</cl-row>

		<cl-upsert ref="Upsert" />

		<el-dialog title="导入库存记录" v-model="importDialogVisible" width="30%">
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

<script lang="ts" name="stock-list" setup>
import { useCrud, useTable, useUpsert } from '@cool-vue/crud';
import { ref } from 'vue';
import { ElMessage, ElLoading } from 'element-plus';
import axios from 'axios';

const Table = useTable({
	columns: [
		{ type: 'selection', width: 60 },
		{ label: 'ID', prop: 'id', minWidth: 80 },
		{ label: '商品编码', prop: 'product_code', minWidth: 150 },
		{ label: '重量', prop: 'weight', minWidth: 120 },
		{ label: '采购价', prop: 'purchase_price', minWidth: 120 },
		{ label: '打款价', prop: 'payment_price', minWidth: 120 },
		{ label: '箱规', prop: 'box_specification', minWidth: 150 },
		{ label: '商品数量', prop: 'product_quantity', minWidth: 120 },
		{ label: '品牌方', prop: 'brand_owner', minWidth: 150 },
		{ label: '供货方', prop: 'supplier', minWidth: 150 },
		{ label: '是否计算返利', prop: 'is_rebate_calculated', minWidth: 120 },
		{ label: '费用1', prop: 'fee1', minWidth: 100 },
		{ label: '费用2', prop: 'fee2', minWidth: 100 },
		{ label: '费用3', prop: 'fee3', minWidth: 100 },
		{ label: '费用4', prop: 'fee4', minWidth: 100 },
		{ label: '费用5', prop: 'fee5', minWidth: 100 },
		{ label: '返利', prop: 'rebate', minWidth: 100 },
		{ label: '操作', type: 'op', buttons: ['delete'], minWidth: 120 }
	]
});

const Upsert = useUpsert({
	dialog: { width: '600px' },
	items: [
		{
			prop: 'product_code',
			label: '商品编码',
			component: { name: 'el-input' },
			required: true
		},
		{
			prop: 'weight',
			label: '重量',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{
			prop: 'purchase_price',
			label: '采购价',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{
			prop: 'payment_price',
			label: '打款价',
			component: { name: 'el-input-number', props: { min: 0, precision: 2 } }
		},
		{ prop: 'box_specification', label: '箱规', component: { name: 'el-input' } },
		{
			prop: 'product_quantity',
			label: '商品数量',
			component: { name: 'el-input-number', props: { min: 0 } }
		},
		{ prop: 'brand_owner', label: '品牌方', component: { name: 'el-input' } },
		{ prop: 'supplier', label: '供货方', component: { name: 'el-input' } },
		{
			prop: 'is_rebate_calculated',
			label: '是否计算返利',
			component: { name: 'el-input' }
		},
		{ prop: 'fee1', label: '费用1', component: { name: 'el-input' } },
		{ prop: 'fee2', label: '费用2', component: { name: 'el-input' } },
		{ prop: 'fee3', label: '费用3', component: { name: 'el-input' } },
		{ prop: 'fee4', label: '费用4', component: { name: 'el-input' } },
		{ prop: 'fee5', label: '费用5', component: { name: 'el-input' } },
		{ prop: 'rebate', label: '返利', component: { name: 'el-input' } }
	]
});

const getDynamicPrefix = () => {
	return import.meta.env.MODE === 'development' ? '/dev' : '/api';
};

const importDialogVisible = ref(false);
const importAction = `${getDynamicPrefix()}/order/finance/stock/import`;

const fetchStockList = async (params: any) => {
	const loading = ElLoading.service({
		lock: true,
		text: '正在加载数据...',
		background: 'rgba(0, 0, 0, 0.7)'
	});
	try {
		const response = await axios.post(`${getDynamicPrefix()}/order/finance/stock/page2`, {
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
			ElMessage.error(`获取库存列表失败: ${response.data?.message || '响应格式错误'}`);
			return { list: [], pagination: { total: 0, page: 1, size: 10 } };
		}
	} catch (error: unknown) {
		loading.close();
		let message = '未知错误';
		if (axios.isAxiosError(error)) {
			message = error.response?.data?.message || error.message || '网络错误或服务器无响应';
		}
		ElMessage.error(`获取库存列表失败: ${message}`);
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
		const response = await axios.get(`${getDynamicPrefix()}/order/finance/stock/export`, {
			params,
			responseType: 'blob'
		});

		const url = window.URL.createObjectURL(new Blob([response.data]));
		const link = document.createElement('a');
		link.href = url;
		const contentDisposition = response.headers['content-disposition'];
		let filename = 'finance-stock.xlsx';
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
			page: fetchStockList,
			list: (params: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/stock/list`, params)
					.then(res => res.data.data),
			add: (data: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/stock/add`, data)
					.then(res => res.data),
			delete: (data: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/stock/delete`, data)
					.then(res => res.data),
			update: (data: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/stock/update`, data)
					.then(res => res.data),
			info: (params: any) =>
				axios
					.get(`${getDynamicPrefix()}/order/finance/stock/info/${params.id}`)
					.then(res => res.data.data)
		}
	},
	(app: any) => {
		app.refresh();
	}
);
</script>

<style scoped></style>
