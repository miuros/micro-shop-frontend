<!--
 * @Description: 商品详情页面组件
 * @Author: hai-27
 * @Date: 2020-02-16 20:20:26
 * @LastEditors: hai-27
 * @LastEditTime: 2020-03-07 21:59:26
 -->
<template>
  <div id="details">
    <!-- 头部 -->
    <div class="page-header">
      <div class="title">
        <p>{{productDetails.name}}</p>
        <div class="list">
          <ul>
            <li>
              <router-link to>概述</router-link>
            </li>
            <li>
              <router-link to>参数</router-link>
            </li>
            <li>
              <router-link to>用户评价</router-link>
            </li>
          </ul>
        </div>
      </div>
    </div>
    <!-- 头部END -->

    <!-- 主要内容 -->
    <div class="main">
      <!-- 左侧商品轮播图 -->
      <div class="block">
        <el-carousel height="560px" v-if="productPicture.length>1">
          <el-carousel-item v-for="item in productPicture" :key="item.id">
            <img style="height:560px;" :src="item" :alt="item" />
          </el-carousel-item>
        </el-carousel>
        <div v-if="productPicture.length==1">
          <img
            style="height:560px;"
            :src="productPicture[0]"
            :alt="productPicture[0]"
          />
        </div>
      </div>
      <!-- 左侧商品轮播图END -->

      <!-- 右侧内容区 -->
      <div class="content">
        <h1 class="name">{{productDetails.name}}</h1>
        <p class="intro">{{productDetails.desc}}</p>
        <p class="store">{{shopInfo.name}} <router-link :to="{path: '/chat?id='+shopInfo.userUuid}">点击联系</router-link></p>
        <div class="price">
          <span>{{productDetails.sellPrice}}元</span>
          <span
            v-show="productDetails.originPrice != productDetails.sellPrice"
            class="del"
          >{{productDetails.originPrice}}元</span>
        </div>
        <div class="pro-list">
          <span class="pro-name">{{productDetails.name}}</span>
          <span class="pro-price">
            <span>{{productDetails.sellPrice}}元</span>
            <span
              v-show="productDetails.originPrice != productDetails.sellPrice"
              class="pro-del"
            >{{productDetails.originPrice}}元</span>
          </span>
          <p class="price-sum">总计 : {{productDetails.sellPrice}}元</p>
        </div>
        <!-- 内容区底部按钮 -->
        <div class="button">
          <el-button class="shop-cart" :disabled="dis" @click="addShoppingCart">加入购物车</el-button>
        </div>
        <!-- 内容区底部按钮END -->
        <div class="pro-policy">
          <ul>
            <li>
              <i class="el-icon-circle-check"></i> 小米自营
            </li>
            <li>
              <i class="el-icon-circle-check"></i> 小米发货
            </li>
            <li>
              <i class="el-icon-circle-check"></i> 7天无理由退货
            </li>
            <li>
              <i class="el-icon-circle-check"></i> 7天价格保护
            </li>
          </ul>
        </div>
      </div>
      
      <!-- 右侧内容区END -->
    </div>
     <ul>
        <li style="font-size: 36px;position: relative;left: 200px">评论</li>
        <el-input v-model="comment" style="width: 400px;margin: auto auto;position:relative;left: 200px" placeholder="请输入内容"></el-input><el-button style="position: relative;margin-left: 200px;" @click="uploadCm">提交</el-button>
        <div class="list">
        <li v-for="item in cmList" :key="item.id" class="infinite-list-item">
          <div class="cm">
            
          <p style="max-width: 110px;overflow: hidden;text-overflow: ellipsis;white-space: nowrap;float: left;left: 20px;margin-left: 10px;">{{item.userUuid}}</p>: {{item.content}}
          </div>
        </li>
        </div>
      
    </ul>
    <!-- 主要内容END -->
  </div>
</template>
<script>
import { mapActions } from "vuex";
export default {
  data() {
    return {
      page: 1,
      limit: 10,
      dis: false, // 控制“加入购物车按钮是否可用”
      productID: "", // 商品id
      productDetails: "", // 商品详细信息
      productPicture: "", // 商品图片
      shopInfo: "",
      cmList: [],
      comment:"",
    };
  },
  // 通过路由获取商品id
  activated() {
    if (this.$route.query.productID != undefined) {
      this.productID = this.$route.query.productID;
       this.listCm();
    }
  },
  created(){
   

  },
  watch: {
    // 监听商品id的变化，请求后端获取商品数据
    productID: function(val) {
      this.getDetails(val);
      this.productID=val;
      this.cmList=[];
      this.page=1;
    }
  },
  methods: {
    uploadCm(){
      this.$axios.post('/api/cm/v1/create',JSON.stringify({
          content: this.comment,
          productId: parseInt(this.productID),
      }),{
        headers: {
          'Content-Type':'application/json'
        }
      }).then(res=>{
        if(res.data.code==='200'){
          this.notifySucceed(res.data.msg);
          this.comment="";
        }
      }).catch(err=>{
        console.log(err);
      })
    },
    async listCm(){
      var res=await this.$axios.get('/api/cm/v1/list?productid='+this.productID+"&page="+this.page+"&limit="+this.limit);
      if (res.data.code==='200'){
        this.page=this.page+1;
        this.cmList=res.data.data;
      }else{
        this.notifyError(res.data.msg);
      }
    },
    ...mapActions(["unshiftShoppingCart", "addShoppingCartNum"]),
    // 获取商品详细信息
    async getDetails(val) {
      await this.$axios
        .get("/api/pd/v1/get?id="+val, {
          productID: val,
        })
        .then(res => {
          this.productDetails = res.data.data;
          this.productPicture=res.data.data.imageUrl.split(";");
        })
        .catch(err => {
          return Promise.reject(err);
        });
      var res=await this.$axios.get('/api/sp/v1/get?id='+this.productDetails.shopId)
      if (res.data.code==='200'){
        this.shopInfo=res.data.data;
      }
    },
    // 获取商品图片
    /*
    getDetailsPicture(val) {
      this.$axios
        .post("/api/product/getDetailsPicture", {
          productID: val
        })
        .then(res => {
          this.productPicture = res.data.ProductPicture;
        })
        .catch(err => {
          return Promise.reject(err);
        });
    },
    */
    // 加入购物车
    addShoppingCart() {
      // 判断是否登录,没有登录则显示登录组件
      if (!this.$store.getters.getUser) {
        this.$store.dispatch("setShowLogin", true);
        return;
      }
      this.$axios
        .post("/api/ct/v1/create", {
          productId: parseInt(this.productID),
          num: 1,
        })
        .then(res => {
          switch (res.data.code) {
            case "200":
              // 新加入购物车成功
              this.unshiftShoppingCart(res.data.data);
              this.notifySucceed(res.data.msg);
              break;
            default:
              this.notifyError(res.data.msg);
          }
        })
        .catch(err => {
          return Promise.reject(err);
        });
    },
  }
};
</script>
<style>
ul{
  margin-top: 80px;
}

.infinite-list{
  width: 100%;

}

.cm{
  position: relative;
  left:10px;
  height: 40px;
}
.infinite-list-item{
  width: 600px;
  left:250px;
  position: relative;
  margin: 10px 20px;
  font-size: 24px;
}
.input-area {
    width: 85%;
    border-radius: 4px;
    height: 100px;
    width: 100px;
    margin: auto;
    margin-top: 20px;
    border:0 solid black;
}
.send-btn {
    position: absolute;
    right: 10px;
    bottom: 10px;
}
.input {
    width: 100%;
    height: 100%;
    border:  0px #ebebeb solid;
    border-radius: 4px;
    outline: none;
    padding: 5px;
}
.list{
  background-color: #ebebeb;
  width: 75%;
  margin: auto auto;
  border: 10px;
}

/* 头部CSS */
#details .page-header {
  height: 64px;
  margin-top: -20px;
  z-index: 4;
  background: #fff;
  border-bottom: 1px solid #e0e0e0;
  -webkit-box-shadow: 0px 5px 5px rgba(0, 0, 0, 0.07);
  box-shadow: 0px 5px 5px rgba(0, 0, 0, 0.07);
}
#details .page-header .title {
  width: 1225px;
  height: 64px;
  line-height: 64px;
  font-size: 18px;
  font-weight: 400;
  color: #212121;
  margin: 0 auto;
}
#details .page-header .title p {
  float: left;
}
#details .page-header .title .list {
  height: 64px;
  float: right;
}
#details .page-header .title .list li {
  float: left;
  margin-left: 20px;
}
#details .page-header .title .list li a {
  font-size: 14px;
  color: #616161;
}
#details .page-header .title .list li a:hover {
  font-size: 14px;
  color: #ff6700;
}
/* 头部CSS END */

/* 主要内容CSS */
#details .main {
  width: 1225px;
  height: 560px;
  padding-top: 30px;
  margin: 0 auto;
}
#details .main .block {
  float: left;
  width: 560px;
  height: 560px;
}
#details .el-carousel .el-carousel__indicator .el-carousel__button {
  background-color: rgba(163, 163, 163, 0.8);
}
#details .main .content {
  float: left;
  margin-left: 25px;
  width: 640px;
}
#details .main .content .name {
  height: 30px;
  line-height: 30px;
  font-size: 24px;
  font-weight: normal;
  color: #212121;
}
#details .main .content .intro {
  color: #b0b0b0;
  padding-top: 10px;
}
#details .main .content .store {
  color: #ff6700;
  padding-top: 10px;
}
#details .main .content .price {
  display: block;
  font-size: 18px;
  color: #ff6700;
  border-bottom: 1px solid #e0e0e0;
  padding: 25px 0 25px;
}
#details .main .content .price .del {
  font-size: 14px;
  margin-left: 10px;
  color: #b0b0b0;
  text-decoration: line-through;
}
#details .main .content .pro-list {
  background: #f9f9fa;
  padding: 30px 60px;
  margin: 50px 0 50px;
}
#details .main .content .pro-list span {
  line-height: 30px;
  color: #616161;
}
#details .main .content .pro-list .pro-price {
  float: right;
}
#details .main .content .pro-list .pro-price .pro-del {
  margin-left: 10px;
  text-decoration: line-through;
}
#details .main .content .pro-list .price-sum {
  color: #ff6700;
  font-size: 24px;
  padding-top: 20px;
}
#details .main .content .button {
  height: 55px;
  margin: 10px 0 20px 0;
}
#details .main .content .button .el-button {
  float: left;
  height: 55px;
  font-size: 16px;
  color: #fff;
  border: none;
  text-align: center;
}
#details .main .content .button .shop-cart {
  width: 340px;
  background-color: #ff6700;
}
#details .main .content .button .shop-cart:hover {
  background-color: #f25807;
}

#details .main .content .button .like {
  width: 260px;
  margin-left: 40px;
  background-color: #b0b0b0;
}
#details .main .content .button .like:hover {
  background-color: #757575;
}
#details .main .content .pro-policy li {
  float: left;
  margin-right: 20px;
  color: #b0b0b0;
}
/* 主要内容CSS END */
</style>