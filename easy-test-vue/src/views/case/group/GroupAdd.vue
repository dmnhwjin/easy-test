<template>
  <div class="container">
    <div class="title">新建分组信息</div>
    <el-row>
      <el-col :lg="16" :md="20" :sm="24" :xs="24">
        <div class="content">
          <el-form
            status-icon
            :rules="rules"
            :model="form"
            ref="form"
            label-position="right"
            label-width="100px"
            v-loading="loading"
            @submit.native.prevent
          >
            <el-form-item label="分组名称" prop="name">
              <el-input size="medium" clearable v-model="form.name"></el-input>
            </el-form-item>
            <el-form-item label="分组描述" prop="info">
              <el-input size="medium" clearable v-model="form.info"></el-input>
            </el-form-item>
            <el-form-item>
              <group-users
                @updateAuths="updateAuths"
                ref="groupUsers"
                title="分配人员"
              >
              </group-users>
            </el-form-item>
            <el-form-item class="submit">
              <el-button type="primary" @click="submitForm('form')">保 存</el-button>
              <el-button @click="resetForm('form')">重 置</el-button>
            </el-form-item>
          </el-form>
        </div>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import { post } from '@/lin/plugins/axios'
import GroupUsers from '../../../components/GroupUsers'

export default {
  components: {
    GroupUsers,
  },
  inject: ['eventBus'],
  data() {
    const checkName = (rule, value, callback) => {
      // eslint-disable-line
      if (!value) {
        return callback(new Error('分组名称不能为空'))
      }
      callback()
    }
    return {
      groupUsers: [], // 子组件传递的用户组人员被选情况 需处理
      form: {
        name: '',
        info: null,
        users: [],
      },
      rules: {
        name: [{ validator: checkName, trigger: ['blur', 'change'], required: true }],
        info: [],
      },
      loading: false,
    }
  },
  methods: {
    updateAuths(groupUsers) {
      this.groupUsers = groupUsers
    },
    // 获取所有授权用户的id放入数组
    getAuths() {
      const allAuthsUsers = []
      for (const group of this.groupUsers) {
        if (typeof group.choose !== 'undefined') {
          allAuthsUsers.push(...group.choose)
        }
      }
      this.form.users = allAuthsUsers
    },
    async submitForm(formName) {
      this.$refs[formName].validate(async valid => {
        // eslint-disable-line
        if (valid) {
          this.getAuths()
          let res
          try {
            this.loading = true
            res = await post('/v1/caseGroup', this.form, { showBackend: true })
          } catch (e) {
            this.loading = false
            console.log(e)
          }
          if (res.error_code === 0) {
            this.loading = false
            this.$message.success(`${res.msg}`)
            this.eventBus.$emit('addGroup', true)
            this.$router.push('/case/group/list')
            this.resetForm('form')
          } else {
            this.loading = false
            this.$message.error(`${res.msg}`)
          }
        } else {
          this.$message.error('请将信息填写完整')
          return false
        }
      })
    },
    resetForm(formName) {
      this.$refs[formName].resetFields()
      this.$refs.groupUsers.getGroupAuths()
    },
  },
}
</script>

<style lang="scss" scoped>
.container {
  .title {
    height: 59px;
    line-height: 59px;
    color: $parent-title-color;
    font-size: 16px;
    font-weight: 500;
    text-indent: 40px;
    border-bottom: 1px solid #dae1ec;
  }

  .content {
    margin-top: 10px;
    padding-left: 25px;
    padding-right: 40px;
  }

  .submit {
    float: left;
  }
}
</style>
