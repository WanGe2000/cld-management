<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <!-- <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="会员名">
              <a-input placeholder="请输入会员名" v-model="queryParam.name"></a-input>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
            </span>
          </a-col>

        </a-row> -->
      </a-form>
    </div>

    <!-- 操作按钮区域 -->
    <div class="table-operator" style='display:flex'>
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <!-- <a-button type="primary" icon="download" @click="handleExportXls('会员级别表')">导出</a-button>
      <import-modal :actionUpload="importExcelUrl" :actionDownload="downloadTepUrl" :headers="tokenHeader"></import-modal> -->
      <!-- <a-dropdown v-if="selectedRowKeys.length > 0"> -->
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>删除</a-menu-item>
        </a-menu>
        <!-- <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /></a-button> -->
      <!-- </a-dropdown> -->
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
    <baseVip-modal ref="modalForm" @ok="modalFormOk"></baseVip-modal>
  </a-card>
</template>

<script>
  import BaseVipModal from './modules/BaseVipModal'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
 import importModal from '@/components/jeecg/import'
  export default {
    name: "BaseVipList",
    mixins:[JeecgListMixin],
    components: {
      BaseVipModal,importModal
    },
    data () {
      return {
        description: '会员级别表管理页面',
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
            title: '会员名',
            align:"center",
            dataIndex: 'name'
           },
		   {
            title: '升级条件',
            align:"center",
            dataIndex: 'integral'
           },
            {
            title: '推荐积分',
            align:"center",
            dataIndex: 'num'
           },
		   {
            title: '折扣',
            align:"center",
            dataIndex: 'discount',
             customRender:(discount)=>{
              if(discount){
                return discount*100+"折"
              }
            }
           },
            {
            title: '分享效益',
            align:"center",
            dataIndex: 'benefit',
               customRender:(benefit)=>{
              if(benefit){
                return benefit*100+"%"
              }
            }
           },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            scopedSlots: { customRender: 'action' },
          }
        ],
		url: {
          list: "/base/baseVip/list",
          delete: "/base/baseVip/delete",
          deleteBatch: "/base/baseVip/deleteBatch",
          exportXlsUrl: "base/baseVip/exportXls",
          importExcelUrl: "base/baseVip/importExcel",
          downloadTepUrl:"base/baseVip/exportXls"
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