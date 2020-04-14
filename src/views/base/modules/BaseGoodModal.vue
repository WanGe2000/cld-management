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
          label="商品名">
          <a-input placeholder="请输入商品名" v-decorator="['name', {}]" />
        </a-form-item>
         <a-form-item label="商品图片" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-image-upload class="avatar-uploader" text="上传" v-model="fileList" ></j-image-upload>
        </a-form-item>
       <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="商品类型">
          <j-search-select-tag
            placeholder="商品类型"
            v-decorator="['goodTypeId', {rules: [{required: true, message: '商品类型!'}]}]"
            :dict="dicGoodType">
          </j-search-select-tag>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="商品价格">
          <a-input-number v-decorator="[ 'price', {}]" />
        </a-form-item>
      <a-form-item label="商品状态" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-search-select-tag
            placeholder="请选择"
            dict="good_state"
            v-decorator="['goodState', { rules: [{ required: true, message: '不为空' }] },]"
          ></j-search-select-tag>
      </a-form-item>
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"
  import JSearchSelectTag from '@/components/dict/JSearchSelectTag'
  import JImageUpload from '../../../components/jeecg/JImageUpload'
  export default {
    name: "BaseGoodModal",
     components: {
    JSearchSelectTag,
    JImageUpload
    },
    data () {
      return {
        dicGoodType:"base_good_type,name,id,del_flag=0 and tenant_id="+JSON.parse(localStorage.getItem('userInfo')).id,
        title:"操作",
        visible: false,
        uploadLoading:false,
        fileList:[],
        model: {},
        picUrl: "",
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },

        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules:{
        },
        url: {
          fileUpload: window._CONFIG['domianURL']+"/sys/common/upload",
          imgerver: window._CONFIG['staticDomainURL'],
          add: "/base/baseGood/add",
          edit: "/base/baseGood/edit",
        },
      }
    },
     computed:{
      uploadAction:function () {
        return this.url.fileUpload;
      }
    },
    created () {
    },
    methods: {
      add () {
        this.edit({});
      },
      edit (record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        if(record.hasOwnProperty("id")){
            this.fileList = record.img;
        }
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'tenantId','img','departId','delFlag','name','goodTypeId','price','goodState'))
		  //时间格式化
        });

      },
   
      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleOk () {
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
            if(this.fileList){
              this.model.img = this.fileList
            }
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


    }
  }
</script>

<style lang="less" scoped>

</style>