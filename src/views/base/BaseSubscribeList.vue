<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="预约人">
              <a-input placeholder="请输入预约人" v-model="queryParam.peopleId"></a-input>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="服务名">
              <a-input placeholder="请输入服务名" v-model="queryParam.serviceId"></a-input>
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
      <a-button type="primary" icon="download" @click="handleExportXls('预约实体')">导出</a-button>
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
          <a @click="showDetail(record)">详情</a>
          <a-divider type="vertical" />
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
    <baseSubscribe-modal ref="modalForm" @ok="modalFormOk"></baseSubscribe-modal>
    <a-drawer title="详情" :width="800" placement="right" :closable="false" :visible="visible">
      <h3>订单信息</h3>
      <div style="margin-left:30px">
        <a-row>
          <a-col :span="12">
            <span>创建时间:</span>
            <span>{{target.createTime}}</span>
          </a-col>
          <a-col :span="12">
            <span>订单编号:</span>
            <span>{{target.id}}</span>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12">
            <span>订单类型:</span>
            <span>预约订单</span>
          </a-col>
          <a-col :span="12">
            <span>预约门店:</span>
            <span>{{target.sysDepart && target.sysDepart.departName}}</span>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12">
            <span>应付金额:</span>
            <span>1</span>
          </a-col>
          <a-col :span="12">
            <span>订单状态:</span>
            <span>{{target.reserveType_dictText}}</span>
          </a-col>
        </a-row>
      </div>

      <a-divider />
      <h3>服务信息</h3>
      <div style="margin-left:30px">
         <a-row>
          <a-col :span="12">
            <span>服务名称:</span>
            <span>{{target.baseService && target.baseService.name}}</span>
          </a-col>
          <a-col :span="12">
            <span>保养师:</span>
            <span>{{target.sysUser && target.sysUser.realname}}</span>
          </a-col>
        </a-row>
         <a-row>
          <a-col :span="12">
            <span>到店时间:</span>
            <span>{{target.time}}</span>
          </a-col>
          <a-col :span="12">
            <span>客户评论:</span>
            <span></span>
            
          </a-col>
        </a-row>
         <a-row>
          <a-col :span="24">
            <span>保养照片:</span>
            <a-row :gutter="16">
              <a-col v-for="item in 4" :key="item"  :span="6">
                <img src="http://img2.imgtn.bdimg.com/it/u=132500056,2200488628&fm=26&gp=0.jpg">
              </a-col>
            </a-row>
          </a-col>
        </a-row>
      </div>

      <a-divider style="margin-top:30px" />
      <h3>用户信息</h3>
      <div style="margin-left:30px">
        <a-row>
          <a-col :span="8">
            <span>昵称:</span>
            <span>{{target.basePeople && target.basePeople.nickname}}</span>
          </a-col>
          <a-col :span="8">
            <span>电话:</span>
            <span>{{target.iphone}}</span>
          </a-col>
          <a-col :span="8">
            <span>会员类型:</span>
            <span>123123123123</span>
          </a-col>
        </a-row>
      </div>
      <a-button type="primary" @click="visible = false">关闭</a-button>
    </a-drawer>
  </a-card>
</template>

<script>
  import BaseSubscribeModal from './modules/BaseSubscribeModal'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
 import importModal from '@/components/jeecg/import'
  export default {
    name: "BaseSubscribeList",
    mixins:[JeecgListMixin],
    components: {
      BaseSubscribeModal,importModal
    },
    data () {
      return {
        description: '预约实体管理页面',
        visible:false,
        target:{},
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
            title: '订单编号',
            align:"center",
            dataIndex: 'id'
           },
		   {
            title: '客户姓名',
            align:"center",
            dataIndex: 'basePeople.name'
           },
            {
            title: '手机号',
            align:"center",
            dataIndex: 'basePeople.iphone'
           },
            {
            title: '预约门店',
            align:"center",
            dataIndex: 'sysDepart.departName'
           },
		   {
            title: '服务名',
            align:"center",
            dataIndex: 'baseService.name'
           },
		   {
            title: '店员',
            align:"center",
            dataIndex: 'sysUser.realname'
           },
		   {
            title: '到店时间',
            align:"center",
            dataIndex: 'time'
           },
            {
            title: '预约状态',
            align:"center",
            dataIndex: 'reserveType_dictText'
           },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            width:180,
            scopedSlots: { customRender: 'action' },
          }
        ],
		url: {
          list: "/base/baseSubscribe/list",
          delete: "/base/baseSubscribe/delete",
          deleteBatch: "/base/baseSubscribe/deleteBatch",
          exportXlsUrl: "base/baseSubscribe/exportXls",
          importExcelUrl: "base/baseSubscribe/importExcel",
          downloadTepUrl:"base/baseSubscribe/exportXls"
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
     showDetail(item) {
      console.log(item);
      this.target = item;
      this.visible = true;
    }
    }
  }
</script>
<style scoped lang='less'>
  @import '~@assets/less/common.less';
  .ant-drawer-content-wrapper {
  .ant-btn {
    margin-left: 30px;
    margin-bottom: 30px;
    margin-top: 30px;
    float: right;
  }
  .ant-row {
    height: 40px;
    line-height: 40px;
  }
  img {
    width: 100%;
  }
}
</style>