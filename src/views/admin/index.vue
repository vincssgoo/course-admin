<template>
  <div class="app-container">
    <div class="filter-container">
      <el-button class="filter-item"
                 type="primary"
                 icon="el-icon-edit"
                 @click="handleCreate">添加</el-button>
    </div>

    <el-table v-loading="listLoading"
              :data="list"
              element-loading-text="Loading"
              border
              fit
              highlight-current-row>
      <el-table-column align="center"
                       label="ID"
                       width="95">
        <template slot-scope="scope">
          {{ scope.row.id }}
        </template>
      </el-table-column>
      <el-table-column label="用户名"
                       min-width="110"
                       align="center">
        <template slot-scope="scope">
          {{ scope.row.username }}
        </template>
      </el-table-column>
      <el-table-column label="角色"
                       min-width="110"
                       align="center">
        <template slot-scope="scope">
          <el-tag class="m-r-5"
                  type="success"
                  v-for="role in scope.row.roles"> {{role.name}} </el-tag>
        </template>
      </el-table-column>
      <el-table-column align="center"
                       prop="created_at"
                       label="创建时间"
                       min-width="200">
        <template slot-scope="scope">
          <i class="el-icon-time" />
          <span>{{ scope.row.created_at }}</span>
        </template>
      </el-table-column>
      <el-table-column label="操作"
                       align="center"
                       width="180"
                       class-name="small-padding fixed-width">
        <template v-if="scope.row.id != 1"
                  slot-scope="scope">
          <el-row>
            <el-button type="primary"
                       size="mini"
                       @click="handleUpdate(scope.row)">编辑</el-button>
            <el-button type="danger"
                       size="mini"
                       @click="handleDel(scope.row.id)">删除</el-button>
          </el-row>
        </template>
      </el-table-column>
    </el-table>

    <div class="pagination-container">
      <el-pagination :current-page="listQuery.page"
                     :page-sizes="[10,20,30, 50]"
                     :page-size="listQuery.limit"
                     :total="total"
                     background
                     layout="total, sizes, prev, pager, next, jumper"
                     @size-change="handleSizeChange"
                     @current-change="handleCurrentChange" />
    </div>

    <el-dialog :title="textMap[dialogStatus]"
               :visible.sync="dialogFormVisible"
               :close-on-click-modal="false">
      <el-form ref="form"
               :rules="rules[dialogStatus]"
               :model="form"
               label-position="left"
               label-width="70px"
               style="width: 400px; margin-left:50px;">
        <el-form-item label="用户名"
                      prop="username">
          <el-input v-model="form.username" />
        </el-form-item>
        <el-form-item label="密码"
                      prop="password">
          <el-input type="text"
                    v-model="form.password" />
        </el-form-item>
        <el-form-item label="角色"
                      prop="roles">
          <el-checkbox :indeterminate="isIndeterminateRoles"
                       v-model="checkAllRoles"
                       @change="handleCheckAllRolesChange">全选</el-checkbox>
          <div style="margin: 15px 0;"></div>
          <el-checkbox-group v-model="form.roles"
                             @change="handleCheckedRolesChange">
            <el-checkbox v-for="role in allRoles"
                         :label="role.id"
                         :key="role.id">{{role.name}}</el-checkbox>
          </el-checkbox-group>
        </el-form-item>
        <el-form-item label="权限"
                      prop="permissions">
          <el-checkbox :indeterminate="isIndeterminatePermissions"
                       v-model="checkAllPermissions"
                       @change="handleCheckAllPermissionsChange">全选</el-checkbox>
          <div style="margin: 15px 0;"></div>
          <el-checkbox-group v-model="form.permissions"
                             @change="handleCheckedPermissionsChange">
            <el-checkbox v-for="permission in allPermissions"
                         :label="permission.id"
                         :key="permission.id">{{permission.display_name}}</el-checkbox>
          </el-checkbox-group>
        </el-form-item>
      </el-form>
      <div slot="footer"
           class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取消</el-button>
        <el-button type="primary"
                   @click="saveData"
                   :loading="btnLoading">确定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>

import request from '@/utils/request'

export default {
  filters: {
    statusFilter (status) {
      const statusMap = {
        published: 'success',
        draft: 'gray',
        deleted: 'danger'
      }
      return statusMap[status]
    }
  },
  data () {
    return {
      list: null,
      total: null,
      listLoading: true,
      btnLoading: false,
      listQuery: {
        page: 1,
        limit: 10
      },

      dialogFormVisible: false,
      dialogStatus: '',
      textMap: {
        update: '编辑',
        create: '添加'
      },
      dialogPvVisible: false,
      pvData: [],
      form: {
        id: '',
        username: '',
        password: '',
        roles: [],
        permissions: [],
      },
      rules: {
        create: {
          username: [{ required: true, message: '用户名不能为空', trigger: 'change' }],
          password: [{ required: true, message: '密码不能为空', trigger: 'change' }]
        },
        update: {
          username: [{ required: true, message: '用户名不能为空', trigger: 'change' }]
        }
      },
      //角色复选框
      checkAllRoles: false,
      isIndeterminateRoles: false,
      allRoles: [],
      //权限复选框
      checkAllPermissions: false,
      isIndeterminatePermissions: false,
      allPermissions: [],
      //按钮提交loading
      btnLoading: false,
    }
  },
  created () {
    this.getList();
    this.getRoleAll();
    this.getPermissionAll();
  },
  methods: {
    getList () {
      this.listLoading = true
      request({
        url: '/api/backend/admin/list',
        method: 'get',
        params: this.listQuery
      }).then(response => {
        this.list = response.data.data
        this.total = response.data.total
        this.listLoading = false
      })
    },
    getRoleAll () {
      request({
        url: '/api/backend/role/all',
        method: 'get',
        params: {}
      }).then(response => {
        this.allRoles = response.data
      })
    },
    getPermissionAll () {
      request({
        url: '/api/backend/permission/all',
        method: 'get',
        params: {}
      }).then(response => {
        this.allPermissions = response.data
      })
    },
    handleFilter () {
      this.listQuery.page = 1
      this.getList()
    },
    handleSizeChange (val) {
      this.listQuery.limit = val
      this.getList()
    },
    handleCurrentChange (val) {
      this.listQuery.page = val
      this.getList()
    },
    resetForm () {
      this.form = {
        id: '',
        username: '',
        password: '',
        roles: [],
        permissions: [],
      },
        this.btnLoading = false
      this.checkAllRoles = false
      this.isIndeterminateRoles = false
      this.checkAllPermissions = false
      this.isIndeterminatePermissions = false
    },

    handleCreate () {
      this.resetForm()
      this.dialogStatus = 'create'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['form'].clearValidate()
      })
    },
    handleUpdate (row) {
      this.resetForm()
      let form = Object.assign({}, row) // copy obj
      let roles = [];
      let permissions = [];
      for (let i = 0; i < form.roles.length; i++) {
        roles.push(form.roles[i].id);
      }
      for (let i = 0; i < form.permissions.length; i++) {
        permissions.push(form.permissions[i].id);
      }
      this.handleCheckedRolesChange(form.roles)
      this.handleCheckedPermissionsChange(form.permissions)
      form.permissions = permissions
      form.roles = roles
      this.form = form
      this.dialogStatus = 'update'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['form'].clearValidate()
      })
    },
    saveData () {
      this.$refs['form'].validate((valid) => {
        if (valid) {
          this.btnLoading = true
          request({
            url: '/api/backend/admin/save',
            method: 'post',
            data: this.form
          }).then(response => {
            this.btnLoading = false
            this.dialogFormVisible = false
            this.$notify({
              title: '成功',
              message: '保存成功',
              type: 'success',
              duration: 2000
            })
            this.getList();
          }).catch((err) => {
            this.btnLoading = false
          })
        }
      })
    },
    handleDel (val) {
      this.$confirm('此操作将删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        request({
          url: '/api/backend/admin/del',
          method: 'get',
          params: { id: val }
        }).then(() => {
          this.getList()
          this.$message({
            type: 'success',
            message: '删除成功!'
          });
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消'
        });
      });
    },
    handleCheckAllRolesChange (val) {
      let checkedRoles = [];
      if (val) {
        for (let i = 0; i < this.allRoles.length; i++) {
          checkedRoles.push(this.allRoles[i].id);
        }
      }
      this.form.roles = checkedRoles;
      this.isIndeterminateRoles = false;
    },
    handleCheckedRolesChange (value) {
      let checkedCount = value.length;
      this.checkAllRoles = checkedCount === this.allRoles.length;
      this.isIndeterminateRoles = checkedCount > 0 && checkedCount < this.allRoles.length;
    },
    handleCheckAllPermissionsChange (val) {
      let checkedPermissions = [];
      if (val) {
        for (let i = 0; i < this.allPermissions.length; i++) {
          checkedPermissions.push(this.allPermissions[i].id);
        }
      }
      this.form.permissions = checkedPermissions;
      this.isIndeterminatePermissions = false;
    },
    handleCheckedPermissionsChange (value) {
      let checkedCount = value.length;
      this.checkAllPermissions = checkedCount === this.allPermissions.length;
      this.isIndeterminatePermissions = checkedCount > 0 && checkedCount < this.allPermissions.length;
    },
  }
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
.user-avatar {
  width: 60px;
  height: 60px;
  border-radius: 6px;
}
</style>
