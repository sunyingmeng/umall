<template>
  <div class="add">
    <el-dialog :title="info.title" :visible.sync="info.isshow" @closed="closed">
      <el-form :model="user">
        <el-form-item label="活动名称" label-width="120px">
          <el-input v-model="user.title" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="活动期限" label-width="120px">
          <el-input v-model="user.catename" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="一级分类" label-width="120px" prop="first_cateid">
          <el-select placeholder="请选择一级分类" v-model="user.first_cateid" @change="changeFirst">
            <!-- 6.遍历一级分类 -->

            <el-option
              v-for="item in cateList"
              :key="item.id"
              :label="item.catename"
              :value="item.id"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="二级分类" label-width="120px" prop="second_cateid">
          <el-select placeholder="请选择二级分类" v-model="user.second_cateid">
            <el-option
              v-for="item in secondCateList"
              :key="item.id"
              :label="item.catename"
              :value="item.id"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="商品" label-width="120px" prop="goodsname">
          <el-input v-model="user.goodsname" placeholder="请输入商品名称"></el-input>
        </el-form-item>

        <el-form-item label="状态" label-width="120px">
          <el-switch v-model="user.status" :active-value="1" :inactive-value="2"></el-switch>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="cancel">取 消</el-button>
        <el-button type="primary" v-if="info.title=='添加分类'" @click="add">添 加</el-button>
        <el-button type="primary" v-else @click="update">修 改</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import path from "path";
import { mapGetters, mapActions } from "vuex";
import {
  reqCateList,
  reqSpecsList,
  reqseckList,
  reqseckAdd,
  reqseckDetail,
  reqseckUpdate,
  
} from "../../../utils/http";
import { successAlert, errorAlert } from "../../../utils/alert";
export default {
  props: ["info"],
  data() {
    return {
      rules: {
        first_cateid: [
          { required: true, message: "请输入一级分类", trigger: "change" }
        ],
        second_cateid: [
          { required: true, message: "请输入二级分类", trigger: "change" }
        ],
        goodsname: [
          { required: true, message: "请输入商品名称", trigger: "blur" }
        ]
      },
      user: {
        title: "",
        begintime: "",
        endtime: "",
        first_cateid: "",
        second_cateid: "",
        goodsid: "",
        status: 1,
      },
    };
  },
  computed: {
    ...mapGetters({
      //3。一级分类list
      cateList: "cate/list",
      // 7.规格list
      specsList: "specs/list",
    }),
  },
  methods: {
    ...mapActions({
      // 4.请求一级分类list
      reqCateList: "cate/reqList",
      //8.请求规格list
      reqSpecsList: "specs/reqList",
      //商品list和总数
      reqseckList: "seck/reqList",
      reqseckCount: "seck/reqCount",
    }),
    // 5.根据一级分类id，得到二级分类list
    changeFirst() {
      //二级分类的id重置
      this.user.second_cateid = "";
      this.getSecondList();
    },
    getSecondList() {
      //获取二级分类list
      reqcateList({ pid: this.user.first_cateid }).then((res) => {
        this.secondCateList = res.data.list;
      });
    },

    //6.点了取消
    cancel() {
      this.info.isshow = false;
    },
    //17.清空数据
    empty() {
      this.user = {
        pid: "",
        catename: "",
        img: null,
        status: 1,
      };
      this.imgUrl = "";
    },
    //10.点了添加按钮
    add() {
      //16.ajax
      reqseckAdd(this.user).then((res) => {
        if (res.data.code == 200) {
          successAlert("添加成功");
          this.cancel();
          this.empty();
          this.reqList();
        }
      });
    },
    //37 获取详情
    getOne(id) {
      reqseckDetail(id).then((res) => {
        this.user = res.data.list;
        this.imgUrl = this.$imgPre + this.user.img;
        this.user.id = id;
      });
    },
    //39 修改
    update() {
      reqseckUpdate(this.user).then((res) => {
        if (res.data.code == 200) {
          successAlert("修改成功");
          this.cancel();
          this.empty();
          this.reqList();
        }
      });
    },
    //41.处理消失
    closed() {
      if (this.info.title === "编辑分类") {
        this.empty();
      }
    },
  },
  mounted() {
    reqseckList().then((res) => {
      if (res.data.code == 200) {
        this.roleList = res.data.list;
      }
    });
  },
};
</script>
<style scoped>
</style>