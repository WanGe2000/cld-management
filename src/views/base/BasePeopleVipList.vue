<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="姓名">
              <a-input placeholder="请输入姓名" v-model="queryParam.name"></a-input>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="身份证号码">
              <a-input placeholder="请输入身份证号码" v-model="queryParam.idCard"></a-input>
            </a-form-item>
          </a-col>
      
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'"/>
              </a>
            </span>
          </a-col>

        </a-row>
      </a-form>
    </div>

    <!-- 操作按钮区域 -->
    <div class="table-operator" style='display:flex'>
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('用户表')">导出</a-button>
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
            <a-avatar shape="square" :src="getAvatarView(record.avatar)" icon="user"/>
          </div>
        </template>
        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical" />
           <a @click="edit(record,1)" v-if="record.status==0">禁用</a>
           <a @click="edit(record,0)" v-if="record.status==1">启动</a>
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
    <basePeople-modal ref="modalForm" @ok="modalFormOk"></basePeople-modal>
  </a-card>
</template>

<script>
  import BasePeopleModal from './modules/BasePeopleModal'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import importModal from '@/components/jeecg/import'
  import {putAction,getFileAccessHttpUrl} from '@/api/manage'
  import { httpAction, getAction } from '@/api/manage'
  export default {
    name: "BasePeopleVipList",
    mixins:[JeecgListMixin],
    components: {
     BasePeopleModal,importModal
    },
    data () {
      return {
        description: '会员表管理页面',
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
            title: '头像',
            align: "center",
            width: 120,
            dataIndex: 'avatar',
            scopedSlots: {customRender: "avatarslot"}
          },
             {
            title: '昵称',
            align:"center",
            dataIndex: 'nickname'
           },
		      {
            title: '会员姓名',
            align:"center",
            dataIndex: 'name'
           },
         {
            title: '身份证号码',
            align:"center",
            dataIndex: 'idCard'
           },
		   {
            title: '电话号码',
            align:"center",
            dataIndex: 'iphone'
           },
		   {
            title: 'vip类型',
            align:"center",
            dataIndex: 'baseVip.name'
           },
      //        {
      //       title: '开通时间',
      //       align:"center",
      //       dataIndex: 'createTime'
      //      },
      //        {
      //       title: '开通门店',
      //       align:"center",
      //       dataIndex: 'sysDepart.departName'
      //      },
		  //  {
      //       title: '电话号码',
      //       align:"center",
      //       dataIndex: 'iphone'
      //      },
            {
            title: '会员状态',
            align:"center",
            dataIndex: 'status_dictText'
           },
          //   {
          //   title: '推荐会员数',
          //   align:"center",
          //   dataIndex: 'num'
          //  },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            scopedSlots: { customRender: 'action' },
          }
        ],
		url: {
          list: "/base/basePeople/vipList",
          delete: "/base/basePeople/delete",
          deleteBatch: "/base/basePeople/deleteBatch",
          exportXlsUrl: "base/basePeople/exportXls",
          importExcelUrl: "base/basePeople/importExcel",
          downloadTepUrl:"base/basePeople/exportXls",
          editState:"base/basePeople/editState"
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
      edit(record,state){
             getAction(this.url.editState,{
          id:record.id,state:state
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