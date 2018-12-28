<template>
  <div class="users">
    <!-- 导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item :to="{ path: '/users' }">用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 搜索 -->
    <el-input placeholder="请输入内容" v-model="query" class="input-with-select">
      <el-button slot="append" icon="el-icon-search" @click="searchUser"></el-button>
    </el-input>
    <el-button type="success" @click="addUser" plain>添加用户</el-button>
    <!-- 表格 -->
    <el-table :data="userList" style="width: 100%">
      <el-table-column label="姓名" width="200" prop="username"></el-table-column>
      <el-table-column label="邮箱" width="200" prop="email"></el-table-column>
      <el-table-column label="电话" width="200" prop="mobile"></el-table-column>
      <el-table-column label="状态" width="200">
        <template slot-scope="scope">
          <el-switch
            @change="changeState(scope.row)"
            v-model="scope.row.mg_state"
            active-color="#13ce66"
            inactive-color="#ff4949"
          ></el-switch>
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button type="primary" size="mini" icon="el-icon-edit" plain circle></el-button>
          <el-button
            type="danger"
            size="mini"
            icon="el-icon-delete"
            plain
            circle
            @click="delItem(scope.row.id)"
          ></el-button>
          <el-button type="success" size="mini" icon="el-icon-check" plain round>分配角色</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="currentPage"
      :page-sizes="[2,4,6,8,10]"
      :page-size="pageSize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
      background
    ></el-pagination>
  </div>
</template>

<script>
import axios from 'axios';
export default {
  data() {
    return {
      query: '',
      userList: [],
      currentPage:1,
      pageSize:2,
      total:0,
      baseurl:"http://localhost:8888/api/private/v1/"
    }
  },
  methods:{
    render(){
      axios({
        method: "get",
        url: this.baseurl+'users',
        params: {
          pagenum:this.currentPage,
          pagesize:this.pageSize,
          query:this.query
        },
        headers: {
          Authorization :localStorage.getItem("token")
        }
      }).then(res=>{
        console.log(res);
        if(res.data.meta.status===200){
        this.userList=res.data.data.users
        this.total = res.data.data.total
        }
      })
    },
    addUser(){

    },
    searchUser(){
      this.currentPage = 1
      this.render()
    },
    changeState(users){
      axios({
        method:"put",
        url:this.baseurl+`users/${users.id}/state/${users.mg_state}`,
        headers: {
          Authorization: localStorage.getItem('token')
        }
      }).then(res=>{
        if(res.data.meta.status===200){
          this.$message.success('修改成功')
          this.render()
        }
      })
    },
    delItem(id){
      this.$confirm('删他?', '？？？', {
        type: 'warning'
      }).then(() => {
          // 发送ajax请求
          axios({
            method: 'delete',
            url: this.baseurl + `users/${id}`,
            headers: {
              Authorization: localStorage.getItem('token')
            }
          }).then(res => {
            if (res.data.meta.status === 200) {
              console.log(res.data);
              // 如果当前就剩一条，还渲染当前，就会有问题，当前页没有数据了
              if (this.userList.length <= 1 && this.currentPage > 1) {
                this.currentPage--
              }
              this.render()
              this.$message.success('删除成功')
            } else {
              this.$message.error('删除失败')
            }
          })
        })
        .catch(() => {
          this.$message.info('取消删除')
        })
    },
    handleSizeChange(value){
      this.pageSize=value
      this.render()
    },
    handleCurrentChange(value){
      this.currentPage=value
      this.render()
    }
  },
    created(){
      this.render()
    }
}
</script>

<style lang="less">
.el-breadcrumb {
  height: 40px;
  line-height: 40px;
}
.input-with-select {
  width: 300px;
  margin-bottom: 5px;
}
</style>
