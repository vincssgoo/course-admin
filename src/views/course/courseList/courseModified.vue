<template>
  <div class="app-container">

    <el-form ref="form"
             :model="form"
             label-width="120px">
      <el-form-item label="图片">

        <upload-one v-model="form.image"></upload-one>
      </el-form-item>
      <el-form-item label="名称">
        <el-input v-model="form.title"
                  placeholder="请输入名称"
                  style="width:30%"
                  clearable />
      </el-form-item>
      <el-form-item label="价格">
        <el-input v-model="form.price"
                  placeholder="请输入价格"
                  style="width:30%"
                  clearable />
      </el-form-item>
      <el-form-item label="学校">
        <el-select v-model="form.site_id"
                   placeholder="请选择">
          <el-option v-for="item in list_location"
                     :label="item.name"
                     :value="item.id">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="科目">
        <el-select v-model="form.type_id"
                   placeholder="请选择">
          <el-option v-for="item in list_type"
                     :label="item.name"
                     :value="item.id">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="限制人数">
        <el-input v-model="form.volume"
                  placeholder="请输入限制人数"
                  style="width:30%"
                  clearable />
      </el-form-item>
      <el-form-item label="课程周期">
        <el-date-picker v-model="form.start_date"
                        type="date"
                        placeholder="开始日期时间"
                        value-format="yyyy-MM-dd"></el-date-picker>
        至
        <el-date-picker v-model="form.end_date"
                        type="date"
                        placeholder="结束日期时间"
                        value-format="yyyy-MM-dd">
        </el-date-picker>
      </el-form-item>
      <!-- <el-form-item label="上课时间点">
        <el-time-picker v-model="form.start_time"
                        :picker-options="{
      selectableRange: '00:00:00 - 23:59:59'
    }"
                        placeholder="上课时间点">
        </el-time-picker>
        至
        <el-time-picker v-model="form.end_time"
                        :picker-options="{
      selectableRange: '00:00:00 - 23:59:59'
    }"
                        placeholder="下课时间点">
        </el-time-picker>
      </el-form-item> -->
      <el-form-item label="上课开始时间:">
        <el-time-select v-model="form.start_time"
                        :picker-options="{
start: '00:00',
step: '00:30',
end: '23:30'
}"
                        placeholder="任意时间点"></el-time-select>
      </el-form-item>
      <el-form-item label="上课结束时间:">
        <el-time-select v-model="form.end_time"
                        :picker-options="{
start: '00:00',
step: '00:30',
end: '23:30',
minTime: form.start_time
}"
                        placeholder="任意时间点"></el-time-select>
      </el-form-item>
      <el-form-item label="报名周期">
        <el-date-picker v-model="form.enroll_start_date"
                        type="date"
                        placeholder="报名开始日期时间"
                        value-format="yyyy-MM-dd"></el-date-picker>
        至
        <el-date-picker v-model="form.enroll_end_date"
                        type="date"
                        placeholder="报名结束日期时间"
                        value-format="yyyy-MM-dd">
        </el-date-picker>
      </el-form-item>
      <el-form-item label="上课详细地点">
        <el-input v-model="form.address_desc"
                  placeholder="请输入上课详细地点"
                  style="width:50%"
                  clearable />
      </el-form-item>
      <el-form-item label="联系电话">
        <el-input v-model="form.phone"
                  placeholder="请输入联系电话"
                  style="width:30%"
                  clearable />
      </el-form-item>
      <el-form-item label="经纬度">
        <!-- <div v-if="form.latitude">{{form.latitude}},{{form.longitude}}</div>
        <div v-else>请在地图上选点，然后点击“选择该位置”按钮获取经纬度</div>

<<<<<<< HEAD
        <!--  <AMapPosition ref="map"
=======
          <AMapPosition ref="map"
>>>>>>> origin/master
                        @choosePosition="choosePosition"
                        :height="300">

          </AMapPosition> -->
        <AMapPosition ref="map"
                      @choosePosition="choosePosition"
                      :height="300"></AMapPosition>
      </el-form-item>
      <el-form-item label="详细介绍">
        <tinymce :height="400"
                 v-model="form.desc"
                 :value="form.desc" />
      </el-form-item>
    </el-form>
    <div style="text-align:center">
      <el-button style="margin-right:15px"
                 @click="backIndex">返 回</el-button>
      <el-button type="primary"
                 @click="saveData"
                 :loading="btnLoading">提 交</el-button>
    </div>
  </div>
</template>

<script>
import request from "@/utils/request";

import AMapPosition from "@/components/AMapPosition";
import Tinymce from '@/components/Tinymce'

var moment = require("moment");
export default {
  data () {
    return {
      list_location: null,
      list_type: null,
      // listQuery: {
      //   // page: 0,
      //   // limit: 1000000,
      //   keyword: '',
      //   sale_statue: '',
      //   type_id: '',
      // },
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
      },
      btnLoading: false,
      listLoading: true,
      mapQuery: {
        key: '"e189ae2e6e040e0861df1281cb162f6b"',
        Keyword: '',
      },
    }
  },
  created () {
    // console.log(this.$route.query.item);
    // if (this.$route.query.item) {
    //   this.getItemInfo(this.$route.query.item)
    // }
    if (this.$route.query.id) {
      this.getDetail()
    }
    this.getLocationList();
    this.getTypeList();
    // this.getSchoolList();
  },
  methods: {
    search () {
      request({
        url: "https://restapi.amap.com/v3/place/text",
        method: "get",
        params: this.mapQuery
      }).then(response => {
        // console.log(234);
        console.log(response);

      });
    },
    getLocationList () {
      // console.log(123);

      this.listLoading = true;
      request({
        url: "/api/backend/courseSite/index",
        method: "get",
        // params: this.listQuery
      }).then(response => {
        // console.log(234);
        this.list_location = response.data.data;
      });
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
    getDetail () {
      // console.log(123);

      this.listLoading = true;
      request({
        url: "/api/backend/course/detail",
        method: "get",
        params: { id: this.$route.query.id }
      }).then(response => {
        // this.list = response.data.data;
        this.form = response.data
        console.log(this.form.start_time);
        // this.form.start_time = '2019-12-28T' + this.form.start_time + '.000Z'
        // this.form.end_time = '2019-12-28T' + this.form.end_time + '.000Z'
        console.log(this.form.start_time);


      }).catch(err => {
        console.log(err);

      });
    },
    // getItemInfo (item) {
    //   this.form = {
    //     id: item.id,
    //     name: item.name,
    //     title: item.title,
    //     image: item.image,
    //     price: item.price,
    //     volume: item.volume,
    //     site_id: item.site_id,
    //     type_id: item.type_id,
    //     start_time: item.start_time,
    //     end_time: item.end_time,
    //     start_date: moment(item.start_date).format(),
    //     end_date: item.end_date,
    //     address_desc: item.address_desc,
    //     latitude: item.latitude,
    //     longitude: item.longitude,
    //     desc: item.desc,
    //     enroll_start_date: item.enroll_start_date,
    //     enroll_end_date: item.enroll_end_date,
    //     phone: item.phone,
    //   }
    // },
    backIndex () {
      this.$router.replace({ path: '/course/index' })
    },
    choosePosition (res) {
      console.log(res);
      this.form.latitude = res.position.lat;
      this.form.longitude = res.position.lng;
    },
    saveData () {
      // this.form.school_id = this.school_id
      if (!this.form.image) {
        this.$message({
          type: "warning",
          message: "请先选择图片"
        });
        return;
      }
      if (!this.form.title) {
        this.$message({
          type: "warning",
          message: "请输入名称"
        });
        return;
      }
      if (!this.form.price) {
        this.$message({
          type: "warning",
          message: "请输入价格"
        });
        return;
      }
      if (!this.form.site_id) {
        this.$message({
          type: "warning",
          message: "请选择学校"
        });
        return;
      }
      if (!this.form.type_id) {
        this.$message({
          type: "warning",
          message: "请选择科目"
        });
        return;
      }
      if (!this.form.volume) {
        this.$message({
          type: "warning",
          message: "请输入限制人数"
        });
        return;
      }
      if (!this.form.start_time) {
        this.$message({
          type: "warning",
          message: "请输入上课时间"
        });
        return;
      }
      if (!this.form.address_desc) {
        this.$message({
          type: "warning",
          message: "请输入上课详细地点"
        });
        return;
      }
      if (!this.form.longitude) {
        this.$message({
          type: "warning",
          message: "请选择经度"
        });
        return;
      }
      if (!this.form.longitude) {
        this.$message({
          type: "warning",
          message: "请选择纬度"
        });
        return;
      }
      if (!this.form.desc) {
        this.$message({
          type: "warning",
          message: "请输入详细介绍"
        });
        return;
      }
      // console.log(this.form.start_time);
      // this.form.start_time = moment(this.form.start_time).format('H:m:s');
      // this.form.end_time = moment(this.form.end_time).format('H:m:s');
      // console.log(this.form.start_time);
      this.btnLoading = true;
      console.log(this.form);

      request({
        url: "/api/backend/course/store",
        method: "post",
        data: this.form
      })
        .then(() => {
          // this.dialogVisible = false;
          this.getLocationList();
          this.getTypeList();
          this.$message({
            type: "success",
            message: "操作成功!"
          });
          this.backIndex()
        })
        .finally(() => {
          this.btnLoading = false;
        });
    }



  },
  components: {
    AMapPosition,
    Tinymce,
  }
}
</script>

<style scoped>
.line {
  text-align: center;
}
</style>

