<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">
          <a-col :md="6" :sm="8">
            <a-form-item label="内容">
              <a-input placeholder="请输入内容" v-model="queryParam.content"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="处理状态">
              <a-select v-model="queryParam.status" placeholder="请选择处理状态">
                <a-select-option value="true">已处理</a-select-option>
                <a-select-option value="false">未处理</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>

          <template v-if="toggleSearchStatus">

            <a-col :md="6" :sm="8">
              <a-form-item label="投诉用户id">
                <a-input placeholder="请输入投诉用户id" v-model="queryParam.userId"></a-input>
              </a-form-item>
            </a-col>
          </template>
          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
             <!-- <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'"/>
              </a>-->
            </span>
          </a-col>

        </a-row>
      </a-form>
    </div>

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <!--<a-button type="primary" icon="download" @click="handleExportXls('投诉建议')">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl"
                @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>-->
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

        <template slot="imgslot" slot-scope="text, record">
          <div class="anty-img-wrap">
            <a-avatar shape="square" :src="getAvatarView(record.img)" icon="user"/>
          </div>
        </template>

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">回复</a>

          <a-divider type="vertical"/>
          <!--<a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down"/></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>-->
           <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
            <a>删除</a>
          </a-popconfirm>
        </span>

      </a-table>
    </div>
    <!-- table区域-end -->

    <!-- 表单区域 -->
    <suggest-modal ref="modalForm" @ok="modalFormOk"></suggest-modal>
  </a-card>
</template>

<script>
  import SuggestModal from './modules/SuggestModal'
  import {JeecgListMixin} from '@/mixins/JeecgListMixin'

  export default {
    name: "SuggestList",
    mixins: [JeecgListMixin],
    components: {
      SuggestModal
    },
    data() {
      return {
        description: '投诉建议管理页面',
        // 表头
        columns: [

          {
            title: '内容',
            align: "center",
            dataIndex: 'content'
          },
          {
            title: '相关图片',
            align: "center",
            dataIndex: 'img',
            scopedSlots: {customRender: "imgslot"}
          },
          {
            title: '投诉用户',
            align: "center",
            dataIndex: 'userName'
          },
          {
            title: '投诉时间',
            align: "center",
            dataIndex: 'createTime'
          },
          {
            title: '投诉状态',
            align: "center",
            dataIndex: 'status',
            customRender: (text) => {
              return text ? "已处理" : "未处理";
            }
          },
          {
            title: '户室信息',
            align: "center",
            dataIndex: 'adress'
          },
          {
            title: '是否公示',
            align: "center",
            dataIndex: 'ifPublic',
            customRender: (text) => {
              return text ? "是" : "否";
            }
          },
          {
            title: '操作',
            dataIndex: 'action',
            align: "center",
            scopedSlots: {customRender: 'action'},
          }
        ],
        url: {
          imgerver: window._CONFIG['domianURL'] + "/sys/common/view",
          list: "/serve/suggest/list",
          delete: "/serve/suggest/delete",
          deleteBatch: "/serve/suggest/deleteBatch",
          exportXlsUrl: "serve/suggest/exportXls",
          importExcelUrl: "serve/suggest/importExcel",
        },
      }
    },
    computed: {
      importExcelUrl: function () {
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      }
    },
    methods: {
      getAvatarView: function (avatar) {
        return this.url.imgerver + "/" + avatar;
      },
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>