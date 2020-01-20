<template>
  <div class="app-container">

    <div class="block"
         style="margin-bottom:85px;">
      <el-button style="float:left"
                 type="primary"
                 @click="dialogVisible = true">新建结算</el-button>
      <div style="float:right;">
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
        <el-button type="primary"
                   plain
                   style="margin-left:30px"
                   icon="el-icon-search"
                   @click="handleFilter(value)">搜索</el-button>
      </div>

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
      <el-table-column align="center"
                       label="结算时间开始"
                       width="280">

        <template slot-scope="scope">
          {{ scope.row.start_date }}
        </template>
      </el-table-column>
      <el-table-column label="结算时间结束"
                       width="280"
                       align="center">
        <template slot-scope="scope">
          {{scope.row.end_date}}
        </template>
      </el-table-column>
      <el-table-column label="总金额"
                       width="160"
                       align="center">
        <template slot-scope="scope">
          {{ scope.row.amount }}
        </template>
      </el-table-column>
      <el-table-column class-name="status-col"
                       label="操作时间"
                       align="center">
        <template slot-scope="scope">
          {{scope.row.handle_datetime}}

        </template>
      </el-table-column>
      <el-table-column align="center"
                       label="操作"
                       width="200">
        <template slot-scope="scope">
          <el-button type="primary"
                     @click="goDetail(scope.row)">结算明细</el-button>
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
               :close-on-click-modal="false"
               class="dialog">
      <div class="first">
        <span>上次结算至:{{last}}</span>
        <div class="last"><span>本次结算日期从 {{now}} 至:</span></div>

        <el-form ref="form"
                 :model="form"
                 label-position="left"
                 label-width="70px">
          <el-form-item label="日期"
                        prop="name">
            <el-date-picker v-model="form.end_date"
                            type="date"
                            placeholder="选择日期"
                            value-format="yyyy-MM-dd">
            </el-date-picker>
          </el-form-item>
        </el-form>

      </div>
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
      },
      form: {
        start_date: '',
        end_date: '',
      },
      flag: 0,
      value1: '',
      time: null,
      last: '',
      now: '',
    }
  },
  created () {
    this.getList()
    this.getTime()
  },
  watch: {
    dialogVisible (newVal, oldVal) {
      // 编辑框一异隐藏，马上清除旧数据
      if (newVal === false) {
        this.form = {
          start_date: '',
          end_date: '',
        };
      }
    }
  },
  methods: {
    handleFilter () {

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
    goDetail (row) {
      this.$router.replace({        path: '/detail', query: {
          start_date: row.start_date,
          end_date: row.end_date
        }      })
    },
    getTime () {
      this.listLoading = true;
      request({
        url: "/api/backend/finance/createSettleDetail",
        method: "get",
      }).then(response => {
        this.time = response.data;
        this.last = this.time.last_end_date
        this.now = this.time.now_start_date
        this.listLoading = false;
      });
    },
    getList () {
      this.listLoading = true;

      request({
        url: "/api/backend/finance/orderSettle",
        method: "get",
        params: this.listQuery
      }).then(response => {
        this.list = response.data.data;
        console.log(this.list);
        this.total = response.data.total;
        this.listLoading = false;
      });

    },
    saveData () {
      this.form.start_date = this.time.now_start_date
      this.btnLoading = true;
      request({
        url: "/api/backend/finance/createOrderSettle",
        method: "post",
        data: this.form
      })
        .then(() => {
          this.$message({
            type: "success",
            message: "操作成功!"
          });
          this.dialogVisible = false;
          this.getList()
          this.getTime()
        })
        .finally(() => {
          this.btnLoading = false;
        });
    }
  }
}
</script>
<style>
.first {
  /* margin-top: 20px; */
}
.last {
  margin-top: 30px;
  margin-bottom: 30px;
}
.dialog {
  margin-left: 250px;
  /* position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  margin: 0; */
  width: 70%;
}
</style>