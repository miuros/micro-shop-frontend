<template>
  <div>
    <!--面包屑导航区域-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item >首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!--卡片视图区域-->
    <el-card class="box-card">


      <!--  用户列表区域-->
      <el-table :data="orderList"
                :stripe="true"
                border
                element-loading-text="拼命加载中"
                style="width: 100%">
        <el-table-column label="#"
                         type="index">
        </el-table-column>
        <el-table-column label="用户uid"
                         prop="userUuid">
        </el-table-column>
        <el-table-column label="商品id"
                         prop="productId">
        </el-table-column>

        <el-table-column label="数量"
                         prop="number">
        </el-table-column>
        <el-table-column label="支付时间"
                         prop="payTime">
        </el-table-column>
        <el-table-column label="订单状态" prop="status">
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-tooltip :enterable="false" class="item" content="获取用户信息" effect="dark" placement="top">
              <el-button icon="el-icon-edit" size="mini" type="primary" @click="getInfo(scope.row.addressId,scope.row.userUuid)"
              ></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <el-dialog :visible.sync="editDialogVisible"
      title=""
      width="35%"
      @close="editDialogClosed">
        <el-table :data="ar" width="800px">
        <el-table-column label="#"
                         type="index">
        </el-table-column>
        <el-table-column label="mobile"
                         prop="mobile" width="200px">
        </el-table-column>
        <el-table-column label="address"
                         prop="address" width="200px">
        </el-table-column>
        <el-table-column label="alias" prop="alias" width="100px"></el-table-column>
        </el-table>
      </el-dialog>
      <!--分页区域-->
      <el-pagination
        :current-page="queryInfo.page"
        :page-size="this.queryInfo.limit"
        :page-sizes="[2, 4, 5, 10]"
        :total="total"
        layout="total, sizes, prev, pager, next, jumper"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange">
      </el-pagination>
    </el-card>
  </div>
</template>

<script>
export default {
  name: 'Orders.vue',
  data () {
    // 验证邮箱的规则
    return {
      editDialogVisible: false,
      editDialogClosed: false,

      ar: [],
      shop: "",
      // 获取用户列表的参数对象
      queryInfo: {
        extra: '',
        // 当前的页数
        page: 1,
        limit: 2
      },
      orderList: [],
      total: 0,

      orderInfo: {},
      // 所有角色的数据列表
      // 已选中的角色Id值
      selectedRoled: '',
      // loading状态控制
      loading: false
    }
  },
  created () {
    this.shop=JSON.parse(localStorage.getItem('shop'));
    this.getOrderList()
  },
  methods: {
    getInfo(id,uid){
      this.$axios.get('/api/ar/v1/get?id='+id+"&userUuid="+uid).then(res=>{
        if(res.data.code==='200'){
          this.ar=[res.data.data];
          this.notifySucceed(res.data.msg);
          console.log(this.ar);
        }else{
          this.notifyError(res.data.msg);
        }
      })
      this.editDialogVisible=true;
    },
    async getOrderList () {
      this.loading = true
      const { data: res } = await this.$axios.get('/api/od/v1/listforsp?status=0&page='+this.queryInfo.page+"&limit="+this.queryInfo.limit+"&shopId="+this.shop.id)
      console.log(res)
      if (res.code !== '200') {
        return this.notifyError('获取订单列表失败!')
      }
      this.orderList = res.data
      this.total = res.data.length
      this.loading = false
    },
    /**
     * 监听 pagesize 改变的事件
     * @param newSize
     */
    handleSizeChange (newSize) {
      // console.log(newSize)
      this.queryInfo.pagesize = newSize
      this.getOrderList()
    },
    /**
     * 监听 页面值 改变的事件
     * @param newPage
     */
    handleCurrentChange (newPage) {
      // console.log(newPage)
      this.queryInfo.pagenum = newPage
      this.getOrderList()
    },
  }
}
</script>

<style lang="less" scoped>
</style>
