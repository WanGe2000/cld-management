<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="商品名">
              <a-input placeholder="请输入商品名" v-model="queryParam.name"></a-input>
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
      <a-button type="primary" icon="download" @click="handleExportXls('商品实体类')">导出</a-button>
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
        <template slot="avatarslot" slot-scope="text, record">
          <div class="anty-img-wrap">
            <a-avatar shape="square" :src="getAvatarView(record.img)" icon="user"/>
          </div>
        </template>
        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical" />
           <a @click="editState(record.id,1)" v-if="record.goodState==0">下架</a>
           <a @click="editState(record.id,0)" v-if="record.goodState==1">上架</a>
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
    <baseGood-modal ref="modalForm" @ok="modalFormOk"></baseGood-modal>
  </a-card>
</template>

<script>
  import BaseGoodModal from './modules/BaseGoodModal'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import importModal from '@/components/jeecg/import'
  import { httpAction, getAction,getFileAccessHttpUrl } from '@/api/manage'
  export default {
    name: "BaseGoodList",
    mixins:[JeecgListMixin],
    components: {
      BaseGoodModal,importModal
    },
    data () {
      return {
        description: '商品实体类管理页面',
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
            title: '商品图片',
            align: "center",
            width: 120,
            dataIndex: 'avatarslot',
            scopedSlots: {customRender: "avatarslot"}
          },
		   {
            title: '商品名',
            align:"center",
            dataIndex: 'name'
           },
		   {
            title: '商品类别',
            align:"center",
            dataIndex: 'baseGoodType.name'
           },
		   {
            title: '商品价格',
            align:"center",
            dataIndex: 'price'
           },
		   {
            title: '商品状态)',
            align:"center",
            dataIndex: 'goodState_dictText'
           },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            scopedSlots: { customRender: 'action' },
          }
        ],
		url: {
          imgerver: window._CONFIG['staticDomainURL'],
          list: "/base/baseGood/list",
          delete: "/base/baseGood/delete",
          deleteBatch: "/base/baseGood/deleteBatch",
          exportXlsUrl: "base/baseGood/exportXls",
          importExcelUrl: "base/baseGood/importExcel",
          downloadTepUrl:"base/baseGood/exportXls",
          editState:"base/baseGood/editState"
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
       getAvatarView: function (avatar) {
        return getFileAccessHttpUrl(avatar,this.url.imgerver,"http")
      },
     editState(id,state){
           getAction(this.url.editState,{
          id:id,state:state
        }).then(res=>{
          if(res.success){
             this.$message.info("修改成功")
             this.loadData(1);
          }
        })
     }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>