<template>
  <div class="app-container">
    <div class="block"
         style="margin-bottom:15px;">
      <span style="margin-left:40px;">时间段</span>
      <el-date-picker style="margin-left:20px;"
                      v-model="listQuery.start_datetime"
                      type="date"
                      placeholder="开始日期时间"
                      value-format="yyyy-MM-dd"></el-date-picker>
      至
      <el-date-picker v-model="listQuery.end_datetime"
                      type="date"
                      placeholder="结束日期时间"
                      value-format="yyyy-MM-dd">
      </el-date-picker>
      <el-input placeholder="请输入学校名称"
                v-model="listQuery.course_site"
                style="width: 320px;"
                clearable>
      </el-input>
      <el-select v-model="value"
                 placeholder="是否选择"
                 style="width:120px;margin-right:20px">
        <el-option v-for="item in options"
                   :key="item.value"
                   :label="item.label"
                   :value="item.value">
        </el-option>
      </el-select>
      <el-button type="primary"
                 plain
                 style="float:right;"
                 icon="el-icon-search"
                 @click="handleFilter(value)">搜索</el-button>
    </div>
    <el-table v-loading="listLoading"
              :data="list"
              element-loading-text="Loading"
              border
              fit
              highlight-current-row>
      <el-table-column align="center"
                       label="序号"
                       width="135">
        <template slot-scope="scope">
          {{ scope.$index+1 }}
        </template>
      </el-table-column>
      <el-table-column align="center"
                       label="分点名称">
        <template slot-scope="scope">
          {{ scope.row.course_site.name }}
        </template>
      </el-table-column>
      <el-table-column label="结算时间段"
                       width="250"
                       align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.start_date }} ~ {{scope.row.end_date}}</span>
        </template>
      </el-table-column>
      <el-table-column label="金额"
                       width="160"
                       align="center">
        <template slot-scope="scope">
          {{ scope.row.amount }}
        </template>
      </el-table-column>
      <el-table-column class-name="status-col"
                       label="状态"
                       width="160"
                       align="center">
        <template slot-scope="scope">
          <!-- <span>{{scope.row.status}}</span> -->
          <div v-if="scope.row.status == '1' ">
            已处理
          </div>
          <div v-else-if="scope.row.status == '2'">
            未处理
          </div>
        </template>
      </el-table-column>
      <el-table-column align="center"
                       label="操作"
                       width="200">
        <template slot-scope="scope">
          <el-button size="mini"
                     type="danger"
                     v-if="scope.row.status == '2' "
                     @click="handle(scope.row)">处理</el-button>
          <span v-else-if="scope.row.sale_status == '1' "></span>
        </template>
      </el-table-column>
    </el-table>
    <div class="block"
         style="margin-top:25px">
      <!-- <span class="demonstration">完整功能</span> -->
      <el-pagination @size-change="handleSizeChange"
                     @current-change="handleCurrentChange"
                     :current-page="listQuery.page"
                     :page-sizes="[5,10,20,30, 50]"
                     :page-size="listQuery.limit"
                     layout="total, sizes, prev, pager, next, jumper"
                     :total="total">
      </el-pagination>
    </div>
    <el-dialog :visible.sync="dialogVisible"
               :close-on-click-modal="false">
      <el-form ref="form"
               :model="form"
               label-position="left"
               label-width="130px">
        <el-form-item label="请上传凭证">
          <UploadList v-model="form.proof"></UploadList>
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
  </div>
</template>

<script>
import request from "@/utils/request";
import UploadList from '@/components/UploadList'
export default {
  components: {
    UploadList
  },
  data () {
    return {
      options: [
        {
          value: '选项1',
          label: '未处理',
        },
        {
          value: '选项2',
          label: '已处理',
        },
      ],
      dialogVisible: false,
      value: '',
      list: null,
      total: null,
      btnLoading: false,
      listLoading: true,
      listQuery: {
        page: 1,
        limit: 10,
        end_datetime: '',
        start_datetime: '',
        course_site: '',
        handle_status: '',
      },
      form: {
        id: '',
        proof: [],
      },
      flag: 0
    }
  },
  created () {
    this.getList()
  },
  watch: {
    dialogVisible (newVal, oldVal) {
      // 编辑框一异隐藏，马上清除旧数据
      if (newVal === false) {
        this.form = {
          id: "",
          proof: [],
        };
      }
    }
  },
  methods: {
    handleFilter (value) {
      console.log(1223233);

      if (value == '选项1') {
        this.listQuery.handle_status = '2'
      }
      else if (value == '选项2') {
        this.listQuery.handle_status = '1'
      }
      this.listQuery.page = 1;
      this.getList();
    },
    handleSizeChange (val) {
      this.listQuery.limit = val;
      this.getList();
    },
    handleCurrentChange (val) {
      this.listQuery.page = val;
      this.getList();
    },
    handle (row) {
      this.dialogVisible = true;
      this.form.id = row.id;
      if (this.flag == 1) {
        if (row.status == '1') {
          row.status = '2'
          this.flag = 0
        }
        else if (row.status == '2') {
          row.status = '1'
          this.flag = 0
        }
      }

    },
    getList () {
      this.listLoading = true;
      console.log(123);

      request({
        url: "/api/backend/finance/orderCount",
        method: "get",
        params: this.listQuery
      }).then(response => {
        console.log(response);

        this.list = response.data.data;
        this.total = response.data.total;
        this.listLoading = false;
      });

    },
    saveData () {
      this.btnLoading = true;
      request({
        url: "/api/backend/finance/handle",
        method: "post",
        data: this.form
      })
        .then(() => {
          this.flag = 1
          this.$message({
            type: "success",
            message: "操作成功!"
          });
          this.dialogVisible = false;
          this.getList()
        })
        .finally(() => {
          this.btnLoading = false;
        });
    }
  }
}
</script>
