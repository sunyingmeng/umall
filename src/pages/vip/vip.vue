<template>
  <div>
    <v-list :list="list"  @edit="edit" @init="init"></v-list>

    <v-form :info="info"  @init="init" ref="form"></v-form>
  </div>
</template>
<script>
import { mapGetters, mapActions } from "vuex";
import vList from "./components/list.vue";
import vForm from "./components/form.vue";
import { reqVipList} from "../../utils/http";
export default {
  components: {
    vList,
    vForm
  },
  data() {
    return {
      // 1.初始值 弹框数据
      info: {
        isshow: false,
      },
      //19.列表数据初始值
      list: [],
    };
  },
  computed: {
    ...mapGetters({})
  },
  methods: {
    ...mapActions({}),
   
    //获取列表数据发请求
   init() {
      reqVipList().then(res => {
        this.list = res.data.list;
      });
    },
    
    
    //36 编辑
    edit(uid) {
      this.info = {
        isshow: true,
        title: "编辑会员"
      };
      this.$refs.form.getOne(uid);
    }
  },
  mounted() {
    //20.一进来发请求
    this.init();
    
  }
};
</script>
<style scoped>
</style>