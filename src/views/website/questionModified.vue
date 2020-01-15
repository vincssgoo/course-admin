<template>
  <div class="app-container">
    <el-form ref="form"
             :model="form"
             label-width="120px">
      <el-form-item label="问题">
        <el-input type="textarea"
                  :rows="5"
                  placeholder="请输入问题"
                  v-model="form.title"
                  style="width:70%">
        </el-input>
      </el-form-item>
      <el-form-item label="答案">
        <el-input type="textarea"
                  :rows="5"
                  placeholder="请输入答案"
                  v-model="form.content"
                  style="width:70%">
        </el-input>
      </el-form-item>
      <el-form-item label="权重">
        <el-input type="number"
                  placeholder="请输入权重"
                  v-model="form.weight"
                  style="width:70%"
                  clearable></el-input>
      </el-form-item>
    </el-form>
    <div style="text-align:center;margin-top:200px">
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

export default {

  data () {
    return {
      form: {
        id: '',
        title: '',
        content: '',
        weight: '',
      },
      btnLoading: false,
      listLoading: true,
    }
  },
  created () {
    if (this.$route.query.id) {
      this.getDetail()
    }
  },
  methods: {
    getDetail () {
      console.log(this.$route.query.id);
      this.listLoading = true;
      request({
        url: "/api/backend/question/index",
        method: "get",
      }).then(response => {
        for (let i = 0; i < response.data.data.length; i++) {
          if (response.data.data[i].id == this.$route.query.id) {
            this.form = response.data.data[i]
          }
        }
      }).catch(err => {
        console.log(err);

      })
    },
    backIndex () {
      this.$router.replace({ path: '/website/question' })
    },
    saveData () {
      if (this.$route.query.id) { this.form.id = this.$route.query.id; }
      this.btnLoading = true;
      request({
        url: "/api/backend/question/store",
        method: "post",
        data: this.form
      })
        .then(() => {
          // this.dialogVisible = false;
          this.$message({
            type: "success",
            message: "操作成功!"
          });
        })
        .finally(() => {
          this.btnLoading = false;
        });
    }
  }
}
</script>
