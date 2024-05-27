<template>
  <div>
    <fe-panel>
      <section class="toolbar-panel">
        <div class="toolbar-item">
          <span class="item-label">创建时间</span>
          <el-date-picker
            v-model="dateRes.vModel"
            type="datetimerange"
            clearable
            value-format="YYYY-MM-DD HH:mm:ss"
            format="YYYY-MM-DD HH:mm:ss"
            start-placeholder="开始时间"
            end-placeholder="结束时间"
            :default-time="dateRes.defaultTime"
            @change="dateRes.handleValueChange"
          />
        </div>
        <div class="toolbar-item">
          <span class="item-label">编号</span>
          <el-input
            v-model.trim="tableRes.tableOptions.orderNo"
            :maxlength="20"
            clearable
            placeholder="请输入"
            @keyup.enter="handleSearch"
          ></el-input>
        </div>
        <div class="toolbar-item">
          <span class="item-label">状态</span>
          <el-select v-model="tableRes.tableOptions.orderStatus" placeholder="请选择" clearable filterable>
            <el-option v-for="item in orderStatusArr" :key="item.value" :value="item.value" :label="item.label"></el-option>
          </el-select>
        </div>
        <div class="toolbar-item">
          <el-button type="primary" @click="handleSearch">查询</el-button>
          <el-button @click="handleReset">重置</el-button>
        </div>
      </section>
    </fe-panel>

    <fe-panel :title="(route.meta.title as string)">
      <template #action>
        <el-button type="primary" @click="handleAdd">新增</el-button>
      </template>
      <fe-table
        border
        ref="tableRef"
        :tableData="tableRes.data"
        :columns="tableColumns"
        :show-pagination="true"
        :pagination-config="tableRes.pagination"
        @pagination-current-change="tableRes.pageCurrentChange"
        @pagination-size-change="tableRes.pageSizeChange"
      >
        <template #handle="{ scope }">
          <div>
            <span class="link-btn" @click="handleDetail(scope.row)">查看</span>
            <span class="link-btn" @click="handleEdit(scope.row)">编辑</span>
            <span class="link-btn" @click="handleDelete(scope.row)">删除</span>
          </div>
        </template>
      </fe-table>
    </fe-panel>
  </div>
</template>

<script setup lang="ts">
import { reactive, ref, onActivated } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import { ElMessage, ElMessageBox } from 'element-plus';
import { dictEscape, thousands, dateFormat } from '@fe/filter';
import { useGlobalStore } from '@/store/globalStore';
import api from '@/api/api';
import useAxios from '@/utils/useAxios';
import { TableColumn, useTable, useDate } from '@fe/element-component-library';

interface ITableSearch {
  orderNo: string;
  startTime: string;
  endTime: string;
  orderStatus: string;
}

interface ITableRow {
  id: number;
  orderNo: string;
  orderTitle: string;
  orderTime: string;
  orderStatus: string;
  orderAmount: number;
  createTime: string;
}

const router = useRouter();
const route = useRoute();
const store = useGlobalStore();

const orderStatusArr = ref<Dict[]>([]);

const tableColumns = reactive<TableColumn<ITableRow>[]>([
  { label: '订单编号', prop: 'orderNo', minWidth: '140' },
  { label: '订单标题', prop: 'orderTitle', minWidth: '140' },
  {
    label: '订单状态',
    prop: 'orderStatus',
    minWidth: '150',
    filter: dictEscape,
    filterParams: [orderStatusArr.value]
  },
  { label: '订单时间', prop: 'orderTime', minWidth: '150', filter: dateFormat },
  {
    label: '订单金额',
    prop: 'orderAmount',
    minWidth: '140',
    filter: thousands
  },
  { label: '操作', prop: 'handle', width: '160', fixed: 'right' }
]);

const tableRes = useTable<ITableRow, ITableSearch>({
  tableOptions: {
    orderNo: '',
    startTime: '',
    endTime: '',
    orderStatus: ''
  },
  axiosConfig: { method: 'post', url: api.getPageList }
});

const dateRes = useDate(tableRes.tableOptions, 'startTime', 'endTime');

const handleSearch = () => {
  tableRes.search();
};
const handleReset = () => {
  dateRes.reset();
  tableRes.reset();
};
const handleAdd = () => {
  router.push('/list/add');
};
const handleDetail = (row: ITableRow) => {
  router.push(`/list/detail/${row.id}`);
};
const handleEdit = (row: ITableRow) => {
  router.push(`/list/edit/${row.id}`);
};

const handleDelete = async (row: ITableRow) => {
  ElMessageBox.confirm('是否确认删除该数据', '提示', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning'
  }).then(async () => {
    await useAxios.post(api.deleteSome, { id: row.id });
    ElMessage.success('删除成功');
    handleSearch();
  });
};

const onInit = async () => {
  store.setBreadcrumb([{ title: '首页' }, { title: route.meta.title as string }]);
  orderStatusArr.value = store.getDict('order_status');
  tableRes.search();
};

onActivated(() => onInit());
</script>

<style lang="postcss" scoped></style>
