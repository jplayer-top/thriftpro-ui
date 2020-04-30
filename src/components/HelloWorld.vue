<template>
  <div class="hello">
    <el-card class="box-card">
      <div slot="header" class="clearfix">
        <span>所有客户端</span>
      </div>
      <el-card v-for="o in engints" :key="o.engkey" class="text item">
        <div class="span_item" v-on:click="toNext(o)"> {{'客户端： ' + o.engkey }}
            <el-button class="btn_item">查看详情</el-button>
        </div>
      </el-card>
    </el-card>
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
        .then(function (response) {
          self.engints = response.data;
        })
        .catch(function (error) {
          console.log(error);
        });
    },
    methods: {
      toNext(event) {
        //带参数跳转
        this.$router.push({path:'/infos',query:{engkey:event.engkey}});
      }
    }
  }
</script>

<style>
  .btn_item {
    margin-left: 50px;
  }

  .span_item {
    justify-content: center;
    justify-items: center;
    width: 100%;
  }

  .hello {

  }

  .text {
    font-size: 14px;
  }

  .item {
    justify-content: center;
    display: flex;
    margin-bottom: 18px;
  }

  .clearfix:before,
  .clearfix:after {
    display: table;
    content: "";
  }

  .clearfix:after {
    clear: both
  }

  .box-card {
    margin-top: 50px;
    margin-left: 10%;
    width: 80%;
    height: 60%;

  }
</style>
