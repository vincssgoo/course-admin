<template>
  <div class="app-container">
    <div class="block"
         style="margin-bottom:15px;">

      <el-button type="primary"
                 plain
                 style="float:right;margin-right:30px"
                 icon="el-icon-search"
                 @click="handleFilter(value)">搜索</el-button>
      <el-select v-model="value"
                 placeholder="是否处理"
                 style="width:120px;margin-right:80px;float:right">
        <el-option v-for="item in options"
                   :key="item.value"
                   :label="item.label"
                   :value="item.value">
        </el-option>
      </el-select>
      <el-input placeholder="请输入分点名称"
                v-model="listQuery.keyword"
                style="width: 320px;float:right;margin-right:30px"
                clearable>
      </el-input>

    </div>
    <div style="margin-top:65px;margin-bottom:30px">
      <span> 结算时间:{{startDate}} ~ {{endDate}}</span>
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
      <el-table-column label="金额"
                       width="200"
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
                       width="160">
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
    <div style="text-align:center">
      <el-button style="margin-top:100px"
                 type="primary"
                 @click="backIndex">返 回</el-button>
    </div>
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
        {
          value: '选项3',
          label: '全部',
        },
      ],
      value: '',
      list: [],
      total: null,
      btnLoading: false,
      listLoading: true,
      listQuery: {
        page: 1,
        limit: 10,
        start_date: "",
        keyword: '',
        handle_status: '',
      },
      startDate: '',
      endDate: '',
    }
  },
  created () {
    this.startDate = this.$route.query.start_date
    this.endDate = this.$route.query.end_date
    this.listQuery.start_date = this.startDate
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
    backIndex () {
      this.$router.replace({ path: '/finance/financeStatement' })
    },
    handleFilter (value) {
      console.log(1223233);

      if (value == '选项1') {
        this.listQuery.handle_status = '2'
      }
      else if (value == '选项2') {
        this.listQuery.handle_status = '1'
      }
      else if (value == '选项3') {
        this.listQuery.handle_status = ''
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
      // this.dialogVisible = true;
      // this.form.id = row.id;
      // if (this.flag == 1) {
      //   if (row.status == '1') {
      //     row.status = '2'
      //     this.flag = 0
      //   }
      //   else if (row.status == '2') {
      //     row.status = '1'
      //     this.flag = 0
      //   }
      // }

      this.btnLoading = true;
      request({
        url: "/api/backend/finance/handle",
        method: "post",
        data: { id: row.id }
      })
        .then(() => {
          this.$message({
            type: "success",
            message: "处理成功!"
          });
          if (row.status == '1') {
            row.status = '2'
            // this.flag = 0
          }
          else if (row.status == '2') {
            row.status = '1'
            // this.flag = 0
          }
          this.getList()
        })
        .finally(() => {
          this.btnLoading = false;
        });



    },
    getList () {
      this.listLoading = true;
      console.log(123);

      request({
        url: "/api/backend/finance/settleDetail",
        method: "get",
        params: this.listQuery
      }).then(response => {
        // console.log(response);

        this.list = response.data.data;
        console.log(this.list);

        // console.log(this.list.start_date);

        this.total = response.data.total;
        this.listLoading = false;
      });

    },
    // saveData () {
    //   this.btnLoading = true;
    //   request({
    //     url: "/api/backend/finance/handle",
    //     method: "post",
    //     data: this.form
    //   })
    //     .then(() => {
    //       this.flag = 1
    //       this.$message({
    //         type: "success",
    //         message: "操作成功!"
    //       });
    //       this.dialogVisible = false;
    //       this.getList()
    //     })
    //     .finally(() => {
    //       this.btnLoading = false;
    //     });
    // }
  }
}
</script>
