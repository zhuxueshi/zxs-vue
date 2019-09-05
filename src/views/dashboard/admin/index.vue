<template>
  <div class="dashboard-editor-container">
    <github-corner class="github-corner" />
    <el-button type="primary" @click.native.prevent="sayHiToNginx">sayHiToNginx</el-button>
    <el-button type="primary" @click.native.prevent="sayHiToLocal">sayHiToLocal</el-button>
    <span>{{users}}</span>
    <panel-group @handleSetLineChartData="handleSetLineChartData" :groupData="xxxData"/>

    <el-row style="background:#fff;padding:16px 16px 0;margin-bottom:32px;">
      <line-chart :chart-data="lineChartData" />
    </el-row>

    <el-row :gutter="32">
      <el-col :xs="24" :sm="24" :lg="8">
        <div class="chart-wrapper">
          <raddar-chart />
        </div>
      </el-col>
      <el-col :xs="24" :sm="24" :lg="8">
        <div class="chart-wrapper">
          <pie-chart />
        </div>
      </el-col>
      <el-col :xs="24" :sm="24" :lg="8">
        <div class="chart-wrapper">
          <bar-chart />
        </div>
      </el-col>
    </el-row>

    <el-row :gutter="8">
      <el-col :xs="{span: 24}" :sm="{span: 24}" :md="{span: 24}" :lg="{span: 12}" :xl="{span: 12}" style="padding-right:8px;margin-bottom:30px;">
        <transaction-table />
      </el-col>
      <el-col :xs="{span: 24}" :sm="{span: 12}" :md="{span: 12}" :lg="{span: 6}" :xl="{span: 6}" style="margin-bottom:30px;">
        <todo-list />
      </el-col>
      <el-col :xs="{span: 24}" :sm="{span: 12}" :md="{span: 12}" :lg="{span: 6}" :xl="{span: 6}" style="margin-bottom:30px;">
        <box-card />
      </el-col>
    </el-row>
  </div>
</template>

<script>
import GithubCorner from '@/components/GithubCorner'
import PanelGroup from './components/PanelGroup'
import LineChart from './components/LineChart'
import RaddarChart from './components/RaddarChart'
import PieChart from './components/PieChart'
import BarChart from './components/BarChart'
import TransactionTable from './components/TransactionTable'
import TodoList from './components/TodoList' 
import BoxCard from './components/BoxCard'
import { toLocal , toNginx } from '@/api/user'

const lineChartData = {
  newVisitis: {
    expectedData: [100, 120, 161, 134, 105, 160, 165],
    actualData: [120, 82, 91, 154, 162, 140, 145]
  },
  messages: {
    expectedData: [200, 192, 120, 144, 160, 130, 140],
    actualData: [180, 160, 151, 106, 145, 150, 130]
  },
  purchases: {
    expectedData: [80, 100, 121, 104, 105, 90, 100],
    actualData: [120, 90, 100, 138, 142, 130, 130]
  },
  shoppings: {
    expectedData: [130, 140, 141, 142, 145, 150, 160],
    actualData: [120, 82, 91, 154, 162, 140, 130]
  }
}

export default {
  name: 'DashboardAdmin',
  components: {
    GithubCorner,
    PanelGroup,
    LineChart,
    RaddarChart,
    PieChart,
    BarChart,
    TransactionTable,
    TodoList,
    BoxCard
  },
  data() {
    return {
      lineChartData: lineChartData.newVisitis,
      xxxData:{
        visitsCount:1,
        messagesCount:2,
        purchasesCount:3,
        shoppingsCount:4
      },
      users:''
    }
  },
  methods: {
    handleSetLineChartData(type) {
      this.lineChartData = lineChartData[type]
    },
    sayHiToNginx:function(){
      toNginx().then(res=>{
        this.users=JSON.stringify(res.list);
        console.log(res);
      });
      this.$message({
        message:'sayHiToNginx',
        type:'success'
      });
    },
    sayHiToLocal:function(){
      toLocal().then(res=>{
        this.users=JSON.stringify(res.list);
        console.log(res);
      });
      this.$message({
        message:'sayHiToLocal',
        type:'success'
      });
    },
    websocket:function() {
    const param = {};
    param.name='sendWebsocket';
    console.log('请求成功！');
    sendWebsocket(param).then(res=>{
        console.log('请求成功！');
        console.log(res);
        this.message = res.content;
        this.$message({
            message: "sendWebsocket请求成功！",
            type: "success"
        });
      }).catch(err => {
          console.log(err);
      });
    },
    updatePanlGroup:function(data){
      this.xxxData.visitsCount = data.expectedData[0];
      this.xxxData.messagesCount = data.expectedData[1];
      this.xxxData.purchasesCount = data.expectedData[2];
      this.xxxData.shoppingsCount = data.expectedData[3];
    },
    initWebSocket: function () {
        this.websock = new WebSocket("ws://192.168.1.104:8484/springlcoud/websocket");
        this.websock.onopen = this.websocketonopen;
        this.websock.onerror = this.websocketonerror;
        this.websock.onmessage = this.websocketonmessage;
        this.websock.onclose = this.websocketclose;
    },websocketonopen: function () {
      console.log("WebSocket连接成功");
    },
    websocketonerror: function (e) {
      console.log("WebSocket连接发生错误");
    },
    websocketonmessage: function (e) {
      console.log("websocketonmessage");
      var da = JSON.parse(e.data);
      console.log(da.expectedData);
      console.log(da.actualData);
      lineChartData.expectedData = da.expectedData;
      lineChartData.actualData = da.actualData;
      this.lineChartData = lineChartData;
      this.updatePanlGroup(this.lineChartData);
    },
    websocketclose: function (e) {
      console.log("connection closed (" + e.code + ")");
    }
  },
  created() {
    this.initWebSocket();
  },
  destroyed: function () {
    this.websocketclose();
  }
}
</script>

<style lang="scss" scoped>
.dashboard-editor-container {
  padding: 32px;
  background-color: rgb(240, 242, 245);
  position: relative;

  .github-corner {
    position: absolute;
    top: 0px;
    border: 0;
    right: 0;
  }

  .chart-wrapper {
    background: #fff;
    padding: 16px 16px 0;
    margin-bottom: 32px;
  }
}
</style>
