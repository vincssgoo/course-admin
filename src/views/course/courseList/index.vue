<template>
  <div class="app-container">
    <div style="margin-bottom:15px;">
      <el-button type="primary"
                 @click="goCourseModified">新增课程</el-button>
      <el-button type="primary"
                 plain
                 style="float:right;"
                 icon="el-icon-search"
                 @click="handleFilter(value)">搜索</el-button>
      <el-select v-model="listQuery.type_id"
                 placeholder="科目"
                 style="float:right;width:120px;margin-right:20px"
                 clearable>
        <el-option v-for="item in list_type"
                   :label="item.name"
                   :value="item.id">
        </el-option>
      </el-select>
      <el-select v-model="listQuery.site_id"
                 placeholder="学校"
                 style="float:right;width:120px;margin-right:20px"
                 clearable>
        <el-option v-for="item in site_type"
                   :label="item.name"
                   :value="item.id">
        </el-option>
      </el-select>
      <el-select v-model="value"
                 placeholder="状态"
                 style="float:right;width:120px;margin-right:20px"
                 clearable>
        <el-option v-for="item in options"
                   :key="item.value"
                   :label="item.label"
                   :value="item.value">
        </el-option>
      </el-select>
      <el-input placeholder="请输入名称"
                v-model="listQuery.keyword"
                style="width: 320px;float:right;margin-right:20px"
                clearable>
      </el-input>

    </div>
    <el-table v-loading="listLoading"
              :data="list"
              element-loading-text="Loading"
              border
              fit
              highlight-current-row>
      <el-table-column align="center"
                       label="序号"
                       width="75">
        <template slot-scope="scope">
          {{ scope.$index+1 }}
        </template>
      </el-table-column>
      <el-table-column label="图片"
                       align="center"
                       width="110">
        <template slot-scope="scope">
          <!-- {{ scope.row.image }} -->
          <img :src="scope.row.image"
               class="user-avatar">

        </template>
      </el-table-column>
      <el-table-column align="center"
                       label="学校">
        <template slot-scope="scope">
          <div v-if="scope.row.course_site != null && scope.row.course_type != null">
            {{ scope.row.course_site.name }}
          </div>
          <div v-else>

          </div>
        </template>
      </el-table-column>
      <el-table-column align="center"
                       label="科目"
                       width="130">
        <template slot-scope="scope">
          <div v-if="scope.row.course_site != null && scope.row.course_type != null">
            {{ scope.row.course_type.name }}
          </div>
          <div v-else>

          </div>
        </template>
      </el-table-column>
      <el-table-column align="center"
                       label="名称"
                       width="150">
        <template slot-scope="scope">
          {{ scope.row.title }}
        </template>
      </el-table-column>
      <el-table-column align="center"
                       label="价格"
                       width="90">
        <template slot-scope="scope">
          {{ scope.row.price }}
        </template>
      </el-table-column>
      <el-table-column align="center"
                       label="限制人数"
                       width="85">
        <template slot-scope="scope">
          {{ scope.row.volume }}
        </template>
      </el-table-column>
      <el-table-column align="center"
                       label="已报名人数"
                       width="95">
        <template slot-scope="scope">
          {{ scope.row.enroll_amount }}
        </template>
      </el-table-column>
      <el-table-column align="center"
                       label="上课时间"
                       width="165">
        <template slot-scope="scope">
          {{ scope.row.start_time }} - {{ scope.row.end_time }}
        </template>
      </el-table-column>
      <el-table-column align="center"
                       label="上课地点"
                       width="170">
        <template slot-scope="scope">
          {{ scope.row.address_desc }}
        </template>
      </el-table-column>
      <el-table-column align="center"
                       label="联系电话"
                       width="125">
        <template slot-scope="scope">
          {{ scope.row.phone }}
        </template>
      </el-table-column>
      <el-table-column align="center"
                       label="状态"
                       width="65">
        <template slot-scope="scope">
          <!-- {{ scope.row.sale_status }} -->
          <div v-if="scope.row.sale_status == '1' ">
            正常
          </div>
          <div v-else-if="scope.row.sale_status == '2'">
            下架
          </div>
        </template>
      </el-table-column>
      <el-table-column label="操作"
                       width="220"
                       align="center">
        <template slot-scope="scope"
                  style="">
          <div>
            <el-button size="mini"
                       @click="handleEdit(scope.row)"
                       type="primary">修改</el-button>
            <el-button v-if="scope.row.sale_status == '1' "
                       size="mini"
                       @click="handleCourse(scope.row)"
                       type="danger">下架</el-button>
            <el-button v-else
                       size="mini"
                       @click="handleCourse(scope.row)"
                       type="danger">上架</el-button>
          </div>
          <div style="margin-top:8px">
            <el-button size="mini"
                       @click="goCourseTime(scope.row)"
                       type="warning"> 上课时间设置</el-button>
            <el-button size="mini"
                       @click="goSignUpDetail(scope.row)"
                       type="success">报名详情</el-button>
          </div>

        </template>
      </el-table-column>

    </el-table>
    <el-dialog :visible.sync="dialogVisible"
               :close-on-click-modal="false">

    </el-dialog>
    <!-- <router-view /> -->
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
      isSale: false,
      total: null,
      list: null,
      list_type: null,
      site_type: null,
      listLoading: true,
      btnLoading: false,
      dialogVisible: false,
      listQuery: {
        page: 1,
        limit: 5,
        keyword: '',
        sale_status: '',
        type_id: '',
        site_id: '',
      },
      options: [{
        value: '选项1',
        label: '正常'
      }, {
        value: '选项2',
        label: '下架'
      },], value: '',
      form: {
        id: '',
        title: '',
        image: '',
        price: '',
        volume: '',
        site_id: '',
        type_id: '',
        start_time: '',
        end_time: '',
        start_date: '',
        end_date: "",
        address_desc: '',
        latitude: '',
        longitude: '',
        desc: '',
        enroll_start_date: '',
        enroll_end_date: '',
        phone: '',
      }
    }
  },
  created () {
    this.getList();
    this.getTypeList();
    this.getSiteList();
  },
  // watch: {
  //   dialogVisible (newVal, oldVal) {
  //     // 编辑框一异隐藏，马上清除旧数据
  //     if (newVal === false) {
  //       this.form = {
  //         id: "",
  //         name: '',
  //       };
  //     }
  //   }
  // },
  methods: {
    handleSizeChange (val) {
      this.listQuery.limit = val;
      this.getList();
    },
    handleCurrentChange (val) {
      this.listQuery.page = val;
      this.getList();
    },
    getTypeList () {
      // console.log(123);

      this.listLoading = true;
      request({
        url: "/api/backend/courseType/index",
        method: "get",
        // params: this.listQuery
      }).then(response => {
        // console.log(234);
        this.list_type = response.data.data;
      });
    },
    getSiteList () {
      // console.log(123);

      this.listLoading = true;
      request({
        url: "/api/backend/courseSite/index",
        method: "get",
        // params: this.listQuery
      }).then(response => {
        // console.log(234);
        this.site_type = response.data.data;
      });
    },
    getList () {
      this.listLoading = true;
      request({
        url: "/api/backend/course/index",
        method: "get",
        params: this.listQuery
      }).then(response => {
        this.list = response.data.data;
        this.listLoading = false;
        this.total = response.data.total;
        this.value = ""
        console.log(this.value);
        console.log(this.list);

        // console.log(this.list);
        // this.listQuery.sale_status = ''
      });

    },
    goCourseModified () {
      this.$router.replace({ path: '/courseModified' })
    },
    goCourseTime (row) {
      this.$router.replace({
        path: '/courseTime',
        query: {
          course_id: row.id
        }
      })
    },
    goSignUpDetail (row) {
      this.$router.replace({
        path: '/signUpDetail',
        query: {
          id: row.id,
          title: row.title,
          school: row.course_site.name
        }
      })
    },
    // handleSizeChange (val) {
    //   this.listQuery.limit = val;
    //   this.getList();
    // },
    // handleCurrentChange (val) {
    //   this.listQuery.page = val;
    //   this.getList();
    // },
    changeStatus (row) {
      request({
        url: "/api/backend/course/saleStatus",
        method: "post",
        data: { id: row.id }
      }).then(response => {
        if (row.sale_status == '1') {
          row.sale_status = '2'
        }
        else if (row.sale_status == '2') {
          row.sale_status = '1'
        }
        this.getList()
      });
    },
    handleCourse (row) {

      console.log(row.id);
      this.$confirm('确定要改变课程状态吗?', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        // callback: action => {
        //   // this.$message({
        //   //   type: 'info',
        //   //   message: `action: ${ action }`
        //   // });
        //   
        // }
      }).then(() => {
        this.changeStatus(row)
      }).catch(() => {

      });


    },
    handleEdit (item) {
      // this.form = {
      //   id: item.id,
      //   name: item.name,
      //   title: item.title,
      //   image: item.image,
      //   price: item.price,
      //   volume: item.volume,
      //   site_id: item.site_id,
      //   type_id: item.type_id,
      //   start_time: item.start_time,
      //   end_time: item.end_time,
      //   start_date: item.start_date,
      //   end_date: item.end_date,
      //   address_desc: item.address_desc,
      //   latitude:item.latitude,
      //   longitude: item.longitude,
      //   desc: item.desc,
      //   enroll_start_date: item.enroll_start_date,
      //   enroll_end_date: item.enroll_end_date,
      //   phone: item.phone,
      // };
      // console.log(item);

      // this.dialogVisible = true;
      this.$router.replace({
        path: '/courseModified',
        query: { id: item.id }

      })
    },
    // handleDelete (row) {
    //   this.$confirm("确定要删除分点吗？", "提示", {
    //     confirmButtonText: "确定",
    //     cancelButtonText: "取消",
    //     type: "warning"
    //   }).then(() => {
    //     console.log(1234);
    //     request({
    //       url: "/api/backend/courseSite/delete",
    //       method: "post",
    //       data: { id: row.id },
    //     }).then(() => {
    //       console.log(123);

    //       // 删除最后一条数据时无数据问题
    //       this.list.length <= 1 ? this.listQuery.page-- : false;
    //       this.getList();
    //       this.$message({
    //         type: "success",
    //         message: "操作成功!"
    //       });
    //     });
    //   });
    // },
    handleFilter (value) {
      // console.log(value);

      if (value == '选项1') {
        this.listQuery.sale_status = '1'
      }
      else if (value == '选项2') {
        this.listQuery.sale_status = '2'
      }
      console.log(this.listQuery.sale_status)
      this.getList();
      // this.listQuery.sale_status = ''
    },
    // handleClose (done) {
    //   this.$confirm('确认关闭？')
    //     .then(_ => {
    //       done();
    //     })
    //     .catch(_ => { });
    // },


  },


}

</script>

<style scope>
.user-avatar {
  width: 60px;
  height: 60px;
  border-radius: 6px;
}
</style>