<!--
 * @Description: 首页组件
 * @Author: hai-27
 * @Date: 2020-02-07 16:23:00
 * @LastEditors: hai-27
 * @LastEditTime: 2020-02-27 13:36:12
 -->
<template>
  <div class="home" id="home" name="home">
    <!-- 轮播图 -->
    <div class="block">
      <el-carousel height="460px">
        <el-carousel-item v-for="item in carousel" :key="item.id">
          <img style="height:460px;" :src="item.imageUrl" alt="" />
        </el-carousel-item>
      </el-carousel>
    </div>
    <!-- 轮播图END -->
    <div class="main-box">
      <div class="main">
        <!-- 手机商品展示区域 -->
        <div class="phone">
          <div class="box-hd">
            <div class="title">手机</div>
          </div>
          <div class="box-bd">
            <div class="promo-list">
              <ul>
                <li>
                  <img src="https://g-search3.alicdn.com/img/bao/uploaded/i4/i1/1917047079/O1CN01WvMcdY22AEUdIiK9y_!!2-item_pic.png_580x580Q90.jpg" />
                </li>
              </ul>
            </div>
            <div class="list">
              <MyList :list="phoneList" :isMore=true></MyList>
            </div>
          </div>
        </div>
        <!-- 手机商品展示区域END -->

        <!-- 家电商品展示区域 -->
        <div class="appliance" id="promo-menu">
          <div class="box-hd">
            <div class="title">水果</div>
          </div>
          <div class="box-bd">
            <div class="promo-list">
              <ul>
                <li>
                  <img src="https://photo-static-api.fotomore.com/creative/vcg/veer/nowarter800/new/VCG41N525961770.jpg" />
                </li>
              </ul>
            </div>
            <div class="list">
              <MyList :list="fruitList" :isMore=true></MyList>
            </div>
          </div>
        </div>
        <div class="accessory" id="promo-menu">
          <div class="box-hd">
            <div class="title">书籍</div>
          </div>
          <div class="box-bd">
            <div class="promo-list">
              <ul>
                <li>
                  <img src="https://img.alicdn.com/tps/i4/TB1seoIJFXXXXaQXFXXvMNvHpXX-460-780.jpg" alt />
                </li>
              </ul>
            </div>
            <div class="list">
              <MyList :list="bookList" :isMore=true></MyList>
            </div>
          </div>
        </div>
        <!-- 配件商品展示区域END -->
      </div>
    </div>
  </div>
</template>
<script>
import MyList from "../components/MyList.vue";
export default {
    data() {
        return {
            cgList: [],
            carousel: [],
            phoneList: [],
            bookList: [],
            data: {},
            fruitList: "",
        };
    },
    watch: {
    },
    async created() {
        // 获取轮播图数据
        var res = "";
        res = await this.$axios.get("/api/bn/v1/list", {});
        this.carousel = res.data.data;
        // 获取各类商品数据
        res = await this.$axios.get("/api/cg/v1/list?limit=3");
        this.cgList = res.data.data;
        for (var i = 0; i < this.cgList.length; i++) {
            res = await this.$axios.get("/api/pd/v1/listbycg?categoryId=" + this.cgList[i].id);
            if (res.data.code == 200) {
                if (this.cgList[i].name == "phone") {
                    this.phoneList = res.data.data;
                }
                else if (this.cgList[i].name == "book") {
                    this.bookList = res.data.data;
                }
                else if (this.cgList[i].name == "fruit") {
                    this.fruitList = res.data.data;
                }
            }
        }
    },
    methods: {
        // 获取家电模块子组件传过来的数据
        getChildMsg(val) {
            this.applianceActive = val;
        },
        // 获取配件模块子组件传过来的数据
        getChildMsg2(val) {
            this.accessoryActive = val;
        },
        // 获取各类商品数据方法封装
    },
    components: { MyList }
};
</script>
<style scoped>
@import "../assets/css/index.css";
</style>