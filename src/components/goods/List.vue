<!--  -->
<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input
            placeholder="请输入内容"
            v-model="queryInfo.query"
            clearable
            @clear="getGoodList"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getGoodList"
            ></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="goAddpage">添加商品</el-button>
        </el-col>
      </el-row>

      <!-- table表格区 -->
      <el-table :data="goodslist" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="商品名称" prop="goods_name"></el-table-column>
        <el-table-column
          label="商品价格(元)"
          prop="goods_price"
          width="95px"
        ></el-table-column>
        <el-table-column
          label="商品重量"
          prop="goods_weight"
          width="70px"
        ></el-table-column>
        <el-table-column label="创建时间" prop="add_time" width="140px">
          <template slot-scope="scope">
            {{ scope.row.add_time | dataFormat }}
          </template></el-table-column
        >
        <el-table-column label="操作" width="130px">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditDialog(scope.row.goods_id)"
            ></el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="removeById(scope.row.goods_id)"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页区 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
        background
      >
      </el-pagination>
    </el-card>

    <!-- 编辑商品对话框 -->
    <el-dialog
      title="编辑"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClose"
    >
      <el-form
        :model="editGoods"
        :rules="editGoodsRules"
        ref="editFormRef"
        label-width="70px"
      >
        <el-form-item label="商品名称">
          <el-input v-model="editGoods.goods_name"></el-input>
        </el-form-item>
        <el-form-item label="商品价格">
          <el-input v-model="editGoods.goods_price" type="number"></el-input>
        </el-form-item>
        <el-form-item label="商品数量">
          <el-input v-model="editGoods.goods_number" type="number"></el-input>
        </el-form-item>
        <el-form-item label="商品重量">
          <el-input v-model="editGoods.goods_weight" type="number"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editGoodsInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "List",
  data() {
    return {
      //商品分页数据
      queryInfo: {
        query: "",
        pagenum: 1,
        pagesize: 10,
      },
      //商品列表
      goodslist: [],
      //总数据条数
      total: 0,
      //编辑对话框是否可见
      editDialogVisible: false,
      //查询到的商品信息
      editGoods: {},
      //编辑对话框的验证规则对象
      editGoodsRules: {},
    };
  },
  created() {
    this.getGoodList();
  },
  components: {},
  methods: {
    //根据分页获取商品数据
    async getGoodList() {
      const { data: res } = await this.$http.get("goods", {
        params: this.queryInfo,
      });
      if (res.meta.status !== 200) {
        return this.$message.error("获取商品列表失败！");
      }
      this.$message.success("获取商品列表成功！");
      // console.log(res.data);
      this.goodslist = res.data.goods;
      this.total = res.data.total;
    },
    //
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize;
      this.getGoodList();
    },
    //
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage;
      this.getGoodList();
    },
    //根据id删除商品
    async removeById(id) {
      const confirmResult = await this.$confirm(
        "此操作将永久删除该商品, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).catch((err) => err);
      if (confirmResult !== "confirm") {
        return this.$message.info("已取消删除");
      }
      const { data: res } = await this.$http.delete(`goods/${id}`);
      if (res.meta.status !== 200) {
        return this.$message.error("删除商品失败!");
      }
      this.$message.success("删除商品成功！");
      this.getGoodList();
    },
    //跳转到添加商品的页面
    goAddpage() {
      this.$router.push("/goods/add");
    },
    //展示编辑对话框
    async showEditDialog(id) {
      const { data: res } = await this.$http.get("goods/" + id);
      if (res.meta.status !== 200) {
        return this.$message.error("查询商品信息失败");
      }
      this.editGoods = res.data;
      console.log(res.data);
      this.editDialogVisible = true;
    },
    //监听编辑对话框关闭事件
    editDialogClose() {
      this.$refs.editFormRef.resetFields();
    },
    //修改商品信息并提交
    editGoodsInfo() {
      this.$refs.editFormRef.validate(async (valid) => {
        if (!valid) return;
        const { data: res } = await this.$http.put(
          "goods/" + this.editGoods.goods_id,
          {
            goods_name: this.editGoods.goods_name,
            goods_price: this.editGoods.goods_price,
            goods_number: this.editGoods.goods_number,
            goods_weight: this.editGoods.goods_weight,
          }
        );

        if (res.meta.status !== 200) {
          return this.$message.error("更新商品信息失败");
        }
        this.editDialogVisible = false;
        this.getGoodList();
        this.$message.success("修改商品信息成功！");
      });
    },
  },
};
</script>

<style  scoped lang="less">
</style>
