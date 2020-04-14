<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">
          <a-col :md="6" :sm="8">
            <a-form-item label="关键词">
              <a-input placeholder="请输入关键词" v-model="queryParam.name"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="按状态筛选">
              <a-select v-model="queryParam.tenantState" placeholder="请选择状态查询">
                <a-select-option value>请选择状态查询</a-select-option>
                <a-select-option value="0">正常</a-select-option>
                <a-select-option value="1">已到期</a-select-option>
                <a-select-option value="2">即将到期</a-select-option>
                <a-select-option value="3">禁用</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button
                type="primary"
                @click="searchReset"
                icon="reload"
                style="margin-left: 8px"
              >重置</a-button>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('校园管理')">导出</a-button>
      <a-upload
        name="file"
        :showUploadList="false"
        :multiple="false"
        :headers="tokenHeader"
        :action="importExcelUrl"
        @change="handleImportExcel"
      >
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel">
            <a-icon type="delete" />删除
          </a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px">
          批量操作
          <a-icon type="down" />
        </a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择
        <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
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
        @change="handleTableChange"
      >
        <template slot="logoslot" slot-scope="text, record">
          <div class="anty-img-wrap">
            <a-avatar shape="square" :src="getAvatarView(record.logo)" icon="user" />
          </div>
        </template>

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>
          <a-divider type="vertical" />
          <a @click="handlePerssion(record)">授权服务</a>
          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">
              更多
              <a-icon type="down" />
            </a>
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
    <sysTenant-modal ref="modalForm" @ok="modalFormOk"></sysTenant-modal>

    <a-modal title="服务列表" v-model="visible" @ok="handleOk">
      <a-checkbox :indeterminate="indeterminate" @change="onCheckAllChange" :checked="checkAll">全选</a-checkbox>
      <br />
      <a-checkbox-group v-model="checkedList">
        <a-checkbox v-for="item in items" :key="item.id" :value="item.id">{{item.name}}</a-checkbox>
      </a-checkbox-group>
    </a-modal>
  </a-card>
</template>

<script>
import SysTenantModal from './modules/SysTenantModal'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { httpAction, getAction } from '@/api/manage'
export default {
  name: 'SysTenantList',
  mixins: [JeecgListMixin],
  components: {
    SysTenantModal
  },
  created() {
    this.getList()
  },
  data() {
    return {
      checkedList: [],
      indeterminate: true,
      checkAll: false,
      visible: false,
      description: '学校管理管理页面',
      id:"",
      items: [],
      // 表头
      columns: [
        {
          title: '序号',
          dataIndex: '',
          key: 'rowIndex',
          width: 60,
          align: 'center',
          customRender: function(t, r, index) {
            return parseInt(index) + 1
          }
        },
        {
          title: '租户名称',
          align: 'center',
          dataIndex: 'name'
        },
        {
          title: 'Logo',
          align: 'center',
          width: 120,
          dataIndex: 'logo',
          scopedSlots: { customRender: 'logoslot' }
        },
        {
          title: '用户名',
          align: 'center',
          dataIndex: 'sysUser.username'
        },
        {
          title: '状态',
          align: 'center',
          dataIndex: 'tenantState_dictText'
        },
        {
          title: '开始时间',
          align: 'center',
          dataIndex: 'startTime'
        },
        {
          title: '结束时间',
          align: 'center',
          dataIndex: 'endTime'
        },
        {
          title: '服务周期',
          align: 'center',
          dataIndex: 'servicePeriod'
        },
        // {
        //      title: '对接人',
        //      align:"center",
        //      dataIndex: 'username'
        //     },
        {
          title: '联系方式',
          align: 'center',
          dataIndex: 'phone'
        },
        {
          title: '操作',
          dataIndex: 'action',
          align: 'center',
          scopedSlots: { customRender: 'action' }
        }
      ],
      url: {
        list: '/sys/sysTenant/list',
        delete: '/sys/sysTenant/delete',
        deleteBatch: '/sys/sysTenant/deleteBatch',
        exportXlsUrl: 'sys/sysTenant/exportXls',
        importExcelUrl: 'sys/sysTenant/importExcel',
        imgerver: window._CONFIG['domianURL'] + '/sys/common/view',
        getItemList: '/system/sysItem/list',
        editItem: '/sys/sysTenant/editItem'
      }
    }
  },
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    }
  },
  methods: {
    getAvatarView: function(avatar) {
      return this.url.imgerver + '/' + avatar
    },
    handlePerssion(e) {
      this.checkedList = e.list.map(item => item.id)
      this.id=e.id;
      this.checkAll = this.items.length == this.checkedList.length
      this.visible = true
    },
    getList() {
      getAction(this.url.getItemList).then(res => {
        if (res.success) {
          this.items = res.result.records
        }
      })
    },
    onCheckAllChange(e) {
      Object.assign(this, {
        checkedList: e.target.checked ? this.items.map(item => item.id) : [],
        indeterminate: false,
        checkAll: e.target.checked
      })
    },
    handleOk() {
      getAction(this.url.editItem, {
        id:this.id,
        items: this.checkedList.toString()
      }).then(res => {
        if (res.success) {
          this.visible = false
          this.loadData()
        }
      })
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>