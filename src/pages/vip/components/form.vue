<template>
  <div>
    <!-- 4.绑定数据到模板 -->
    <!-- 40 绑定closed -->
    <el-dialog :title="info.title" :visible.sync="info.isshow" @closed="closed">
      <el-form :model="user" :rules="rules">
        <el-form-item label="手机号" label-width="120px" prop="phone">
          <!-- 9.通过v-model将user绑定到表单上 -->
          <el-input v-model="user.phone" autocomplete="off"></el-input>
        </el-form-item>
         <el-form-item label="昵称" label-width="120px" prop="nickname">
          <el-input v-model="user.nickname" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="密码" label-width="120px">
          <el-input v-model="user.password" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="状态" label-width="120px">
          <el-switch v-model="user.status" :active-value="1" :inactive-value="2"></el-switch>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="cancel">取 消</el-button>
        <el-button type="primary"  @click="update">修 改</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import { mapGetters, mapActions } from "vuex";
import {
  reqVipList,
  reqVipDetail,
  reqVipUpdate
} from "../../../utils/http";
import { successAlert,errorAlert } from "../../../utils/alert";
export default {
  // 3.接收info
  props: ["info"],
  data() {
    return {
      rules: {
        phone: [
          { required: true, message: "请输入手机号", trigger: "blur" },
        ],
         nickname: [
          { required: true, message: "请输入昵称", trigger: "blur" },
        ],
      },
      //7.初始化user
      user: {
        uid: "",
        nickname: "",
        phone:"",
        password: "",
        status: 1
      },
      //  11.角色列表
      roleList: []
    };
  },
  computed: {
    ...mapGetters({})
  },
  methods: {
    ...mapActions({}),
    //6.点了取消
    cancel() {
      this.info.isshow = false;
    },
    //17.清空数据
    empty() {
      this.user = {
        uid: "",
        nickname: "",
        phone:"",
        password: "",
        status: 1
      };
    },
    check() {
      return new Promise((resolve, reject) => {
        //验证
        if (this.user.phone === "") {
          errorAlert("手机号不能为空");
          return;
        }
        if (this.user.nickname === "") {
          errorAlert("昵称不能为空");
          return;
        }
        resolve();
      });
    },
    //37 获取详情
    getOne(uid) {
      reqVipDetail(uid).then(res => {
        //此刻user没有id,有uid,而修改要的就是uid
        this.user = res.data.list;
        //处理密码
        this.user.password=""
       
      });
    },
    //39 修改
    update() {
      if (this.user.password) {
         this.check().then(() => {
        reqVipUpdate(this.user).then(res => {
        if (res.data.code == 200) {
          successAlert(res.data.msg);
          this.cancel();
          this.empty();
          this.$emit("init");
        }
      });
      });
      }
     
    },
    //41.处理消失
    closed() {
      if (this.info.title === "编辑会员") {
        this.empty();
      }
    }
  },
  mounted() {
    //   12.一进来，先获取菜单列表数据
    reqVipList().then(res => {
      if (res.data.code == 200) {
        this.roleList = res.data.list;
      }
    });
  }
};
</script>
<style scoped>
</style>