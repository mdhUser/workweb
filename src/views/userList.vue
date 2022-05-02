<template>
  <div id="div">
    <div class="content-header common-search-border1" style="margin : 10px">
      <el-button type="primary" class="butAdd" @click="handleCreate"> 新增用户
      </el-button>
    </div>

    <div class="app-container">
      <div class="box common-search-border2">
        <div class="filter-container">
          <!-- TODO: 搜索栏 -->
          <!--
              :inline="true	行内表单模式 (表单项横向排列)
              :model="queryParams"  数据绑定
              ref="queryParams" 用于找到本表单使用
          -->
          <el-form :inline="true" :model="queryParams" ref="queryParams"
                   class="formInfo">
            <el-form-item label="用户名称" label-width="80px">
              <el-input placeholder="请输入" v-model="queryParams.username"
                        style="width: 200px;" class="filter-item"></el-input>
            </el-form-item>
            <el-form-item>
              <el-button @click="handleRest">清除</el-button>
              <el-button type="primary" class="butOK" @click="handleSearch">搜索</el-button>
            </el-form-item>
          </el-form>
        </div>
      </div>
      <div class="box common-table-border" style="margin:8px 0 0;">
        <!--
            TODO: 用户列表
        -->
        <!--
            data	显示的数据
            fit	列的宽度是否自撑开
        -->
        <el-table
          :data="userList"
          fit
          style="width: 100%;">

          <el-table-column label="编号" width="50px" align="center" prop="id">
          </el-table-column>
          <el-table-column label="用户名" align="center" prop="username">
          </el-table-column>
          <el-table-column label="邮箱" align="center" prop="email">
          </el-table-column>
          <el-table-column label="创建日期" align="center" prop="createTime">
          </el-table-column>
          <el-table-column label="修改日期" align="center" prop="updateTime">
          </el-table-column>

          <el-table-column
            label="角色"
            width="120"
            align="center">
            <!--
               slot-scope = "scope"
                   用于指定当前的数据(本案例是userList中的当前行元素)
           -->
            <template slot-scope="scope">
              <!--
                  悬浮提示信息展示框
                      trigger = "hover" 鼠标移上去触发
                      placement="left" 悬浮框显示位置在左边

              -->
              <el-popover v-for="role in scope.row.roles" trigger="hover" placement="left">
                <!-- 前两个p标签是提示信息, div标签是展示信息 -->
                <p style="font-size: 10px;color: blue">角色编号: {{ role.id }}</p>
                <p style="font-size: 10px;color: blue">角色描述: {{ role.description }}</p>
                <div slot="reference" class="name-wrapper">
                  <el-tag size="medium">{{ role.name }}</el-tag>
                </div>
              </el-popover>
            </template>
          </el-table-column>
          <el-table-column label="操作" width="180">
            <template slot-scope="scope">
              <!--
                   TODO: 修改和删除
                   v-if 判断如果是admin用户就不要出现修改和删除按钮
               -->
              <div class="operation" v-if="scope.row.username != 'admin'">
                <el-button type="primary" size="mini" @click="handleUpdate(scope.row)">修改
                </el-button>
                <el-button size="mini" type="danger" @click="handleDeleted(scope.row)"> 删除
                </el-button>
              </div>
            </template>
          </el-table-column>

        </el-table>
        <!--
             TODO: 分页组件
               @size-change： 当改变每页条数时触发的函数
               @current-change：当改变页码时触发的函数
               current-page ：默认的页码
               :page-sizes：每页条数选择框中显示的值
               :page-size : 默认的每页条数
               layout： 分页组件的布局
                   total（总条数）, sizes(每页条数), prev（上一页）, pager(所有的页码), next(下一页), jumper（跳转页码）
               :total: 总条数
         -->
        <div class="pagination-container">
          <el-pagination
            class="pagiantion"
            v-show="pagination.total>0"
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="pagination.pageNum"
            :page-sizes="[3, 5, 10, 15]"
            :page-size="pagination.pageSize"
            layout="total, sizes, prev, pager, next, jumper"
            :total="pagination.total">
          </el-pagination>
        </div>
      </div>
      <!--TODO: 新增用户弹层
             :visible.sync 是否显示 (dialogFormVisible=true 显示, =false隐藏)
      -->
      <el-dialog title="新增用户" align="left" :visible.sync="dialogFormVisible" width="40%" top="8vh">
        <!--
            model	表单数据对象
            rules	表单验证规则
        -->
        <el-form label-width="120px" :model="addUser" :rules="rules" ref="form">
          <!--
          prop	表单域 model 字段，在使用 validate、resetFields 方法的情况下，该属性是必填的
          -->
          <el-form-item label="用户名称：" prop="username">
            <!-- v-model 双向数据绑定  autocomplete=off 取消自动补全功能-->
            <el-input v-model="addUser.username" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="邮箱：" prop="email">
            <el-input v-model="addUser.email" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="密码：" prop="password">
            <el-input type="password" v-model="addUser.password" autocomplete="off"></el-input>
          </el-form-item>
          <!--
              TODO: 动态获取角色
          -->
          <el-form-item label="角色: " prop="roleIds">
            <!--
             value / v-model	绑定值 (双向数据绑定)
                multiple	是否多选
                注意: 这是多选组件, addUser.roleIds是数组!!!
            -->
            <el-select v-model="addUser.roleIds" multiple placeholder="请选择至少一个角色">
              <!--
                  v-for : 遍历, roleList是被遍历的数组
                  value	选项的值	string/number/object
                  label	选项的标签，若不设置则默认与 value 相同 (显示出来的标签名)
                  key     文档中没有说明, 就是取值跟value相同,删除也不影响本组件使用
              -->
              <el-option
                v-for="role in roleList"
                :key="role.id"
                :label="role.name"
                :value="role.id">
              </el-option>
            </el-select>
          </el-form-item>

          <el-form-item label="备注：" prop="remark">
            <el-input v-model="addUser.remark" autocomplete="off"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="handleCreateCancel">取 消</el-button>
          <el-button type="primary" @click="handleCreateConfirm">确 定</el-button>
        </div>
      </el-dialog>
      <!-- end -->
      <!-- 修改用户弹层 -->
      <el-dialog title="修改用户" align="left" :visible.sync="dialogUptFormVisible" width="40%" top="8vh">
        <el-form label-width="120px" :model="updateUser" :rules="rules" ref="uptForm">
          <el-form-item label="用户名称：" prop="username">
            <el-input v-model="updateUser.username" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="邮箱：" prop="email">
            <el-input v-model="updateUser.email" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="密码：" prop="password">
            <el-input type="password" v-model="updateUser.password" autocomplete="off"></el-input>
          </el-form-item>

          <el-form-item label="角色: " prop="roleIds">
            <el-select v-model="updateUser.roleIds" multiple placeholder="请选择至少一个角色">
              <el-option
                v-for="role in roleList"
                :key="role.id"
                :label="role.name"
                :value="role.id">
              </el-option>
            </el-select>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="handleUpdateCancel">取 消</el-button>
          <el-button type="primary" @click="handleUpdateConfirm">确 定</el-button>
        </div>
      </el-dialog>
      <!-- end -->
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      pagination: {
        total: 0,  //总条数
        pageNum: 1, // //当前页
        pageSize: 5 //每页显示条数
      },
      userList: [],  //用户列表数据
      roleList: [], // 角色列表数据
      queryParams: { // 搜索条件
        username: ''
      },
      dialogFormVisible: false, // 添加窗口显示状态
      dialogUptFormVisible: false,//修改窗口显示状态
      addUser: { // 用户数据
        username: '',
        email: '',
        password: '',
        remark: '',
        roleIds: []
      },
      updateUser: { //用户的修改数据
        id: "",
        username: '',
        email: '',
        password: '',
        roleIds: []
      },
      rules: { //校验规则
        username: [
          {required: true, message: '请填写', trigger: 'blur'}
        ],
        email: [
          {required: true, message: '请填写', trigger: 'blur'},
          {type: 'email', message: '请输入正确的邮箱地址', trigger: ['blur', 'change']}
        ],
        password: [
          {required: true, message: '请填写', trigger: 'blur'}
        ]
      }

    }
  },
  created() {
    //分页查询用户列表
    this.getUserList()
    //查询所有的角色 TODO ?
    //  (添加用户和修改用户的时候要展示所有角色以供选择)
    this.getRoleList()
  },
  methods: {
    //TODO: 获取用户分页数据
    getUserList() {
      // TODO: 以下是伪数据(用作参考,真实数据需要从服务器获取)
      let url = `/user/userList/${this.pagination.pageSize}/${this.pagination.pageNum}/${this.queryParams.username === '' ? "''" : this.queryParams.username}`;
      axios.get(url).then(response => {

        // 指定总记录数
        this.pagination.total = response.data.data.totalRecords;

        // 执行分页显示的数据
        this.userList = response.data.data.list;

      });

    },
    //TODO: 查询角色表
    getRoleList() {

      let url = "/role/roleList"
      axios.get(url).then(response => this.roleList = response.data);

    },
    //将查询条件置空
    handleRest() {
      this.queryParams = {
        username: ''
      }
    },
    //搜索
    handleSearch() {
      //将当前页码设置为1
      this.pagination.pageNum = 1;
      this.getUserList();
    },
    //每页数量改变的时候
    handleSizeChange(val) {
      this.pagination.pageSize = val;
      this.pagination.pageNum = 1;
      this.getUserList();
    },
    //当前页码改变的时候
    handleCurrentChange(val) {
      this.pagination.pageNum = val;
      this.getUserList();
    },
    // 新增用户
    handleCreate() {
      if (this.$refs['form']) {
        this.$refs['form'].resetFields();
      }
      this.dialogFormVisible = true;

    },
    //新增用户确定 TODO:
    handleCreateConfirm() {
      this.$refs['form'].validate((valid) => {
        if (valid) {
          let url = "/user/addUser"
          let params = this.addUser
          axios.post(url, params).then(response => {
            console.log(response.data);
            if (response.data.status === 200) {
              this.dialogFormVisible = false;
              this.$message.success(response.data.message)
            } else {
              this.$message.error(response.data.message)
            }
          }).finally(() => {
            this.getUserList()
          })
        }
      });
    },
    //新增用户取消
    handleCreateCancel() {
      this.dialogFormVisible = false;
    },
    //修改用户 数据回显
    handleUpdate(row) {
      var t = this;
      if (t.$refs['uptForm']) {
        t.$refs['uptForm'].resetFields();
      }
      t.updateUser.id = row.id;
      t.updateUser.username = row.username;
      t.updateUser.password = row.password;
      t.updateUser.email = row.email;

      //遍历取出此用户所有的角色id
      let roleIds = [];
      for (let i = 0; i < row.roles.length; i++) {
        roleIds[i] = row.roles[i].id;
      }
      t.updateUser.roleIds = roleIds;

      t.dialogUptFormVisible = true;

    },
    //修改用户确定 TODO
    handleUpdateConfirm() {
      this.$refs['uptForm'].validate((valid) => {
        if (valid) {
          let url = "/user/updateUser"
          let params = this.updateUser
          axios.post(url, params).then(response => {
            if (response.data.status === 200) {
              this.dialogFormVisible = false;
              this.$message.success(response.data.message)
            } else {
              this.$message.error(response.data.message)
            }
          }).finally(() => {
            this.getUserList()
          })
        }
      });
    },
    //修改用户取消
    handleUpdateCancel() {
      this.dialogUptFormVisible = false;
    },
    // 删除 TODO:
    handleDeleted(row) {
      //点击确认走then,点击取消走catch
      this.$confirm('此操作将永久删除用户 ' + ', 是否继续?', '提示', {
        type: 'warning'
      }).then(() => {
        let url = "/user/deleteUser"
        let param = `uid=${row.id}`
        axios.get(url + "?" + param).then(response => {
          if (response.data.status === 200) {
            this.$message.success(response.data.message)
          } else {
            this.$message.error(response.data.message)
          }
        }).finally(() => {
          this.getUserList()
        })
      }).catch(() => {
        this.$message.info('已取消操作!')
      });
    }
  }
}
</script>
<style>
</style>
