<template>
	<cl-crud ref="Crud">
		<cl-row>
			<cl-refresh-btn />
			<cl-multi-delete-btn />
			<el-button type="primary" @click="handleExport">导出</el-button>
			<el-button type="success" @click="handleImport">导入</el-button>
			<cl-flex1 />
			<cl-search-key placeholder="搜索省份" />
		</cl-row>

		<cl-row>
			<cl-table ref="Table" />
		</cl-row>

		<cl-row>
			<cl-flex1 />
			<cl-pagination />
		</cl-row>

		<el-dialog title="导入邮资记录" v-model="importDialogVisible" width="30%">
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

<script lang="ts" name="finance-postage" setup>
import { useCrud, useTable } from '@cool-vue/crud';
import { ref } from 'vue';
import { ElMessage, ElLoading } from 'element-plus';
import axios from 'axios';

const Table = useTable({
	columns: [
		{ type: 'selection', width: 60 },
		{ label: 'ID', prop: 'id', minWidth: 80, hidden: true },
		{ label: '省份', prop: 'province', minWidth: 120, fixed: 'left' },
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
		{ label: '0-0.5KG (含打包辅材)', prop: 'weight_0_0_5kg_with_pack', minWidth: 150 },
		{
			label: '0.51-1KG (机打含打包辅材)',
			prop: 'weight_0_51_1kg_machine_with_pack',
			minWidth: 180
		},
		{ label: '1.01-2KG (贴单件)', prop: 'weight_1_01_2kg_label', minWidth: 150 },
		{ label: '1.01-2KG (打包品)', prop: 'weight_1_01_2kg_pack', minWidth: 150 },
		{ label: '2.01-3KG (贴单件)', prop: 'weight_2_01_3kg_label', minWidth: 150 },
		{ label: '2.01-3KG (打包品)', prop: 'weight_2_01_3kg_pack', minWidth: 150 },
		{
			label: '10公斤内首重3公斤 (含辅材和操作费)',
			prop: 'ten_kg_with_fee_first_3kg',
			minWidth: 200
		},
		{
			label: '10公斤内续重每公斤 (3kg以上不含辅材和操作费)',
			prop: 'ten_kg_no_fee_additional_per_kg',
			minWidth: 250
		},
		{ label: '操作', type: 'op', buttons: ['delete'], minWidth: 120 }
	]
});

const getDynamicPrefix = () => {
	return import.meta.env.MODE === 'development' ? '/dev' : '/api';
};

const importDialogVisible = ref(false);
const importAction = `${getDynamicPrefix()}/order/finance/postage/import`;

const fetchPostageList = async (params: any) => {
	const loading = ElLoading.service({
		lock: true,
		text: '正在加载数据...',
		background: 'rgba(0, 0, 0, 0.7)'
	});
	try {
		const response = await axios.post(`${getDynamicPrefix()}/order/finance/postage/page2`, {
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
			ElMessage.error(`获取邮资列表失败: ${response.data?.message || '响应格式错误'}`);
			return { list: [], pagination: { total: 0, page: 1, size: 10 } };
		}
	} catch (error: unknown) {
		loading.close();
		let message = '未知错误';
		if (axios.isAxiosError(error)) {
			message = error.response?.data?.message || error.message || '网络错误或服务器无响应';
		}
		ElMessage.error(`获取邮资列表失败: ${message}`);
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
		const response = await axios.get(`${getDynamicPrefix()}/order/finance/postage/export`, {
			params,
			responseType: 'blob'
		});

		const url = window.URL.createObjectURL(new Blob([response.data]));
		const link = document.createElement('a');
		link.href = url;
		const contentDisposition = response.headers['content-disposition'];
		let filename = '邮资表.xlsx';
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
			page: fetchPostageList,
			list: (params: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/postage/list`, params)
					.then(res => res.data.data),
			add: (data: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/postage/add`, data)
					.then(res => res.data),
			delete: (data: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/postage/delete`, data)
					.then(res => res.data),
			update: (data: any) =>
				axios
					.post(`${getDynamicPrefix()}/order/finance/postage/update`, data)
					.then(res => res.data),
			info: (params: any) =>
				axios
					.get(`${getDynamicPrefix()}/order/finance/postage/info/${params.id}`)
					.then(res => res.data.data)
		}
	},
	(app: any) => {
		app.refresh();
	}
);
</script>

<style scoped></style>
