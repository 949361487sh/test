<template>
  <div class="orderBox">
    <el-form :inline="true" :model="orderQueryForm" class="demo-form-inline">
      <el-form-item label="订单编号">
        <el-input
          v-model="orderQueryForm.orderNumber"
          placeholder="请输入订单编号"
          clearable
        ></el-input>
      </el-form-item>
      <el-form-item label="会员名字">
        <el-input
          v-model="orderQueryForm.memberName"
          placeholder="请输入会员名字"
          clearable
        ></el-input>
      </el-form-item>
      <el-form-item label="订单状态">
        <el-select v-model="orderQueryForm.refund" placeholder="请选择订单状态">
          <el-option label="正常" value="0"></el-option>
          <el-option label="退款" value="1"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="订单日期">
        <el-date-picker
          v-model="orderQueryForm.time"
          type="daterange"
          align="right"
          unlink-panels
          clearable
          range-separator="至"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
          :picker-options="pickerOptions"
        >
        </el-date-picker>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="sreachOrderQuery">查询</el-button>
      </el-form-item>
    </el-form>

    <el-table
      :data="orderTable"
      border
      style="width: 100%"
      :row-class-name="orderClassName"
    >
      <el-table-column fixed="left" type="index" width="50" label="序列">
      </el-table-column>
      <el-table-column prop="orderNumber" label="订单号" min-width="180">
      </el-table-column>
      <el-table-column prop="allPic" label="商品总金额（元）" min-width="100">
      </el-table-column>
      <el-table-column
        prop="addReceivablePic"
        label="应收金额（元）"
        min-width="100"
      >
      </el-table-column>
      <el-table-column
        prop="receiptsPic"
        label="实收金额（元）"
        min-width="100"
      >
      </el-table-column>
      <el-table-column prop="giveChange" label="找零（元）" min-width="100">
      </el-table-column>
      <el-table-column prop="discount" label="优惠（元）" min-width="100">
      </el-table-column>
      <el-table-column prop="memberName" label="会员名字" min-width="100">
      </el-table-column>
      <el-table-column prop="createTime" label="创建时间" min-width="150">
        <template slot-scope="scope">{{
          scope.row.createTime | datefmt
        }}</template>
      </el-table-column>
      <el-table-column prop="createTime" label="修改时间" min-width="150">
        <template slot-scope="scope">{{
          scope.row.updateTime | datefmt
        }}</template>
      </el-table-column>
      <el-table-column fixed="right" label="操作" width="180">
        <template slot-scope="scope">
          <el-button type="primary" @click="orderDetail(scope.row)"
            >详情</el-button
          >
          <el-button
            type="danger"
            v-if="scope.row.refund != '1'"
            @click="delOrderFun(scope.row)"
            >退款</el-button
          >
        </template>
      </el-table-column>
    </el-table>
    <div style="margin: 30px auto 100px; text-align: center">
      <el-pagination
        background
        :total="total_order"
        :page-sizes="[10, 20, 30, 40, 50, 100]"
        :current-page="pageNo_order"
        @size-change="sizeChange_order"
        layout="total,sizes, prev, pager, next, jumper"
        @current-change="currentChange_order"
      >
      </el-pagination>
    </div>
    <el-dialog
      title="订单详情"
      width="70%"
      custom-class="orderDialogName"
      :visible.sync="dialogTableVisible"
      :close-on-click-modal="false"
    >
      <el-table
        :data="stockData"
        v-loading="tableLoading"
        border
        style="width: 100%"
      >
        <el-table-column type="index" fixed="left" label="序号" width="50">
        </el-table-column>
        <el-table-column fixed="left" label="商品名称" width="150">
          <template slot-scope="scope">{{ scope.row.commodityTitle }}</template>
        </el-table-column>
        <el-table-column
          label="销售方式"
          :filters="[
            { text: '普通', value: '普通' },
            { text: '称重销售', value: '称重销售' },
          ]"
          filter-placement="bottom-end"
          width="100"
        >
          <template slot-scope="scope"
            ><el-tag
              :type="scope.row.salesMethods === '普通' ? 'primary' : 'success'"
              disable-transitions
              >{{ scope.row.salesMethods }}</el-tag
            ></template
          >
        </el-table-column>
        <el-table-column label="商品图片" width="100">
          <template slot-scope="scope">
            <el-image
              style="width: 30px; height: 30px"
              :src="
                'http://127.0.0.1:7001/public/uploadFile/' + scope.row.imgUrl
              "
              alt="暂无图片"
              :preview-src-list="[
                'http://127.0.0.1:7001/public/uploadFile/' + scope.row.imgUrl,
              ]"
            ></el-image>
          </template>
        </el-table-column>
        <el-table-column label="库存数量" width="100">
          <template slot-scope="scope">{{
            scope.row.stock + " （ " + scope.row.commodityUnit + " ）"
          }}</template>
        </el-table-column>
        <el-table-column label="零售价（元）" width="110">
          <template slot-scope="scope">{{
            parseFloat(scope.row.retailPrice).toFixed(2)
          }}</template>
        </el-table-column>
        <el-table-column label="会员价（元）" width="110">
          <template slot-scope="scope">{{
            parseFloat(scope.row.memberPic).toFixed(2)
          }}</template>
        </el-table-column>
        <el-table-column label="进货价（元）" width="110">
          <template slot-scope="scope">{{
            parseFloat(scope.row.buyingPrice).toFixed(2)
          }}</template>
        </el-table-column>
        <el-table-column label="毛利率（%）" width="110">
          <template slot-scope="scope">{{
            scope.row.grossProfitRate
          }}</template>
        </el-table-column>
        <el-table-column label="保质期（天）" width="110">
          <template slot-scope="scope">{{ scope.row.termOfValidity }}</template>
        </el-table-column>
        <el-table-column label="商品规格" width="100">
          <template slot-scope="scope">{{
            scope.row.commoditySpecifications
          }}</template>
        </el-table-column>
        <el-table-column label="商品品牌" width="100">
          <template slot-scope="scope">{{ scope.row.commodityBrand }}</template>
        </el-table-column>
        <el-table-column label="商品分类" width="100">
          <template slot-scope="scope">{{ scope.row.commodityType }}</template>
        </el-table-column>
        <el-table-column label="商品单位" width="80">
          <template slot-scope="scope">{{ scope.row.commodityUnit }}</template>
        </el-table-column>
        <el-table-column
          label="销售方式"
          :filters="[
            { text: '参与', value: '参与' },
            { text: '不参与', value: '不参与' },
          ]"
          filter-placement="bottom-end"
          width="100"
        >
          <template slot-scope="scope"
            ><el-tag
              :type="scope.row.isDiscount === '参与' ? 'success' : 'danger'"
              disable-transitions
              >{{ scope.row.isDiscount }}</el-tag
            ></template
          >
        </el-table-column>
        <el-table-column
          label="参与积分"
          :filters="[
            { text: '是', value: '是' },
            { text: '否', value: '否' },
          ]"
          filter-placement="bottom-end"
          width="100"
        >
          <template slot-scope="scope"
            ><el-tag
              :type="scope.row.isIntegral === '是' ? 'success' : 'danger'"
              disable-transitions
              >{{ scope.row.isIntegral }}</el-tag
            ></template
          >
        </el-table-column>
        <el-table-column label="供货商" width="100">
          <template slot-scope="scope">{{ scope.row.supplier }}</template>
        </el-table-column>
        <el-table-column label="供货商电话" width="100">
          <template slot-scope="scope">{{ scope.row.supplierTel }}</template>
        </el-table-column>

        <el-table-column label="条形码" width="150">
          <template slot-scope="scope">{{ scope.row.code }}</template>
        </el-table-column>
        <el-table-column label="购物券" width="100">
          <template slot-scope="scope">{{
            scope.row.ShoppingVoucher
          }}</template>
        </el-table-column>
        <el-table-column label="备注" width="150">
          <template slot-scope="scope">{{ scope.row.remark }}</template>
        </el-table-column>
      </el-table>
    </el-dialog>
  </div>
</template>
<script>
import {
  searchStock,
  searchOderNumber,
  searchOderNumberStock,
  delOrder,
} from "@/api/commodity";
export default {
  name: "OrderQuery",
  data() {
    return {
      dialogTableVisible: false,
      orderTable: [],
      total_order: 1000,
      pageNo_order: 1, // 当前页数
      pageSize_order: 10,
      tableLoading: false,
      stockData: [], // 获取库存
      orderQueryForm: {
        user: "",
        region: "",
        time: "",
        memberName: "",
        orderNumber: "",
        refund: "0", //订单状态
      },
      pickerOptions: {
        shortcuts: [
          {
            text: "最近一周",
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
              picker.$emit("pick", [start, end]);
            },
          },
          {
            text: "最近一个月",
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
              picker.$emit("pick", [start, end]);
            },
          },
          {
            text: "最近三个月",
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 90);
              picker.$emit("pick", [start, end]);
            },
          },
        ],
      },
    };
  },
  created() {
    // this.getStockList();
    this.getOrderTable(["2021-01-01", "2100-12-12"]);
  },
  methods: {
    orderClassName({ row, rowIndex }) {
      if (row.refund == "1") {
        return "warning-row";
      }
      return "";
    },
    delOrderFun(row) {
      this.$confirm("确定退款吗, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          delOrder({ id: row.id }).then((res) => {
            if (res.code == 0) {
              this.msgSuccess("退款成功");
              this.getOrderTable();
            }
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
    orderDetail(row) {
      searchOderNumberStock({ id: row.id }).then((res) => {
        if (res.code == 0) {
          this.stockData = res.data;
          this.total = res.total;
          this.dialogTableVisible = true;
        }
      });
    },
    getOrderTable(time) {
      if (!time) {
        time = ["2021-01-01", "2100-12-30"];
      }
      const data = {
        orderNumber: this.orderQueryForm.orderNumber.trim(),
        memberName: this.orderQueryForm.memberName.trim(),
        timeStart: this.$moment(time[0]).format("YYYY-MM-DD"),
        timeEnd: this.$moment(time[1]).format("YYYY-MM-DD"),
        refund: this.orderQueryForm.refund,
      };
      searchOderNumber(data).then((res) => {
        console.log(res);
        if (res.code == 0) {
          this.orderTable = res.data;
          this.total_order = res.total;
        }
      });
    },
    orderIndexMethod(indedx) {
      return indedx + 1 + (this.pageNo - 1) * this.pageSize;
    },
    sreachOrderQuery() {
      this.getOrderTable(this.orderQueryForm.time);
    },
    sizeChange_order(size) {
      console.log(size, "改变查询总条数");
      this.pageSize_order = size;
      this.getOrderTable();
    },
    currentChange_order(size) {
      console.log(size, "改变页数");
      this.pageNo_order = size;
      this.getOrderTable();
    },
  },
};
</script>
<style lang="scss">
.el-table .warning-row {
  background: rgb(250, 109, 163);
  color: #fff;
}
.el-table .warning-row:hover {
  color: #000;
}
.orderBox {
  padding: 20px;
}
.orderDialogName {
  height: 70vh;
  overflow: auto;
}
</style>
