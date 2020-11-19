<template>
  <div>
    <el-dialog :title="info.title" :visible.sync="info.isshow" @opened="opened" @closed="closed">
      <el-form :model="user" :rules="rules">
        <!-- 2.绑定数据 -->
        <el-form-item label="活动名称" label-width="120px" prop="title">
          <el-input v-model="user.title" placeholder="请输入活动名称"></el-input>
        </el-form-item>
        <span class="demonstration">活动期限</span>
        <el-date-picker
          v-model="value1"
          type="datetimerange"
          range-separator="至"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
        ></el-date-picker>
        <!-- <el-form-item label="活动期限" label-width="120px" prop="title">
          <el-input v-model="user.title" placeholder="请输入活动期限"></el-input>
        </el-form-item>-->
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
          <el-select placeholder="请选择二级分类" v-model="user.second_cateid" @change="changeSecond">
            <el-option
              v-for="item in secondCateList"
              :key="item.id"
              :label="item.catename"
              :value="item.id"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="商品" label-width="120px" prop="goodsid">
          <el-select placeholder="请选择商品" v-model="user.goodsid">
            <el-option
              v-for="item in reqgoodsList"
              :key="item.id"
              :label="item.goodsname"
              :value="item.id"
            ></el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="状态" label-width="120px">
          <el-switch v-model="user.status" :active-value="1" :inactive-value="2"></el-switch>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="cancel">取 消</el-button>
        <el-button type="primary" v-if="info.title=='添加活动'" @click="add">添 加</el-button>
        <el-button type="primary" v-else @click="update">修 改</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import E from "wangeditor";
import { mapGetters, mapActions } from "vuex";
import {
  reqcateList,
  reqseckAdd,
  reqseckDetail,
  reqseckUpdate,
  reqspecsDel,
  reqgoodsList,
} from "../../../utils/http";
import { successAlert, errorAlert } from "../../../utils/alert";
export default {
  props: ["info"],
  data() {
    return {
     
      rules: {
        title: [{ required: true, message: "请输入活动名称", trigger: "blur" }],
        
      },
      //1.初始化数据
      user: {
        title: "",
        begintime: "",
        endtime: "",
        first_cateid: "",
        second_cateid: "",
        goodsid: "",
        status: 1,
      },
      //二级分类的list
      secondCateList: [],
      reqgoodsList: [],
     
       value1: [],
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
      reqseckList: "seck/reqList",
    }),
    // 5.根据一级分类id，得到二级分类list
    changeFirst() {
      //二级分类的id重置
      this.user.second_cateid = "";
      this.getSecondList();
    },
    changeSecond() {
      //三级分类的id重置
      this.user.goodsid = "";
      this.getThirdList();
    },
    getSecondList() {
      //获取二级分类list
      reqcateList({ pid: this.user.first_cateid }).then((res) => {
        this.secondCateList = res.data.list;
      });
    },
    getThirdList() {
      //获取三级分类list
      reqgoodsList({
        sid: this.user.second_cateid,
        fid: this.user.first_cateid,
      }).then((res) => {
        this.reqgoodsList = res.data.list;
      });
    },

    cancel() {
      this.info.isshow = false;
    },

    //清空
    empty() {
      this.user = {
        title: "",
        begintime: "",
        endtime: "",
        first_cateid: "",
        second_cateid: "",
        goodsid: "",
        status: 1,
      };
      //二级分类的list
      this.secondCateList = [];
      this.reqgoodsList = [];
      //图片临时地址
      this.imgUrl = "";
      //规格属性list
      this.attrsList = [];
    },

    //验证
    check() {
      return new Promise((resolve, reject) => {
        //验证
        if (this.user.title === "") {
          errorAlert("活动名称为空");
          return;
        }

        resolve();
      });
    },
    //添加
    add() {
      this.check().then(() => {
          this.user.begintime = this.value1[0].getTime();
          this.user.endtime = this.value1[1].getTime();
          console.log(this.user);
        reqseckAdd(this.user).then((res) => {
          if (res.data.code === 200) {
            successAlert("添加成功");
            this.cancel();
            this.empty();
            //刷新list
            this.reqseckList();
          }
        });
      });
    },
    //详情
    getOne(id) {
      reqseckDetail(id).then((res) => {
        this.user = res.data.list;
        this.user.id = id;
        //请二级list
        this.getSecondList();
        this.getThirdList();
        this.value1=[new Date(parseInt(this.user.begintime)),new Date(parseInt(this.user.endtime))]
      });
    },
    //更新
    update() {
         
      this.check().then(() => {
          this.user.begintime = this.value1[0].getTime();
          this.user.endtime = this.value1[1].getTime();
        reqseckUpdate(this.user).then((res) => {
          if (res.data.code == 200) {
            successAlert("更新成功");
            this.cancel();
            this.empty();
            this.reqseckList();
             this.value1=[new Date(parseInt(this.user.begintime)),new Date(parseInt(this.user.endtime))]
          }
        });
      });
    },
    closed() {
      if (this.info.title == "编辑活动") {
        this.empty();
        this.value1=[]
      }
    },
    //弹框打开，并且动画结束了
    opened() {
      this.editor = new E("#edit");
      this.editor.create();
    },
  },
  mounted() {
    // 5.一进来请求一级分类list
    this.reqCateList();
    this.reqseckList();
  },
};
</script>
<style scoped>
.demonstration {
  padding-left: 50px;
  padding-right: 10px;
}
</style>