<template>
  <a-modal
    :title="title"
    :width="800"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">
    
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="内容">
          <a-textarea placeholder="请输入内容" :disabled="model.id?true:false" :rows="4" v-decorator="['content', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="相关图片">
          <a-upload
            listType="picture-card"
            class="avatar-uploader"
            :showUploadList="false"
            :openFileDialogOnClick="model.id?false:true"
            :action="uploadAction"
            :data="{'isup':1}"
            :headers="headers"
            :beforeUpload="beforeUpload"
            @change="handleChange"
          >
            <img v-if="picUrl" :src="getImgView()" alt="头像" style="height:104px;max-width:300px"/>
            <div v-else>
              <a-icon :type="uploadLoading ? 'loading' : 'plus'" />
              <div class="ant-upload-text">上传</div>
            </div>
          </a-upload>
          <!--<a-input placeholder="请输入相关图片" v-decorator="['img', {}]" />-->
        </a-form-item>



        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="户室信息">
          <a-input placeholder="请输入户室信息" :disabled="model.id?true:false" v-decorator="['adress', {}]" />
        </a-form-item>
        <a-form-item
          v-if="model.id"
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="是否公示">
          <a-switch checkedChildren="是" unCheckedChildren="否" v-model="ifPublic" />
        </a-form-item>

        <a-form-item
          v-if="model.id"
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="回复内容">
          <a-textarea placeholder="请输入回复内容" :disabled="model.response?true:false" :rows="4" v-decorator="['response', {}]" />
        </a-form-item>
		
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"
  import { ACCESS_TOKEN } from "@/store/mutation-types"
  import Vue from 'vue'

  export default {
    name: "SuggestModal",
    data () {
      return {
        ifPublic:false,
        title:"操作",
        visible: false,
        model: {},
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },

        uploadLoading:false,
        confirmLoading: false,
        headers:{},
        picUrl: "",
        form: this.$form.createForm(this),
        validatorRules:{
        },
        url: {
          add: "/serve/suggest/add",
          edit: "/serve/suggest/edit",
          fileUpload: window._CONFIG['domianURL']+"/sys/common/upload",
          imgerver: window._CONFIG['domianURL']+"/sys/common/view",
        },
      }
    },
    created () {
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token":token}
    },
    methods: {
      add () {
        this.picUrl = "";
        this.edit({});
      },
      edit (record) {
        if(record.hasOwnProperty("id")){
          this.picUrl = "Has no pic url yet";
        }
        this.ifPublic = record.ifPublic!=null?record.ifPublic:false;
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'delFlag','tenantId','content','img','userId','status','adress','ifPublic','response'))
		  //时间格式化
        });

      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleOk () {
        this.model.ifPublic=this.ifPublic;
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.id){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
               method = 'put';
            }
            let formData = Object.assign(this.model, values);
            //时间格式化
            
            console.log(formData)
            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
              that.close();
            })



          }
        })
      },
      handleCancel () {
        this.close()
      },
      beforeUpload: function(file){
        var fileType = file.type;
        if(fileType.indexOf('image')<0){
          this.$message.warning('请上传图片');
          return false;
        }
        //TODO 验证文件大小
      },
      handleChange (info) {
        this.picUrl = "";
        if (info.file.status === 'uploading') {
          this.uploadLoading = true;
          return
        }
        if (info.file.status === 'done') {
          var response = info.file.response;
          this.uploadLoading = false;
          if(response.success){
            this.model.img = response.message;
            this.picUrl = "Has no pic url yet";
          }else{
            this.$message.warning(response.message);
          }
        }
      },
      getImgView(){
        return this.url.imgerver +"/"+ this.model.img;
      },


    },
    computed:{
      uploadAction:function () {
        return this.url.fileUpload;
      }
    },
  }
</script>

<style lang="less" scoped>

</style>