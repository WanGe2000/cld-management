<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">

          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="投票主题">
              <a-input placeholder="请输入投票主题" v-model="queryParam.title"></a-input>
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
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('投票')">导出</a-button>
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
          <a @click="handleEdit(record)"><a-icon type="edit"/>编辑</a>

          <a-divider type="vertical"/>
          <a @click="editVoteOption(record)"><a-icon type="setting"/> 选项配置</a>

          <a-divider type="vertical" />
          <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a><a-icon type="delete"/>删除</a>
                </a-popconfirm>

          <!--<a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down" /></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>-->
        </span>

      </a-table>
    </div>
    <!-- table区域-end -->

    <!-- 表单区域 -->
    <voteInfo-modal ref="modalForm" @ok="modalFormOk"></voteInfo-modal>
    <vote-option-list ref="voteOptionList"></vote-option-list>
  </a-card>
</template>

<script>
  import VoteInfoModal from './modules/VoteInfoModal'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import moment from 'moment'
  import VoteOptionList from './VoteOptionList'

  export default {
    name: "VoteInfoList",
    mixins:[JeecgListMixin],
    components: {
      VoteInfoModal,VoteOptionList
    },
    data () {
      return {
        description: '投票管理页面',
        // 表头
        columns: [
		   {
            title: '投票主题',
            align:"center",
            dataIndex: 'title'
           },
          {
            title: '发起日期',
            align:"center",
            dataIndex: 'createTime',
            customRender: (text) => {
              return moment(text).format('YYYY-MM-DD HH:mm')
            }
          },
		    {
            title: '截止日期',
            align:"center",
            dataIndex: 'endTime',

           },
		    {
            title: '所属门店',
            align:"center",
            dataIndex: 'departId'
           },
          {
            title: '状态',
            align:"center",
            dataIndex: 'voteStatus_dictText'
          },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            scopedSlots: { customRender: 'action' },
          }
        ],
		url: {
          list: "/vote/voteInfo/list",
          delete: "/vote/voteInfo/delete",
          deleteBatch: "/vote/voteInfo/deleteBatch",
          exportXlsUrl: "vote/voteInfo/exportXls",
          importExcelUrl: "vote/voteInfo/importExcel",
       },
    }
  },
  computed: {
    importExcelUrl: function(){
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
    }
  },
    methods: {
      editVoteOption(record){
        /*this.$refs.VoteOptionList.edit(record);*/
        this.$refs.voteOptionList.edit(record);
      }
     
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>