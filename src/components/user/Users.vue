<template>
    <div>
        <!-- 面包屑导航区域 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>用户管理</el-breadcrumb-item>
            <el-breadcrumb-item>用户列表</el-breadcrumb-item>
        </el-breadcrumb>

        <!-- 卡片区域 -->
        <el-card class="box-card">
            <!-- 搜索框 -->
            <el-row :gutter="20">
                <el-col :span="8" :offset="0">
                    <!-- 搜索与添加区域 -->
                    <el-input clearable placeholder="请输入内容" v-model="queryInfo.query" @clear="getUserList">
                        <el-button @click="getUserList" slot="append" icon="el-icon-search"></el-button>
                    </el-input>
                </el-col>
                <el-col :span="4" :offset="0">
                    <el-button @click="addDialogVisible = true">添加用户</el-button>
                </el-col>
            </el-row>
            <!-- 用户列表区 -->
            <el-table :data="userList" border stripe>
                <!-- 加个type="index"创建索引列 -->
                <el-table-column type="index"></el-table-column>
                <el-table-column prop="username" label="用户名" width="180"></el-table-column>
                <el-table-column prop="role_name" label="权限" width="180"></el-table-column>
                <el-table-column prop="mobile" label="电话"></el-table-column>
                <el-table-column prop="email" label="邮箱"></el-table-column>
                <el-table-column label="状态">
                    <template slot-scope="scope">
                        <!-- {{scope.row}} -->
                        <el-switch v-model="scope.row.mg_state" @change="userStateChange(scope.row)"></el-switch>
                    </template>
                </el-table-column>
                <el-table-column label="操作">
                <!-- <el-table-column label="操作" slot-scope="scope"> -->
                    <template slot-scope="scope">
                        <el-button type="primary" icon="el-icon-edit" circle @click="showEditDialog(scope.row.id)"></el-button>
                        <el-tooltip effect="dark" content="分配权限" placement="top">
                            <el-button type="warning" icon="el-icon-setting" circle></el-button>
                        </el-tooltip>
                        <el-button type="danger" icon="el-icon-delete" circle @click="removeUserByID(scope.row.id)"></el-button>
                    </template>
                </el-table-column>
            </el-table>
            <!-- 分页区域 -->
            <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="queryInfo.pagenum"
                :page-sizes="[1, 2, 5, 10]"
                :page-size="queryInfo.pagesize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total">
            </el-pagination>
        </el-card>
        <!-- 添加用户对话框 -->
        <el-dialog title="添加用户" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
          <!-- 内容主体区域 -->
          <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
            <el-form-item label="用户名" prop="username">
              <el-input v-model="addForm.username"></el-input>
            </el-form-item>
            <el-form-item label="密码" prop="password">
              <el-input v-model="addForm.password"></el-input>
            </el-form-item>
            <el-form-item label="邮箱" prop="email">
              <el-input v-model="addForm.email"></el-input>
            </el-form-item>
            <el-form-item label="手机" prop="mobile">
              <el-input v-model="addForm.mobile"></el-input>
            </el-form-item>
          </el-form>
          <!-- 底部按钮区域 -->
          <span slot="footer" class="dialog-footer">
            <el-button @click="addDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="addUser">确 定</el-button>
          </span>
        </el-dialog>

        <!-- 修改用户的对话框 -->
        <el-dialog title="修改用户" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
          <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
            <el-form-item label="用户名">
              <el-input v-model="editForm.username" disabled></el-input>
            </el-form-item>
            <el-form-item label="邮箱" prop="email">
              <el-input v-model="editForm.email"></el-input>
            </el-form-item>
            <el-form-item label="手机" prop="mobile">
              <el-input v-model="editForm.mobile"></el-input>
            </el-form-item>
          </el-form>
          <span slot="footer" class="dialog-footer">
            <el-button @click="editDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="editUserInfo">确 定</el-button>
          </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
  data () {
    // 自定义验证规则（验证邮箱和电话）
    var checkEmail = (rule, value, callback) => {
      const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
      if (regEmail.test(value)) {
        // 合法的邮箱
        return callback()
      } else {
        callback(new Error('请输入合法的邮箱'))
      }
    }
    var checkPhone = (rule, value, callback) => {
      const regPhone = /^(0|86|17951)?(13[0-9]|15[0123456789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
      if (regPhone.test(value)) {
        // 合法的手机号
        return callback()
      } else {
        callback(new Error('请输入正确的手机号'))
      }
    }
    return {
      // 获取用户列表的参数对象
      queryInfo: {
        // 搜索关键字
        query: '',
        // 当前的页数
        pagenum: 1,
        // 当前每页显示多少条数据
        pagesize: 2
      },
      userList: [],
      total: 0,
      addDialogVisible: false,
      editDialogVisible: false,
      // 保存查询到的用户信息
      editForm: {},
      // 添加用户表单数据
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      // 添加用户表单规则
      addFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入电话号码', trigger: 'blur' },
          { validator: checkPhone, trigger: 'blur' }
        ]
      },
      // 修改用户的规则
      editFormRules: {
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入电话号码', trigger: 'blur' },
          { validator: checkPhone, trigger: 'blur' }
        ]
      }
    }
  },
  created () {
    this.getUserList()
  },
  methods: {
    async getUserList () {
      const { data: res } = await this.$http.get('/users', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error('获取用户列表失败')
      this.userList = res.data.users
      this.total = res.data.total
    },
    // 监听pagesize改变的事件(n条/页)
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    // 监听页码值改变的函数(第几页)
    handleCurrentChange (newPage) {
    //   console.log(newPage)
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    // 监听switch开关状态的改变
    async userStateChange (e) {
      console.log(e)
      const { data: res } = await this.$http.put(`users/${e.id}/state/${e.mg_state}`)
      if (res.meta.status !== 200) {
        e.mg_state = !e.mg_state
        return this.$message.error('更新用户状态失败')
      }
      this.$message.success('更新状态成功')
    },
    // 对话框关闭之后的事件
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    // 点击按钮添加新用户
    addUser () {
      this.$refs.addFormRef.validate(async value => {
        if (!value) return
        // 验证通过。发起添加的网络请求
        const { data: res } = await this.$http.post('users', this.addForm)
        if (res.meta.status !== 201) return this.$message.error('添加失败')
        this.$message.success('添加用户成功')
        // 隐藏添加用户的对话框
        this.addDialogVisible = false
        // 重新获取用户列表
        this.getUserList()
      })
    },
    // 展示编辑用户的对话框
    async showEditDialog (id) {
      const { data: res } = await this.$http.get('users/' + id)
      if (res.meta.status !== 200) return this.$message.error('查询用户信息失败')
      this.editForm = res.data
      this.editDialogVisible = true
    },
    // 监听修改用户对话框的关闭事件
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    // 修改用户信息并提交
    editUserInfo () {
      this.$refs.editFormRef.validate(async value => {
        if (!value) return
        // 校验通过
        const { data: res } = await this.$http.put('users/' + this.editForm.id, {
          email: this.editForm.email,
          mobile: this.editForm.mobile
        })
        if (res.meta.status !== 200) return this.$message.error('更新失败')
        this.$message.success('更新成功')
        this.editDialogVisible = false
        this.getUserList()
      })
    },
    // 删除用户
    removeUserByID (id) {
      // 弹框询问是否删除
      this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const { data: res } = await this.$http.delete('users/' + id)
        if (res.meta.status !== 200) return this.$message.error('删除失败')
        this.$message.success('删除成功')
        this.getUserList()
      }).catch(() => {
        this.$message('已取消删除')
      })
    }
  }
}
</script>

<style lang="less" scoped>

</style>
