<template>
	<cl-crud ref="Crud">
		<cl-row>
			<cl-refresh-btn />
			<cl-multi-delete-btn />
			<el-button type="primary" @click="handleExport">导出</el-button>
			<el-button type="warning" @click="handleExportTemplate">下载模板</el-button>
			<el-button type="success" @click="handleImport">导入</el-button>
			<cl-flex1 />
			<cl-search-key placeholder="搜索货号" />
		</cl-row>

		<cl-row>
			<cl-table ref="Table" />
		</cl-row>

		<cl-row>
			<cl-flex1 />
			<cl-pagination />
		</cl-row>

		<el-dialog title="导入箱规记录" v-model="importDialogVisible" width="30%">
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

<script lang="ts" name="finance-box-spec" setup>
import { useCrud, useTable } from '@cool-vue/crud';
import { ref } from 'vue';
import { ElMessage, ElLoading } from 'element-plus';
import axios from 'axios';

const Table = useTable({
	columns: [
		{ type: 'selection', width: 60 },
		{ label: 'ID', prop: 'id', minWidth: 80, hidden: true },
		{ label: '货号', prop: 'product_number', minWidth: 120, fixed: 'left' },
		{
			label: '数据时间',
			prop: 'gen_data_time',
			minWidth: 120,
			fixed: 'left',
			formatter: row =>
				row.gen_data_time && typeof row.gen_data_time === 'string'
					? row.gen_data_time.slice(0, 10)
					: ''
		},
		{ label: '仓库', prop: 'warehouse', minWidth: 120 },
		{ label: '品牌', prop: 'brand', minWidth: 120 },
		{ label: '箱规', prop: 'box_specification', minWidth: 150 },
		{ label: '规格包/提/条/瓶', prop: 'unit_specification', minWidth: 150 },
		{ label: '包/提/条/瓶', prop: 'unit_type', minWidth: 120 },
		{ label: '操作', type: 'op', buttons: ['delete'], minWidth: 120 }
	]
});

const getDynamicPrefix = () => {
	return import.meta.env.MODE === 'development' ? '/dev' : '/api';
};

const importDialogVisible = ref(false);
const importAction = `${getDynamicPrefix()}/order/finance/box_spec/import`;

const fetchBoxSpecList = async (params: any) => {
	const loading = ElLoading.service({
		lock: true,
		text: '正在加载数据...',
		background: 'rgba(0, 0, 0, 0.7)'
	});
	try {
		const response = await axios.post(`${getDynamicPrefix()}/order/finance/box_spec/page2`, {
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
			ElMessage.error(`获取箱规列表失败: ${response.data?.message || '响应格式错误'}`);
			return { list: [], pagination: { total: 0, page: 1, size: 10 } };
		}
	} catch (error: unknown) {
		loading.close();
		let message = '未知错误';
		if (axios.isAxiosError(error)) {
			message = error.response?.data?.message || error.message || '网络错误或服务器无响应';
		}
		ElMessage.error(`获取箱规列表失败: ${message}`);
		return { list: [], pagination: { total: 0, page: 1, size: 10 } };
	}
};

const performExport = async (isTemplate: boolean) => {
	const loading = ElLoading.service({
		lock: true,
		text: isTemplate ? '正在生成模板...' : '正在导出数据...',
		background: 'rgba(0, 0, 0, 0.7)'
	});
	try {
		const params = isTemplate
			? { keyWord: 'downloadtemplate' }
			: Crud.value?.search?.params || {};
		const response = await axios.get(`${getDynamicPrefix()}/order/finance/box_spec/export`, {
			params,
			responseType: 'blob'
		});

		const url = window.URL.createObjectURL(new Blob([response.data]));
		const link = document.createElement('a');
		link.href = url;
		const contentDisposition = response.headers['content-disposition'];
		let filename = isTemplate ? '箱规表模板.xlsx' : '箱规表.xlsx';
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

		ElMessage.success(isTemplate ? '模板下载成功' : '导出成功');
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
					message = errorJson.message || (isTemplate ? '模板下载失败' : '导出失败');
				} catch (parseError) {
					message = isTemplate ? '未知模板下载错误' : '未知导出错误';
				}
			} else {
				message =
					error.response?.data?.message || error.message || '网络错误或服务器无响应';
			}
		}
		ElMessage.error(`${isTemplate ? '模板下载失败' : '导出失败'}: ${message}`);
	}
};

const handleExport = (_event: any) => {
	performExport(false);
};

const handleExportTemplate = (_event: any) => {
	performExport(true);
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
			page: fetchBoxSpecList,
			list: (params: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/box_spec/list`, params)
					.then(res => res.data.data),
			add: (data: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/box_spec/add`, data)
					.then(res => res.data),
			delete: (data: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/box_spec/delete`, data)
					.then(res => res.data),
			update: (data: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/box_spec/update`, data)
					.then(res => res.data),
			info: (params: any) =>
				axios
					.get(`${getDynamicPrefix()}/order/finance/box_spec/info/${params.id}`)
					.then(res => res.data.data)
		}
	},
	(app: any) => {
		app.refresh();
	}
);
</script>

<style scoped></style>
