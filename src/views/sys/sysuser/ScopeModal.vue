<template>
  <a-modal
    title="授权"
    :visible="visible"
    @ok="handleOk"
    @cancel="handleClose"
    :confirmLoading="confirmLoading"
  >
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
        <a-form-item
          label="用户名"
          :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input
            v-model="username"
            disabled
            placeholder="用户名"/>
        </a-form-item>
        <a-form-item label="角色"
                     :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-select mode="multiple"
                    style="width: 100%"
                    placeholder="无角色"
                    :allowClear="true"
                    v-model="roleIds">
            <a-select-option v-for="selectData in roleSelectData"
                             :key="selectData.value">
              {{ selectData.name }}
            </a-select-option>
          </a-select>
        </a-form-item>
      </a-form>
    </a-spin>
  </a-modal>
</template>
<script>
import { getUserScope, putUserScope } from '@/api/sys/sysuser'
import { getSelectData } from '@/api/sys/role'

export default {
  name: 'ScopeModal',
  data () {
    return {
      visible: false,
      confirmLoading: true,
      form: this.$form.createForm(this),
      labelCol: {
        xs: { span: 24 },
        sm: { span: 5 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 16 }
      },

      userId: '',
      username: '',
      roleIds: [],

      roleSelectData: []
    }
  },
  created () {
    getSelectData({}).then(res => {
      this.roleSelectData = res.data
    })
  },
  methods: {
    show (record) {
      this.visible = true
      this.confirmLoading = true

      this.userId = record.userId
      this.username = record.username
      this.roleIds = []

      getUserScope(record.userId)
        .then(res => {
          if (res.data.roleIds) {
            this.roleIds = res.data.roleIds.map(String)
            this.confirmLoading = false
          }
        })
    },
    handleOk (e) {
      this.confirmLoading = true
      const userScope = {
        roleIds: this.roleIds
      }
      putUserScope(this.userId, userScope).then(res => {
        if (res.code === 200) {
          this.$message.success(res.msg)
          this.handleClose()
        } else {
          this.$message.warning(res.msg)
        }
      })
        .finally(() => {
          this.confirmLoading = false
        })
    },
    handleClose (e) {
      this.visible = false
      this.confirmLoading = false
    }
  }
}
</script>