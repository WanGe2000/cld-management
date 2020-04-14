<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">

          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="机构名称">
              <a-input placeholder="请输入机构名称" v-model="queryParam.departName"></a-input>
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
          </template>

          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
<!--              <a @click="handleToggleSearch" style="margin-left: 8px">-->
<!--                {{ toggleSearchStatus ? '收起' : '展开' }}-->
<!--                <a-icon :type="toggleSearchStatus ? 'up' : 'down'"/>-->
<!--              </a>-->
            </span>
          </a-col>

        </a-row>
      </a-form>
    </div>

    <!-- 操作按钮区域 -->
    <div class="table-operator" style='display:flex'>
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel">
            <a-icon type="delete"/>
            删除
          </a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作
          <a-icon type="down"/>
        </a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{
        selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        @change="handleTableChange">

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical"/>
          <a @click="handleDelete(record.id)">删除</a>
          <a-divider type="vertical"/>
          <a @click="showAdminUserModul(record.id,record.orgType)">管理员</a>
        </span>

      </a-table>
    </div>
    <!-- table区域-end -->

    <!-- 表单区域 -->
    <depart-modal ref="modalForm" @ok="modalFormOk"></depart-modal>
    <user-tree-modal ref="userTreeForm"></user-tree-modal>
  </a-card>
</template>

<script>
  import DepartModal from './modules/DepartModal'
  import UserTreeModal from './modules/UserTreeModal'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import importModal from '@/components/jeecg/import'

  export default {
    name: 'departList',
    mixins: [JeecgListMixin],
    components: {
      DepartModal, importModal,UserTreeModal
    },
    data() {
      return {
        description: '机构管理管理页面',
        // 表头
        columns: [
          {
            title: '机构名称',
            align: 'center',
            dataIndex: 'departName'
          },
          {
            title: '机构编码',
            align: 'center',
            dataIndex: 'orgCode'
          },
          {
            title: '手机号',
            align: 'center',
            dataIndex: 'mobile'
          },
          {
            title: '地址',
            align: 'center',
            dataIndex: 'address'
          },
          {
            title: '操作',
            dataIndex: 'action',
            align: 'center',
            scopedSlots: { customRender: 'action' }
          }
        ],
        url: {
          list: '/sys/sysDepart/list',
          delete: '/sys/sysDepart/delete',
          deleteBatch: '/sys/sysDepart/deleteBatch',
          exportXlsUrl: 'sys/sysDepart/exportXls',
          importExcelUrl: 'sys/sysDepart/importExcel',
          downloadTepUrl: 'sys/sysDepart/exportXls'
        }
      }
    },
    computed: {
      importExcelUrl: function() {
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
      },
      downloadTepUrl: function() {
        return `${window._CONFIG['domianURL']}/${this.url.downloadTepUrl}`
      }
    },
    methods: {
      showAdminUserModul(id,type){
        this.$refs.userTreeForm.loadTree(id,type);
        this.$refs.userTreeForm.title = "编辑管理员";
        this.$refs.userTreeForm.disableSubmit = false;
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>