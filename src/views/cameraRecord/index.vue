<template>
	<div class="system-role-container layout-padding">
		<div class="system-role-padding layout-padding-auto layout-padding-view">
			<div class="system-user-search mb15">
				<el-input v-model="state.tableData.param.search1" size="default" placeholder="请输入识别时间" style="max-width: 180px"> </el-input>
				<el-input v-model="state.tableData.param.search3" size="default" placeholder="请输入最低阈值" style="max-width: 180px; margin-left: 15px"></el-input>
				<el-button size="default" type="primary" class="predict-button" @click="getTableData()">
					<el-icon>
						<ele-Search />
					</el-icon>
					查询
				</el-button>
			</div>
			<el-table :data="state.tableData.data" v-loading="state.tableData.loading" style="width: 100%">
				<el-table-column prop="num" label="序号" width="100" align="center" />
				<el-table-column prop="outVideo" label="处理结果" width="200" align="center">
					<template #default="scope">
						<video class="video" preload="auto" controls :key="scope.row.outVideo + uniqueKey">
							<source :src="scope.row.outVideo" type="video/mp4" />
						</video>
					</template>
				</el-table-column>
				<el-table-column prop="weight" label="识别权重" align="center" />
				<el-table-column prop="conf" label="最小阈值" show-overflow-tooltip width="100" align="center"></el-table-column>
				<el-table-column prop="username" label="识别用户" show-overflow-tooltip align="center"></el-table-column>
				<el-table-column prop="startTime" label="识别时间" show-overflow-tooltip align="center"></el-table-column>
				<el-table-column label="操作" width="240" align="center">
					<template #default="scope">
						<el-button size="small" text type="primary" @click="onRowDel(scope.row)">删除</el-button>
					</template>
				</el-table-column>
			</el-table>
			<el-pagination @size-change="onHandleSizeChange" @current-change="onHandleCurrentChange" class="mt15"
				:pager-count="5" :page-sizes="[10, 20, 30]" v-model:current-page="state.tableData.param.pageNum"
				background v-model:page-size="state.tableData.param.pageSize"
				layout="total, sizes, prev, pager, next, jumper" :total="state.tableData.total">
			</el-pagination>
		</div>
	</div>
</template>

<script setup lang="ts" name="systemRole">
import { reactive, onMounted, ref } from 'vue';
import { ElMessageBox, ElMessage } from 'element-plus';
import request from '/@/utils/request';
import { useUserInfo } from '/@/stores/userInfo';
import { storeToRefs } from 'pinia';

const stores = useUserInfo();
const { userInfos } = storeToRefs(stores);

const state = reactive<SysRoleState>({
	tableData: {
		data: [] as any,
		total: 0,
		loading: false,
		param: {
			search: '',
			search3: '',
			search2: '',
			pageNum: 1,
			pageSize: 10,
		},
	},
});

// 唯一标识符，动态刷新
const uniqueKey = ref(0);

const getTableData = () => {
	state.tableData.loading = true;
	if (userInfos.value.userName != 'admin') {
		state.tableData.param.search = userInfos.value.userName;
	}
	request
		.get('/api/cameraRecords', {
			params: state.tableData.param,
		})
		.then((res) => {
			if (res.code == 0) {
				state.tableData.data = [];
				setTimeout(() => {
					state.tableData.loading = false;
				}, 500);
				for (let i = 0; i < res.data.records.length; i++) {
					state.tableData.data[i] = res.data.records[i];
					state.tableData.data[i]['num'] = i + 1;
				}
				state.tableData.total = res.data.total;

				// 更新唯一标识符
				uniqueKey.value++;
			} else {
				ElMessage({
					type: 'error',
					message: res.msg,
				});
			}
		});
};

const onRowDel = (row: any) => {
	ElMessageBox.confirm(`此操作将永久删除该信息，是否继续?`, '提示', {
		confirmButtonText: '确认',
		cancelButtonText: '取消',
		type: 'warning',
	})
		.then(() => {
			request.delete('/api/cameraRecords/' + row.id).then((res) => {
				if (res.code == 0) {
					ElMessage({
						type: 'success',
						message: '删除成功！',
					});
				} else {
					ElMessage({
						type: 'error',
						message: res.msg,
					});
				}
			});
			setTimeout(() => {
				getTableData();
			}, 500);
		})
		.catch(() => { });
};

const onHandleSizeChange = (val: number) => {
	state.tableData.param.pageSize = val;
	getTableData();
};

const onHandleCurrentChange = (val: number) => {
	state.tableData.param.pageNum = val;
	getTableData();
};

onMounted(() => {
	getTableData();
});
</script>


<style scoped lang="scss">
.system-role-container {

	// background: radial-gradient(circle, #d3e3f1 0%, #ffffff 100%);
	.system-role-padding {
		padding: 15px;

		.el-table {
			flex: 1;
		}
	}
}

.video {
	width: 100%;
	max-height: 100%;
	/* 限制视频最大高度不超过父元素高度 */
	height: auto;
	object-fit: contain;
}
.predict-button {
	margin-left: 10px;
	background: #4CAF50;
    border-color: #4CAF50;
    &:hover {
        background: #2E7D32;
        border-color: #2E7D32;
    }
}
</style>
