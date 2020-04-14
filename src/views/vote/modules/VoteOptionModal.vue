<template>
  <a-modal
    :title="title"
    :width="800"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭"
  >
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="选项名称">
          <a-input placeholder="请输入名称" v-decorator="['name', validatorRules.name]"/>
        </a-form-item>



      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
  import pick from 'lodash.pick'
  const addVoteOption = (params)=>postAction("/serve/voteOption/add",params);
  const editVoteOption = (params)=>putAction("/serve/voteOption/edit",params);
  import { putAction,postAction} from '@/api/manage'

  export default {
    name: "VoteOptionModal",
    data() {
      return {
        title: "操作",
        visible: false,
        visibleCheck: true,
        model: {},
        voteId: "",
        status: 1,
        labelCol: {
          xs: {span: 24},
          sm: {span: 5},
        },
        wrapperCol: {
          xs: {span: 24},
          sm: {span: 16},
        },
        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules: {
          name: {rules: [{required: true, message: '请输入名称!'}]},
        },
      }
    },
    created() {
    },
    methods: {
      add(voteId) {
        this.voteId = voteId;
        this.edit({});
      },
      edit(record) {
        if (record.id) {
          this.voteId = record.voteId;
        }
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.model.voteId = this.voteId;
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model, 'name',  'sortOrder'))
        });
      },

      // 确定
      handleOk() {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            values.name = (values.name || '').trim()

            let formData = Object.assign(this.model, values);
            let obj;
            if (!this.model.id) {
              obj = addVoteOption(formData);
            } else {
              obj = editVoteOption(formData);
            }
            obj.then((res) => {
              if (res.success) {
                that.$message.success(res.message);
                that.$emit('ok');
              } else {
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
              that.close();
            })
          }
        })
      },
      // 关闭
      handleCancel() {
        this.close();
      },
      close() {
        this.$emit('close');
        this.visible = false;
      },
    }
  }
</script>