<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/index' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>会员管理</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 表格 -->
    <el-table :data="tableData" style="width: 100%" row-key="id">
      <!-- 每一行数据都是对象， -->
      <!-- 列表 
        prop 指定渲染的数据对象 key
      -->
      <el-table-column label="昵称" width="180" prop="nickname"></el-table-column>
      <el-table-column label="手机号" width="180" prop="phone"></el-table-column>
      <el-table-column label="注册日前" width="180" prop="addtime"></el-table-column>
      <el-table-column label="操作">
        <!-- 作用域插槽 -->
        <template slot-scope="scope">
          <!-- scoped.row ：数据对象 -->
          <el-button size="mini" @click="handleEdit(scope.row)">编辑</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 弹窗 -->

    <el-dialog title="会员编辑" :visible.sync="isShow">
      <!-- 表单 -->
      <el-form :model="form" :rules="rules" label-width="100px" ref="form">
        <el-form-item label="昵称">
          <el-input v-model="form.nickname"></el-input>
        </el-form-item>
        <el-form-item label="手机号">
          <el-input v-model="form.phone"></el-input>
        </el-form-item>
        <el-form-item label="密码">
          <el-input v-model="form.password" placeholder="请输入设置密码"></el-input>
        </el-form-item>
        <el-form-item label="状态">
          <el-switch v-model="form.status"></el-switch>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="isShow = false">取 消</el-button>
        <el-button type="primary" @click="submitFn">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import qs from "qs";
let defaultData = {
  uid: "",
  nickname: "",
  phone: "",
  password: "",
  addtime: "",
  status: true,
};
export default {
  data() {
    return {
      tableData: [],
      isShow: false,
      roleList: [],
      form: {
        ...defaultData,
      },
      rules: {},
      page: 1,
      size: 6,
      total: 0,
      addtime: "",
    };
  },
  mounted() {
    this.getTableData();
  },
  methods: {
    currentChange(page) {
      // console.log(page)
      this.page = page;
      this.getTableData();
    },
    // 确定按钮
    submitFn() {
      // 状态修改为number值
      this.form.status = this.form.status ? 1 : 2;
      console.log(this.form);
      this.$refs.form.validate(async (valid) => {
        if (valid) {
          let res = await this.$http.post("/api/memberedit", this.form);
          console.log(res);
          if (res.code == 200) {
            this.$message.success("操作成功");
            this.getTableData();
            this.isShow = false;
          } else if (res.code == 500) {
            this.$message.error(res.msg);
          }
        }
      });
    },
    // 获取数据
    async getTableData() {
      let res = await this.$http.get("/api/memberlist", {
        params: { page: this.page, size: this.size },
      });

      console.log(res);
      if (res.code == 200) {
        //遍历addtime，格式化时间戳
        res.list.map(item=>{
                    item.addtime = this.setTime(JSON.parse(item.addtime))
                })
        this.tableData = res.list;
      }
    },
    // 格式化时间戳
    setTime(addtime) {
      let date = new Date(addtime);
      let y = date.getFullYear();
      let m = date.getMonth() + 1;
      let d = date.getDate();
      let h = date.getHours() < 10 ? "0" + date.getHours() : date.getHours();
      let mi = date.getMinutes() < 10 ? "0" + date.getMinutes() : date.getMinutes();
      let s = date.getSeconds() < 10 ? "0" + date.getSeconds() : date.getSeconds();
      return `${y}-${m}-${d} ${h}:${mi}:${s}`;
    },
    // 编辑
    async handleEdit(row) {
      this.isAdd = false;
      let res = await this.$http.get("/api/memberinfo", {
        params: { uid: row.uid },
      });
      console.log(res);
      if (res.code == 200) {
        this.isShow = true;
        res.list.status = res.list.status == 1 ? true : false;
        this.form = {
          ...res.list,
          password: "",
          uid: row.uid, //编辑时需要id,后台返回数据没有id,自行添加
        };
      } else if (res.code == 500) {
        this.$message.error(res.msg);
      }
    },
  },
};
</script>
