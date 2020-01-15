<template>
  <div class="app-container">
    <el-form ref="form"
             :model="form"
             label-width="120px">
      <el-form-item>
        <div class="content">
          <div class="bili">
            <span>图片</span>
            <span>(690 x 290)</span>
          </div>
          <upload-one v-model="form.image">
          </upload-one>
        </div>

      </el-form-item>
      <el-form-item>
        <div class="weight">
          <span>权重</span>
          <el-input type="number"
                    placeholder="请输入权重"
                    v-model="form.weight"
                    style="width:30%"
                    clearable></el-input>
        </div>

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
        image: '',
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
        url: "/api/backend/banner/index",
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
      this.$router.replace({ path: '/website/index' })
    },
    saveData () {
      if (this.$route.query.id) { this.form.id = this.$route.query.id; }
      this.btnLoading = true;
      request({
        url: "/api/backend/banner/store",
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
<style>
body {
  margin: 0;
  padding: 0;
}
.content {
  display: flex;
  flex-direction: row;
}
.bili {
  display: flex;
  flex-direction: column;
}
.bili > span {
  margin-right: 20px;
}
.weight {
  display: flex;
  flex-direction: row;
}
.weight > span {
  margin-right: 42px;
  width: 50px;
}
</style>