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
    <div class="table-operator" style="display:flex">
      <!-- <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button> -->
      <a-button type="primary" icon="download" @click="handleExportXls('订单实体类')">导出</a-button>
      <import-modal
        :actionUpload="importExcelUrl"
        :actionDownload="downloadTepUrl"
        :headers="tokenHeader"
      ></import-modal>
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
        <span slot="action" slot-scope="text, record">
          <a @click="showDetail(record)">详情</a>
          <a-divider type="vertical" />

          <a @click="handleEdit(record)">编辑</a>

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
    <baseOrder-modal ref="modalForm" @ok="modalFormOk"></baseOrder-modal>
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
            <span>{{target.orderId}}</span>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12">
            <span>订单类型:</span>
            <span>{{target.orderType_dictText}}</span>
          </a-col>
          <a-col :span="12">
            <span>支付方式:</span>
            <span>{{target.getInPassway}}</span>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12">
            <span>下单门店:</span>
            <span>{{target.sysDepart && target.sysDepart.departName}}</span>
          </a-col>
          <a-col :span="12">
            <span>金额:</span>
            <span>{{target.price}}</span>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="12">
            <span>支付单号:</span>
            <span>123</span>
          </a-col>
          <a-col :span="12">
            <span>支付状态:</span>
            <span>未支付</span>
          </a-col>
        </a-row>
      </div>

      <a-divider />
      <h3>商品明细</h3>
      <div style="margin-left:30px">
        <a-row>
          <a-col :span="8">
            <span>商品</span>
          </a-col>
          <a-col :span="8">
            <span>单价</span>
          </a-col>
          <a-col :span="8">
            <span>数量</span>
          </a-col>
        </a-row>
        <a-row v-for="item in target.goodList" :key="item.id">
          <a-col :span="8">
            <span>{{item.name}}</span>
          </a-col>
          <a-col :span="8">
            <span>{{item.price}}</span>
          </a-col>
          <a-col :span="8">
            <span>{{item.count}}</span>
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
import BaseOrderModal from "./modules/BaseOrderModal";
import { JeecgListMixin } from "@/mixins/JeecgListMixin";
import importModal from "@/components/jeecg/import";
import JSearchSelectTag from "@/components/dict/JSearchSelectTag";
export default {
  name: "BaseOrderList",
  mixins: [JeecgListMixin],
  components: {
    BaseOrderModal,
    importModal,
    JSearchSelectTag
  },
  data() {
    return {
      description: "订单实体类管理页面",
      visible: false,
      target: {},
      // 表头
      columns: [
        {
          title: "#",
          dataIndex: "",
          key: "rowIndex",
          width: 60,
          align: "center",
          customRender: function(t, r, index) {
            return parseInt(index) + 1;
          }
        },
        {
          title: "创建时间",
          align: "center",
          dataIndex: "createTime"
        },
        {
          title: "订单编号",
          align: "center",
          dataIndex: "orderId"
        },
        {
          title: "订单类型",
          align: "center",
          dataIndex: "orderType_dictText"
        },
        {
          title: "客户姓名",
          align: "center",
          dataIndex: "basePeople.name"
        },
        {
          title: "电话号码",
          align: "center",
          dataIndex: "basePeople.iphone"
        },

        {
          title: "门店",
          align: "center",
          dataIndex: "sysDepart.departName"
        },
        {
          title: "商品信息",
          align: "center",
          dataIndex: "goodList.get(0).name"
        },
        {
          title: "操作",
          dataIndex: "action",
          align: "center",
          width: 180,
          scopedSlots: { customRender: "action" }
        }
      ],
      url: {
        list: "/base/baseOrder/list",
        delete: "/base/baseOrder/delete",
        deleteBatch: "/base/baseOrder/deleteBatch",
        exportXlsUrl: "base/baseOrder/exportXls",
        importExcelUrl: "base/baseOrder/importExcel",
        downloadTepUrl: "base/baseOrder/exportXls"
      }
    };
  },
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG["domianURL"]}/${this.url.importExcelUrl}`;
    },
    downloadTepUrl: function() {
      return `${window._CONFIG["domianURL"]}/${this.url.downloadTepUrl}`;
    }
  },
  methods: {
    showDetail(item) {
      console.log(item);
      this.target = item;
      this.visible = true;
    }
  }
};
</script>
<style scoped lang='less'>
@import "~@assets/less/common.less";
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