<template>
  <div class="app-container">
    <div class="content">
      <span style="width:80px">公告</span>
      <el-input type="textarea"
                :rows="5"
                placeholder="请输入内容"
                v-model="form.notice">
      </el-input>
    </div>
    <div class="content">
      <span style="width:80px">报名须知</span>

      <tinymce :height="600"
               v-model="form.notes"
               :value="form.notes"
               style="width:100%" />
    </div>
    <div class="content">
      <span style="width:80px">关于我们</span>
      <tinymce :height="600"
               v-model="form.about_us"
               :value="form.about_us"
               style="width:100%" />
    </div>
    <div style="text-align:center">
      <el-button type="primary"
                 @click="saveData"
                 :loading="btnLoading">提 交</el-button>
    </div>

  </div>
</template>

<script>
import request from "@/utils/request";
import Tinymce from '@/components/Tinymce'
export default {
  data () {
    return {
      list: null,
      listLoading: true,
      btnLoading: false,
      form: {
        notice: '',
        notes: '',
        about_us: '',
      }
    }
  },
  components: {
    Tinymce
  },
  created () {
    this.getList()
  },
  methods: {
    getList () {
      this.listLoading = true;
      request({
        url: "/api/backend/globalSetting/detail",
        method: "get",
      }).then(response => {
        console.log(response.data);

        this.form.about_us = response.data[0].value;
        this.form.notice = response.data[1].value;
        this.form.notes = response.data[2].value;
        console.log(this.form.about_us);

        this.listLoading = false;
      });

    },
    saveData () {

      this.btnLoading = true;
      request({
        url: "/api/backend/globalSetting/store",
        method: "post",
        data: this.form
      })
        .then(() => {

          // this.dialogVisible = false;
          this.$message({
            type: "success",
            message: "操作成功!"
          });
          this.getList()
        })
        .finally(() => {
          this.btnLoading = false;
        });
    }
  }
}
</script>
<style>
.content {
  display: flex;
  flex-wrap: nowrap;
  margin-bottom: 60px;
}
.app-container {
  /* display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center; */
}
</style>