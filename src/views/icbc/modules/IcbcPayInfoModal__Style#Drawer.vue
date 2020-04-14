<template>
  <a-drawer
    :title="title"
    :width="width"
    placement="right"
    :closable="false"
    @close="close"
    :visible="visible">
  
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">

        <a-form-item label="应用id" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'appid', validatorRules.appid]" placeholder="请输入应用id"></a-input>
        </a-form-item>
        <a-form-item label="网关公钥" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-textarea v-decorator="['apigwPublicKey']" rows="4" placeholder="请输入网关公钥"/>
        </a-form-item>
        <a-form-item label="应用私钥" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-textarea v-decorator="['privateKey']" rows="4" placeholder="请输入应用私钥"/>
        </a-form-item>
        <a-form-item label="应用公钥" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-textarea v-decorator="['publicKey']" rows="4" placeholder="请输入应用公钥"/>
        </a-form-item>
        <a-form-item label="签名类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'signType', validatorRules.signType]" placeholder="请输入签名类型"></a-input>
        </a-form-item>
        <a-form-item label="加密类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'encryptType', validatorRules.encryptType]" placeholder="请输入加密类型"></a-input>
        </a-form-item>
        <a-form-item label="加密密钥" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'encryptKey', validatorRules.encryptKey]" placeholder="请输入加密密钥"></a-input>
        </a-form-item>
        <a-form-item label="证书私钥" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-textarea v-decorator="['cerPrivateKey']" rows="4" placeholder="请输入证书私钥"/>
        </a-form-item>
        <a-form-item label="证书公钥" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-textarea v-decorator="['cerPublicKey']" rows="4" placeholder="请输入证书公钥"/>
        </a-form-item>
        <a-form-item label="证书密码" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'cerPassword', validatorRules.cerPassword]" placeholder="请输入证书密码"></a-input>
        </a-form-item>
        
      </a-form>
    </a-spin>
    <a-button type="primary" @click="handleOk">确定</a-button>
    <a-button type="primary" @click="handleCancel">取消</a-button>
  </a-drawer>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  
  export default {
    name: "IcbcPayInfoModal",
    components: { 
    },
    data () {
      return {
        form: this.$form.createForm(this),
        title:"操作",
        width:800,
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

        confirmLoading: false,
        validatorRules:{
        appid:{},
        apigwPublicKey:{},
        privateKey:{},
        publicKey:{},
        signType:{},
        encryptType:{},
        encryptKey:{},
        cerPrivateKey:{},
        cerPublicKey:{},
        cerPassword:{},
        },
        url: {
          add: "/icbc/icbcPayInfo/add",
          edit: "/icbc/icbcPayInfo/edit",
        }
     
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
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'appid','apigwPublicKey','privateKey','publicKey','signType','encryptType','encryptKey','cerPrivateKey','cerPublicKey','cerPassword'))
        })
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
            console.log("表单提交数据",formData)
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
      popupCallback(row){
        this.form.setFieldsValue(pick(row,'appid','apigwPublicKey','privateKey','publicKey','signType','encryptType','encryptKey','cerPrivateKey','cerPublicKey','cerPassword'))
      }
      
    }
  }
</script>

<style lang="less" scoped>
/** Button按钮间距 */
  .ant-btn {
    margin-left: 30px;
    margin-bottom: 30px;
    float: right;
  }
</style>