<template>
  <a-card :bordered="false">
    <a-form :form="form" labelAlign="left">
      <h3>微信支付参数设置</h3>
      <a-form-item  labelAlign="left" :labelCol="labelCol" :wrapperCol="wrapperCol" label="APPID">
        <a-input placeholder="请输入APPID" v-decorator="['appId', {}]" />
      </a-form-item>
      <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="商户号">
        <a-input placeholder="请输入商户号" v-decorator="['merchantId', {}]" />
      </a-form-item>
      <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="支付密钥">
        <a-input placeholder="请输入支付密钥" v-decorator="['secert', {}]" />
      </a-form-item>
      <h3>支付宝参数设置</h3>
      <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="APPID">
        <a-input placeholder="请输入APPID" v-decorator="['appId', {}]" />
      </a-form-item>
      <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="商户号">
        <a-input placeholder="请输入商户号" v-decorator="['merchantId', {}]" />
      </a-form-item>
      <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="支付密钥">
        <a-input placeholder="请输入支付密钥" v-decorator="['secert', {}]" />
      </a-form-item>
    </a-form>
    <div style='text-align:center'>
    <a-button type="primary"  @click="save" >保存</a-button>

    </div>
  </a-card>
</template>

<script>
export default {
  data() {
    return {
      form: this.$form.createForm(this),
      labelCol: {
        xs: { span: 2 },
        sm: { span: 2 }
      },
      wrapperCol: {
        xs: { span: 6 },
        sm: { span: 6 }
      }
    };
  },
  created() {},
  methods: {
      save(){

      }
  }
};
</script>

<style scoped lang="less">
</style>
