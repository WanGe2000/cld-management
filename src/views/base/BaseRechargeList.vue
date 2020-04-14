<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="充值方式">
              <a-input placeholder="请输入充值方式" v-model="queryParam.rechargeMode"></a-input>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>

    <!-- 操作按钮区域 -->
    <div class="table-operator" style='display:flex'>
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('充值订单')">导出</a-button>
      <import-modal :actionUpload="importExcelUrl" :actionDownload="downloadTepUrl" :headers="tokenHeader"></import-modal>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>删除</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /></a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
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

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down" /></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>

      </a-table>
    </div>
    <!-- table区域-end -->

    <!-- 表单区域 -->
    <baseRechargeOrder-modal ref="modalForm" @ok="modalFormOk"></baseRechargeOrder-modal>
  </a-card>
</template>

<script>
  import BaseRechargeOrderModal from './modules/BaseRechargeOrderModal'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
 import importModal from '@/components/jeecg/import'
  export default {
    name: "BaseRechargeOrderList",
    mixins:[JeecgListMixin],
    components: {
      BaseRechargeOrderModal,importModal
    },
    data () {
      return {
        description: '充值订单管理页面',
        // 表头
        columns: [
          {
            title: '#',
            dataIndex: '',
            key:'rowIndex',
            width:60,
            align:"center",
            customRender:function (t,r,index) {
              return parseInt(index)+1;
            }
           },
            {
            title: '充值人员',
            align:"center",
            dataIndex: 'peopleId'
           },
              {
            title: '充值金额',
            align:"center",
            dataIndex: 'money'
           },
		   {
            title: '充值方式',
            align:"center",
            dataIndex: 'rechargeMode'
           },
		   {
            title: '充值状态',
            align:"center",
            dataIndex: 'rechargeState'
           },
             {
            title: '门店',
            align:"center",
            dataIndex: 'departId'
           },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            scopedSlots: { customRender: 'action' },
          }
        ],
		url: {
          list: "/base/baseRechargeOrder/list",
          delete: "/base/baseRechargeOrder/delete",
          deleteBatch: "/base/baseRechargeOrder/deleteBatch",
          exportXlsUrl: "base/baseRechargeOrder/exportXls",
          importExcelUrl: "base/baseRechargeOrder/importExcel",
          downloadTepUrl:"base/baseRechargeOrder/exportXls"
       },
    }
  },
  computed: {
     importExcelUrl: function(){
       return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
     },
 downloadTepUrl: function(){
       return `${window._CONFIG['domianURL']}/${this.url.downloadTepUrl}`;
     }
   },
    methods: {
     
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>