<template>
  <div>
    <a-button type="primary" icon="download" @click="visible = true">导入</a-button>
    <a-modal title="导入" v-model="visible" @ok="handleOk" okText="提交"  :bodyStyle="{
        'padding':'0 24px 24px'
    }">
      <span style='color:#1890ff;display:block;margin:10px 0;cursor:pointer' @click="handleDownloadTep">下载导入模版</span>
      <a-upload-dragger
        :name="name"
        :headers="headers"
        :multiple="multiple"
        :showUploadList="showUploadList"
        :action="actionUpload"
        @change="handleChange"
      >
        <p class="ant-upload-drag-icon">
          <a-icon type="inbox" />
        </p>
        <p class="ant-upload-text">点击或拖拽文件到此处进行上传</p>
      </a-upload-dragger>
    </a-modal>
  </div>
</template>
<script>
import { downFile } from '@/api/manage'
export default {
  props: {
    actionUpload: {
      //上传的接口
      required: true,
      type: String
    },
    actionDownload: {
      //下载模版的接口
      required: true,
      type: String
    },
    headers: {
      required: true
    },
    name: {
      //后台接收的filename
      default: 'file',
      type: String
    },
    multiple: {
      //是否支持多选文件
      default: true,
      type: Boolean
    },
    showUploadList: {
      //是否显示已上传文件
      default: true,
      type: Boolean
    }
  },
  data() {
    return {
      visible: false
    }
  },
  methods: {
    handleChange(info) {
      const status = info.file.status
      this.$emit('fileChange',info)
      if (status === 'done') {
        this.$message.success(`${info.file.name} 上传成功`)
      } else if (status === 'error') {
        this.$message.error(`${info.file.name} 上传失败`)
      }
    },
    handleOk(){
      this.$emit('ok')
      this.visible = false
    },
    handleDownloadTep(){
      downFile(this.actionDownload,{}).then((data)=>{
        if (!data) {
          this.$message.warning("文件下载失败")
          return
        }
        if (typeof window.navigator.msSaveBlob !== 'undefined') {
          window.navigator.msSaveBlob(new Blob([data]), '导入模版.xls')
        }else{
          let url = window.URL.createObjectURL(new Blob([data]))
          let link = document.createElement('a')
          link.style.display = 'none'
          link.href = url
          link.setAttribute('download', '导入模版.xls')
          document.body.appendChild(link)
          link.click()
          document.body.removeChild(link); //下载完成移除元素
          window.URL.revokeObjectURL(url); //释放掉blob对象
        }
      })
    },
  }
}
</script>