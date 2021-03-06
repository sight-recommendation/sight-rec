<template>
  <div class="settings">
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: 'user/settings' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>基本资料</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <p>
        <i class="el-icon-user"></i>
        <span> 基本资料</span>
      </p><br />
      <el-form style="width: 95%;"
               :model="userForm"
               ref="userFormRef"
               :rules="userFormRules"
               label-width="100px">
        <el-row>
          <el-col :span="12">
            <!-- prop 为 userFormRules 中的验证规则 -->
            <el-form-item label="用户名"
                          prop="name">
              <el-input maxlength="12"
                        show-word-limit
                        v-model="userForm.name"
                        clearable></el-input>
              <span style="font-size: small;color: #909399;">数字 ID：{{ userForm.id }}</span>
            </el-form-item>
            <el-form-item label="头像">
              <el-tooltip class="item"
                          effect="dark"
                          content="点击图片更换头像"
                          placement="left">
                <el-upload class="avatar-uploader"
                           action="https://jsonplaceholder.typicode.com/posts/"
                           :show-file-list="false"
                           :on-success="handleAvatarSuccess"
                           :before-upload="beforeAvatarUpload">
                  <img v-if="userForm.headUrl"
                       :src="userForm.headUrl"
                       class="avatar">
                  <i v-else
                     class="el-icon-plus avatar-uploader-icon"></i>
                </el-upload>
              </el-tooltip>
              <span style="font-size: small;color: #909399;">头像修改自动保存</span>
            </el-form-item>
            <el-form-item label="邮箱">
              <el-input placeholder="请输入邮箱"
                        v-model="userForm.email"
                        clearable></el-input>
            </el-form-item>
            <el-form-item>
              <el-button type="primary"
                         @click="editUserInfo">保存修改</el-button>
              <el-button @click="resetForm('userFormRef')">重 置</el-button>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="手机"
                          prop="phone">
              <el-input placeholder="请输入手机号"
                        type="number"
                        v-model="userForm.phone"
                        clearable></el-input>
            </el-form-item>
            <el-form-item label="修改密码"
                          prop="password">
              <el-input placeholder="请输入新密码"
                        v-model="userForm.password"
                        show-password
                        clearable></el-input>
            </el-form-item>
            <el-form-item label="确认密码"
                          prop="checkPassword">
              <el-input placeholder="确认密码"
                        v-model="userForm.checkPassword"
                        show-password
                        clearable></el-input>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    var validatePass = (rule, value, callback) => {
      if (value !== this.userForm.password) {
        callback(new Error('两次输入密码不一致!'))
      } else {
        callback()
      }
    }
    return {
      userName: window.sessionStorage.getItem('name'),
      userForm: {
        id: window.sessionStorage.getItem('id'),
        name: window.sessionStorage.getItem('name'),
        headUrl: '',
        email: '',
        phone: '',
        password: '',
        checkPassword: ''
      },
      userFormRules: {
        name: [
          { required: true, message: '请输入用户名', trigger: 'blur' }
        ],
        password: [
          { required: false, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
        ],
        checkPassword: [
          { required: false, validator: validatePass, trigger: 'blur' }
        ]
      }
    }
  },
  created () {
    this.getUserInfo()
  },
  methods: {
    async getUserInfo () {
      const { data: res } = await this.$http.get('users', {
        params: { id: this.userForm.id }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取用户信息失败！')
      }
      this.userForm.email = res.data.email
      this.userForm.phone = res.data.phone
      this.userForm.headUrl = res.data.headUrl
    },
    // 修改用户信息并提交
    editUserInfo () {
      this.$refs.userFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put('users/', this.userForm)
        if (res.meta.status !== 200) {
          return this.$message.error(res.meta.msg)
        }
        console.log(res)
        // 提示修改成功
        this.$message.success('修改成功！')
      })
    },
    handleAvatarSuccess (res, file) {
      this.imageUrl = URL.createObjectURL(file.raw)
    },
    beforeAvatarUpload (file) {
      const isJPG = file.type === 'image/jpeg'
      const isLt2M = file.size / 1024 / 1024 < 2

      if (!isJPG) {
        this.$message.error('上传头像图片只能是 JPG 格式!')
      }
      if (!isLt2M) {
        this.$message.error('上传头像图片大小不能超过 2MB!')
      }
      return isJPG && isLt2M
    },
    resetForm (ruleForm) {
      this.$refs[ruleForm].resetFields()
    }
  }
}
</script>

<style lang="less">
.col-left > p {
  color: #909399;
  text-align: right;
  line-height: 40px;
}

.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409eff;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 89px;
  height: 89px;
  line-height: 89px !important;
  text-align: center;
}
.avatar {
  width: 89px;
  height: 89px;
  display: block;
}
</style>
