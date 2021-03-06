<template>
  <div>
    <el-card>
      <el-row type="flex" class="row-bg" justify="end">
        <el-col :span="2">
          <el-button
            size="mini"
            type="success"
            @click="dialogFormVisible = true">新增
          </el-button>
        </el-col>
      </el-row>

      <el-dialog title="添加账户" :visible.sync="dialogFormVisible"
                 @close="()=>{dialogFormVisible = false;form = {username: '',password: ''}}">
        <el-form :model="form">
          <el-form-item label="用户名">
            <el-input v-model="form.username" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="密码">
            <el-input v-model="form.password" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="任务">
            <el-select v-model="form.taskname" filterable placeholder="请选择">
              <el-option
                v-for="item in taskname_options"
                :key="item.value"
                :label="item.label"
                :value="item.value">
              </el-option>
            </el-select>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="()=>{dialogFormVisible = false;form = {username: '',password: '', taskname:''}}">取 消</el-button>
          <el-button type="primary" @click="handleCreate(form.username, form.password, form.taskname)">确 定</el-button>
        </div>
      </el-dialog>

      <el-table
        :data="tableData.filter(data => !search || data.username.toLowerCase().includes(search.toLowerCase()) || data.taskname.toLowerCase().includes(search.toLowerCase()) || data.tags.toLowerCase().includes(search.toLowerCase()) )"
        style="width: 100%"
        stripe
        @selection-change="handleSelectionChange">
        >
        <el-table-column
          type="selection"
          width="55">
        </el-table-column>
        <el-table-column
          label="用户名"
          prop="username">
        </el-table-column>
        <el-table-column
          label="密码"
          prop="password">
          <template slot-scope="scope">
            <el-input v-model="scope.row.password" v-if="scope.row.edit" class="edit-input" size="mini"/>
            <span v-if="!scope.row.edit">{{ scope.row.password }}</span>
          </template>
        </el-table-column>
        <el-table-column
          label="任务"
          prop="taskname">
          <template slot-scope="scope">
            <el-select v-if="scope.row.edit" v-model="scope.row.taskname" filterable placeholder="请选择">
              <el-option
                v-for="item in taskname_options"
                :key="item.value"
                :label="item.label"
                :value="item.value">
              </el-option>
            </el-select>
            <span v-if="!scope.row.edit">{{ scope.row.taskname }}</span>
          </template>
        </el-table-column>
        <el-table-column
          label="标签 (开发中)"
          prop="tags">
        </el-table-column>
        <el-table-column
          align="right">
          <template slot="header" slot-scope="scope">
            <el-input
              v-model="search"
              size="mini"
              placeholder="用户名/任务/标签"/>
          </template>

          <template slot-scope="scope">
            <el-button
              v-if="!scope.row.edit"
              size="mini"
              @click="handleStartEdit(scope.$index, scope.row)">编辑
            </el-button>
            <el-popconfirm
              v-if="!scope.row.edit"
              title="确定删除这个用户吗？"
              @onConfirm="handleDelete(scope.$index, scope.row)">
              <el-button
                v-if="!scope.row.edit"
                slot="reference"
                size="mini"
                type="danger"
                @click="preHandleDelete(scope.$index, scope.row)">删除
              </el-button>
            </el-popconfirm>
            <el-button
              v-if="scope.row.edit"
              size="mini"
              type="success"
              @click="handleSubmitEdit(scope.$index, scope.row)">确定
            </el-button>
            <el-button
              v-if="scope.row.edit"
              size="mini"
              @click="handleCancelEdit(scope.$index, scope.row)">取消
            </el-button>
          </template>

        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
import {createAccount, deleteAccount, listAccount, listTask, updateAccount} from '@/request/api'

export default {
  name: 'AccountConfig',
  data () {
    return {
      tableData: [],
      search: '',
      dialogFormVisible: false,
      form: {
        username: '',
        password: '',
        taskname: ''
      },
      taskname_options: [],
      value: ''
    }
  },
  mounted () {
    this.refreshTableData()
    this.refreshTasknameOptions()
  },
  methods: {
    handleCreate (username, password, taskname) {
      createAccount(username, password, taskname).then(res => {
        this.$notify({
          title: '创建成功',
          message: username + '/' + password,
          type: 'success'
        })
        this.dialogFormVisible = false
        this.refreshTableData()
      }).catch(err => {
        if (err.response && err.response.status) {
          this.$notify({
            title: err.response.data.msg,
            message: '',
            type: 'error'
          })
        } else {
          this.$notify({
            title: err,
            message: '',
            type: 'error'
          })
        }
      })
    },
    handleStartEdit (index, row) {
      row.edit = !row.edit
    },
    handleCancelEdit (index, row) {
      row.edit = !row.edit
      row.password = '********'
    },
    handleSubmitEdit (index, row) {
      row.edit = !row.edit

      updateAccount(row.username, row.password, row.taskname).then(res => {
        this.$notify({
          title: '修改成功',
          message: row.username + '/' + row.password + '/' + row.taskname,
          type: 'success'
        })
        this.refreshTableData()
      }).catch(err => {
        if (err.response && err.response.status) {
          this.$notify({
            title: err.response.data.msg,
            message: '',
            type: 'error'
          })
        } else {
          this.$notify({
            title: err,
            message: '',
            type: 'error'
          })
        }
      })
    },
    handleDelete (index, row) {
      deleteAccount(row.username).then(res => {
        this.$notify({
          title: '删除成功',
          message: row.username,
          type: 'success'
        })
        this.refreshTableData()
      }).catch(err => {
        if (err.response && err.response.status) {
          this.$notify({
            title: err.response.data.msg,
            message: '',
            type: 'error'
          })
        } else {
          this.$notify({
            title: err,
            message: '',
            type: 'error'
          })
        }
      })
    },
    preHandleDelete (index, row) {
    },
    handleSelectionChange (val) {
      this.multipleSelection = val
    },
    refreshTasknameOptions () {
      var _this = this
      listTask().then(res => {
        var options = []
        for (var i = 0; i < res.data.length; i++) {
          options.push({
            value: res.data[i].taskname,
            label: res.data[i].taskname
          })
        }
        _this.taskname_options = options
      }).catch(err => {
        _this.taskname_options = [{
          value: 'mana号任务',
          label: 'mana号任务'
        }, {
          value: '装备号任务',
          label: '装备号任务'
        }, {
          value: '大号任务',
          label: '大号任务'
        }]
        this.$notify({
          title: '后端未启动, 加载演示数据',
          message: err,
          type: 'success'
        })
      }

      )
    },
    refreshTableData () {
      var _this = this
      listAccount().then(res => {
        console.log(res)
        for (var i = 0; i < res.data.length; i++) {
          res.data[i]['edit'] = false
        }
        _this.tableData = res.data
      }).catch(err => {
        _this.tableData = [{
          username: '张三',
          password: '********',
          taskname: 'mana号任务',
          tags: '大号,1会',
          edit: false
        }, {
          username: '罗某',
          password: '********',
          taskname: 'mana号任务',
          tags: '大号,1会',
          edit: false
        }, {
          username: '某翔',
          password: '********',
          taskname: '装备号任务',
          tags: '农场,1会',
          edit: false
        }, {
          username: '韩立',
          password: '********',
          taskname: '装备号任务',
          tags: '农场,2会',
          edit: false
        }, {
          username: '咕噜灵波',
          password: '********',
          taskname: '大号任务',
          tags: '农场,2会',
          edit: false
        }]
        this.$notify({
          title: '后端未启动, 加载演示数据',
          message: err,
          type: 'success'
        })
      })
    }
  }
}
</script>

<style scoped>

</style>
