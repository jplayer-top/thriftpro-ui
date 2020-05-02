<template>
  <div class="hello">
    <el-container>
      <el-header style="background: #5CACEE;">

        <div class="header_top">
          <span style="width: 90%;color: #FFFFFF;">服务器监控系统</span>
          <img style="width: 2rem; height: 2rem;" src="../assets/logo.png" />
        </div>
      </el-header>
      <el-main>
        <el-card class="box-card">
          <div slot="header" class="clearfix">
            <span style="font-size: large;">客户端列表</span>
          </div>
          <!-- <el-card v-for="o in engints" :key="o.engkey" class="text item">
            <div class="span_item" v-on:click="toNext(o)"> {{'客户端： ' + o.engkey }}
              <el-button class="btn_item">查看详情</el-button>
            </div>
          </el-card> -->
          <el-table :data="engints" border style="width: 100%">
            <el-table-column fixed prop="num" label="序号" width="50">
            </el-table-column>
            <el-table-column prop="engkey" label="客户端机器码">
            </el-table-column>
            <el-table-column prop="cpuing" label="CPU空闲率">
              <template slot-scope="scope">
                <el-button type="success" size="mini"> {{scope.row.cpuing}}</el-button>
              </template>
            </el-table-column>
            <el-table-column prop="memie" label="内存空闲率">
            </el-table-column>
            <el-table-column prop="diskie" label="磁盘">
            </el-table-column>
            <el-table-column prop="createTime" label="最近时间">
            </el-table-column>
            <el-table-column label="操作" width="100">
              <template slot-scope="scope">
                <el-button type="primary" icon="el-icon-edit" size="small" @click="toNext(scope.row)"> 查看</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-card>


      </el-main>
    </el-container>

  </div>
</template>

<script>
  import axios from 'axios'

  export default {
    name: 'HelloWorld',
    data() {
      return {
        engints: [],
        msg: 'Welcome to Your Vue.js App'
      }
    },
    created() {
      let self = this;
      // 为给定 ID 的 user 创建请求
      axios.post('/api/engint/getAllEngint')
        .then(function(response) {
          self.engints = response.data;
        })
        .catch(function(error) {
          console.log(error);
        });
    },
    methods: {
      toNext(event) {
        //带参数跳转
        this.$router.push({
          path: '/infos',
          query: {
            engkey: event.engkey
          }
        });
      }
    }
  }
</script>

<style>
  .header_top {
    display: flex;
    height: 100%;
    justify-content: flex-start;
    align-items: center;
    font-size: large;
  }

  .btn_item {
    margin-left: 10px;
  }

  .span_item {
    justify-content: center;
    justify-items: center;
    width: 100%;
  }

  .hello {}

  .text {
    font-size: 14px;
  }

  .item {
    justify-content: center;
    display: flex;
    margin-bottom: 18px;
  }

  .box-card {
    width: 100%;
    height: 90%;

  }
</style>
