<template>
  <div class="add">
    <el-dialog :title="info.title" :visible.sync="info.isshow" @closed="closed">
      <el-form :model="user" :rules="rules">
        <el-form-item label="标题" label-width="120px" prop="title">
          <el-input v-model="user.title" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图片" label-width="120px" v-if="user.pid!==0">
          
          <div class="myupload">
            <h3>+</h3>
            <img class="img" v-if="imgUrl" :src="imgUrl" alt="">
           
            <input v-if="info.isshow" type="file" class="ipt" @change="changeFile">
          </div>
        </el-form-item>
        <el-form-item label="状态" label-width="120px">
          <el-switch v-model="user.status" :active-value="1" :inactive-value="2"></el-switch>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="cancel">取 消</el-button>
        <el-button type="primary" v-if="info.title=='添加轮播图'" @click="add">添 加</el-button>
        <el-button type="primary" v-else @click="update">修 改</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import path from "path";
import { mapGetters, mapActions } from "vuex";
import {
  reqbannerList,
  reqbannerAdd,
  reqbannerDetail,
  reqbannerUpdate
} from "../../../utils/http";
import { successAlert, errorAlert } from "../../../utils/alert";
export default {
  props: ["info"],
  data() {
    return {
      rules: {
        title: [
          { required: true, message: "请输入手机号", trigger: "blur" },
        ]
      },
      user: {
        title: "",
        img: null,
        status: 1
      },
      imgUrl: ""
    };
  },
  computed: {
    ...mapGetters({
      cateList: "cate/list"
    })
  },
  methods: {
    // 3.选择了文件
    changeFile(e) {
      let file = e.target.files[0];

      if (file.size > 2 * 1024 * 1024) {
        errorAlert("文件大小不能超过2M");
        return;
      }

      //判断文件类型
      let extname = path.extname(file.name); //'.jpg'
      let arr = [".jpg", ".jpeg", ".png", ".gif"];
      if (!arr.includes(extname)) {
        errorAlert("请上传正确的图片格式！");
        return;
      }

      //URL.createObjectURL(file) 将一个文件生成一个URL地址
      this.imgUrl = URL.createObjectURL(file);

      //给user.img赋值
      this.user.img = file;
    },
    //element-ui的上传文件
    changeFile2(e){
      let file=e.raw;


      this.imgUrl=URL.createObjectURL(file)

      this.user.img=file;
    },

    ...mapActions({
      reqList: "banner/reqList"
    }),
    //6.点了取消
    cancel() {
      this.info.isshow = false;
    },
    //17.清空数据
    empty() {
      this.user = {
        title: "",
        img: null,
        status: 1
      };
      this.imgUrl = "";
    },
    check() {
      return new Promise((resolve, reject) => {
        //验证
        if (this.user.title === "") {
          errorAlert("标题不能为空");
          return;
        }
        
        resolve();
      });
    },
    //10.点了添加按钮
    add() {
      
      this.check().then(() => {
      reqbannerAdd(this.user).then(res => {
        if (res.data.code == 200) {
          successAlert("添加成功");
          this.cancel();
          this.empty();
          this.reqList();
          
        }
      });
      });
    },
    //37 获取详情
    getOne(id) {
      reqbannerDetail(id).then(res => {
        this.user = res.data.list;
        this.imgUrl = this.$imgPre + this.user.img;
        this.user.id = id;
      });
    },
    //39 修改
    update() {
      this.check().then(() => {
        reqbannerUpdate(this.user).then(res => {
        if (res.data.code == 200) {
          successAlert("修改成功");
          this.cancel();
          this.empty();
          this.reqList();
        }
      });
      });
    },
    //41.处理消失
    closed() {
      if (this.info.title === "编辑轮播图") {
        this.empty();
      }
    }
  },
  mounted() {
    reqbannerList().then(res => {
      if (res.data.code == 200) {
        this.bannerList = res.data.list;
      }
    });
  }
};
</script>
<style scoped lang="stylus">
.myupload {
  width: 100px;
  height: 100px;
  border-radius: 5px;
  border: 1px dashed #ccc;
  position: relative;
}
.myupload h3 {
  width: 100%;
  height: 100px;
  font-size: 30px;
  text-align: center;
  line-height: 100px;
  color: #666;
  font-weight: 100;
}
.myupload .ipt {
  width: 100px;
  height: 100px;
  position: absolute;
  left: 0;
  top: 0;
  opacity: 0;
}
.myupload .img {
  width: 100px;
  height: 100px;
  position: absolute;
  left: 0;
  top: 0;
}

</style>