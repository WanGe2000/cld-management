<template>
  <a-drawer
    :title="title"
    :width="720"
    @close="close"
    :visible="visible"
    :bodyStyle="{paddingBottom: '80px'}"
  >

    <a-tree
      checkable
      @expand="onExpand"
      v-model="checkedKeys"
      :treeData="userTreeList"
    />

    <div
      :style="{
          position: 'absolute',
          right: 0,
          bottom: 0,
          width: '100%',
          borderTop: '1px solid #e9e9e9',
          padding: '10px 16px',
          background: '#fff',
          textAlign: 'right',
          zIndex: 1,
        }"
    >
      <a-button :style="{marginRight: '8px'}" @click="close"> 取消</a-button>
      <a-button @click="handleOk" type="primary" >提交</a-button>
    </div>
  </a-drawer>
</template>

<script>
  import { getAction,postAction } from '@/api/manage'
  import ATextarea from 'ant-design-vue/es/input/TextArea'
  import JSearchSelectTag from '@/components/dict/JSearchSelectTag'

  export default {
    name: 'SysDepartModal',
    components: { ATextarea, JSearchSelectTag },
    data() {
      return {
        width: 800,
        visible: false,
        title: '操作',
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 }
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 }
        },
        confirmLoading: false,
        url: {
          userTree: '/sys/user/treeDepartUser',
          allUserTree: '/sys/user/treeUser',
          editUser: '/sys/user/editSysDepartWithUser',
          saveUser: '/sys/user/saveDepartUser'
        },
        checkedKeys: [],
        userTreeList: null,
        departId: null
      }
    },
    created() {
    },
    watch: {
      checkedKeys(val) {
        console.log('onCheck', val)
      }
    },
    methods: {
      loadTree(id,type) {
        this.departId = id;
        getAction(this.url.allUserTree, {id})
          .then(res => {
            this.userTreeList = res.result
            getAction(this.url.userTree, { departId: id })
              .then(result => {
                if (result.success) {
                  if(result.result!=null){
                    result.result.forEach(data => {
                      this.checkedKeys.push(data.value)
                    })
                  }
                  this.visible = true
                }
              })
          })
      },
      close() {
        this.$emit('close');
        this.visible = false;
      },
      onExpand(expandedKeys) {
        console.log('onExpand', expandedKeys)
        // if not set autoExpandParent to false, if children expanded, parent can not collapse.
        // or, you can remove all expanded children keys.
        this.expandedKeys = expandedKeys
        this.autoExpandParent = false
      },
      onCheck(checkedKeys) {
        console.log('onCheck', checkedKeys)
        this.checkedKeys = checkedKeys
      },
      handleOk() {
        postAction(this.url.saveUser,{userIds:this.checkedKeys,departId:this.departId})
          .then(res => {
            if(res.success){
              this.$message.success("操作成功");
              this.close()
            }
          })
      }
    }
  }
</script>

<style scoped>

</style>