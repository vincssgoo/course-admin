<template>
  <div class="app-container">
    <div style="margin-bottom:15px;">
      <el-button type="primary"
                 @click="dialogVisible = true">新增</el-button>
      <el-button type="primary"
                 plain
                 style="float:right;"
                 icon="el-icon-search"
                 @click="handleFilter(fck)">搜索</el-button>
      <el-input placeholder="请输入学校名称"
                v-model="listQuery.name"
                style="width: 320px;float:right;"
                clearable>
      </el-input>
      <el-select v-model="fck"
                 placeholder="类型"
                 style="float:right;width:120px;margin-right:20px"
                 clearable>
        <el-option v-for="item in options"
                   :key="item.value"
                   :label="item.label"
                   :value="item.value">
        </el-option>
      </el-select>

    </div>
    <el-table v-loading="listLoading"
              :data="list"
              element-loading-text="Loading"
              border
              fit
              highlight-current-row>
      <el-table-column align="center"
                       label="序号"
                       width="155">
        <template slot-scope="scope">
          {{ scope.$index+1 }}
        </template>
      </el-table-column>
      <el-table-column label="图片"
                       align="center">
        <template slot-scope="scope">
          <img :src="scope.row.image"
               class="fuk">
        </template>
      </el-table-column>
      <el-table-column align="center"
                       label="分点类型"
                       width="255">
        <template slot-scope="scope">
          <div v-if="scope.row.type == '1'">
            <span>中心校</span>
          </div>
          <div v-else-if="scope.row.type == '2'">
            <span>基地校</span>
          </div>
        </template>
      </el-table-column>
      <el-table-column label="分点名称"
                       align="center">
        <template slot-scope="scope">
          {{ scope.row.name }}
        </template>
      </el-table-column>
      <el-table-column label="操作"
                       width="230"
                       align="center">
        <template slot-scope="scope">
          <el-button type="primary"
                     @click="handleEdit(scope.row)">修改</el-button>
          <el-button type="danger"
                     @click="handleDelete(scope.row)">删除</el-button>
        </template>
      </el-table-column>

    </el-table>

    <el-dialog :close-on-click-modal="false"
               :visible.sync="dialogVisible"
               width="50%">
      <el-form ref="form"
               :model="form"
               label-position="left"
               label-width="70px">
        <el-form-item label="学校分点"
                      prop="name">
          <el-input v-model="form.name"
                    placeholder="请输入学校分点名称"
                    style="width:50%" />
        </el-form-item>
        <el-form-item label="分点类型"
                      prop="name">
          <el-select v-model="fck"
                     placeholder="请选择">
            <el-option v-for="item in options"
                       :key="item.value"
                       :label="item.label"
                       :value="item.value">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item>
          <div class="fck">
            <div class="u">
              <span>图片</span>
              <span>(726 x 426)</span>
            </div>

            <upload-one v-model="form.image"
                        style="margin-left:5px"></upload-one>

          </div>

        </el-form-item>

      </el-form>
      <div slot="footer"
           class="dialog-footer">
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary"
                   @click="saveData"
                   :loading="btnLoading">确定</el-button>
      </div>
    </el-dialog>

    <div class="block"
         style="margin-top:25px">
      <!-- <span class="demonstration">完整功能</span> -->
      <el-pagination @size-change="handleSizeChange"
                     @current-change="handleCurrentChange"
                     :current-page="listQuery.page"
                     :page-sizes="[5, 10, 20, 50,100]"
                     :page-size="listQuery.limit"
                     layout="total, sizes, prev, pager, next, jumper"
                     :total="total">
      </el-pagination>
    </div>
  </div>
</template>

<script>
import request from "@/utils/request";

export default {
  data () {
    return {
      total: null,
      list: null,
      listLoading: true,
      btnLoading: false,
      dialogVisible: false,
      listQuery: {
        page: 1,
        limit: 10,
        name: "",
        type: '',
      },
      form: {
        id: "",
        name: "",
        type: "",
        image: '',
      },
      options: [{
        value: '选项1',
        label: '中心校'
      }, {
        value: '选项2',
        label: '基地校'
      },], value: '',
      fck: '',
    }
  },
  created () {
    this.getList();
  },
  watch: {
    dialogVisible (newVal, oldVal) {
      // 编辑框一异隐藏，马上清除旧数据
      if (newVal === false) {
        this.form = {
          id: "",
          name: '',
          type: '',
          image: '',
        };
      }
    }
  },
  methods: {
    getList () {
      console.log(123);

      this.listLoading = true;
      request({
        url: "/api/backend/courseSite/index",
        method: "get",
        params: this.listQuery
      }).then(response => {
        this.list = response.data.data;
        this.total = response.data.total;
        this.listLoading = false;
        console.log(this.list);

      });
    },
    handleSizeChange (val) {
      this.listQuery.limit = val;
      this.getList();
    },
    handleCurrentChange (val) {
      this.listQuery.page = val;
      this.getList();
    },
    handleEdit (item) {

      this.form = {
        id: item.id,
        name: item.name,
        type: item.type,
        image: item.image,
      };

      if (this.form.type == '1') {
        this.fck = '中心校'
      }
      else if (this.form.type == '2') {
        this.fck = '基地校'
      }

      this.dialogVisible = true;
    },
    handleDelete (row) {
      this.$confirm("确定要删除分点吗？", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      }).then(() => {
        console.log(1234);
        request({
          url: "/api/backend/courseSite/delete",
          method: "post",
          data: { id: row.id },
        }).then(() => {
          console.log(123);

          // 删除最后一条数据时无数据问题
          this.list.length <= 1 ? this.listQuery.page-- : false;
          this.getList();
          this.$message({
            type: "success",
            message: "操作成功!"
          });
        });
      });
    },
    handleFilter (value) {
      if (value == '选项1') {
        this.listQuery.type = '1'
      }
      else if (value == '选项2') {
        this.listQuery.type = '2'
      }
      this.listQuery.page = 1;
      this.getList();
    },
    // handleClose (done) {
    //   this.$confirm('确认关闭？')
    //     .then(_ => {
    //       done();
    //     })
    //     .catch(_ => { });
    // },

    saveData () {
      console.log(this.fck);

      if (this.fck == '选项1') {
        this.form.type = '1'
      }
      else if (this.fck == '选项2') {
        this.form.type = '2'
      }
      console.log(this.form.type);

      if (!this.form.name) {
        this.$message({
          type: "warning",
          message: "请输入学校分点名称"
        });
        return;
      }

      this.btnLoading = true;
      request({
        url: "/api/backend/courseSite/store",
        method: "post",
        data: this.form
      })
        .then(() => {
          this.btnLoading = false;
          this.dialogVisible = false;
          this.getList();
          this.$message({
            type: "success",
            message: "操作成功!"
          });
        })
        .finally(() => {
          this.btnLoading = false;
        });
    },
  },


}

</script>
<style>
.fck {
  display: flex;
  flex-direction: row;
  margin-left: -70px;
}
.u {
  display: flex;
  flex-direction: column;
}
.fuk {
  width: 90px;
  height: 90px;
}
</style>