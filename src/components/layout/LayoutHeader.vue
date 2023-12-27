<template>
    <div class="header">
        <div class="header-left">
            <el-link type="primary"  :underline="false" @click="routeToIndex">
                    <img class="title"  src="../../../static/picture/logo.png">

            </el-link>
        </div>
        <!-- 左边的 首页 热门 等等 -->
        <div class="header-left">
            <el-link :underline="false" @click="routeToIndex">
                无领导讨论
            </el-link>
            <el-link :underline="false" @click="routeToLiveroom">
                结构化面试
            </el-link>
            <el-link :underline="false" @click="routeToVideoroom">
                视频
            </el-link>

        </div>
        <!-- 中间的 -->
        <div class="header-mid">
            <el-autocomplete
      class="inline-input "
      v-model="input"
      :fetch-suggestions="querySearch"
      placeholder="请输入内容"
      @select="handleSubmit"
    ></el-autocomplete>

      <el-button slot="append" icon="el-icon-search"
       @click="queryStudyRoom"
       >搜索</el-button>
        </div>

        <!-- 右边的 -->
        <div class="header-right">
            <el-link :underline="false" @click="routeToLoginOrRegister">
                登陆注册
            </el-link>
            <el-link :underline="false" @click="routeToPerson">
                个人中心
            </el-link>
            <el-button type="primary" @click="beginLearn">
                开始学习
            </el-button>
        </div>


        <!-- 展示自习室的对话框 -->
        <el-dialog
  title="我的房间"
  :visible.sync="dialogVisible"
  width="30%">
  <div v-for="(item,index) in studyRoomList" :key="index">
    <div class="box">
        <div class="title">房间名称：{{item.studyRoomName}}</div>
        <el-button class="button" type="primary" @click="enterRoom(item.studyRoomId)">进入</el-button>
    </div>
  </div>
</el-dialog>

    </div>
</template>

<script>
import HeaderAvatar from './HeaderAvatar.vue'

export default {
    name: "LayoutHeader",
    components: {
        HeaderAvatar,

    },
    data() {
        return {
            messageNumber: 15, // 右上角消息数量
            input:"",//搜索的内容
            dialogVisible:false,//是否显示自习室对话框
            studyRoomList:[],//用户所在的所有自习室列表
        };
    },
    computed: {},
    methods: {

        toStudyRoom(){
            this.$router.push("/studyRoom")
        },
        // 点击左上角的 注册登录 按钮之后，跳转路由
        routeToLoginOrRegister() {
            this.$router.push("/login");
        },
        // 点击跳转到视频的页面
        routeToVideoroom() {
            this.$router.push("/videoroom");
        },
        // 点击跳转个人中心
        routeToPerson() {
            this.$router.push("/person");
        },
        toggleIsLogin() {
            this.$store.commit("changeIsLogin", !this.$store.state.isLogin);
        },
        // 点击跳转到主页
        routeToIndex() {
            this.$router.push("/index");
        },
        // 点击跳转到直播间页面
        routeToLiveroom() {
            this.$router.push('/liveroom');
        },
        //展示用户的直播间
        getStudyRoom(){
            //获取用户信息
            this.user=JSON.parse(localStorage.getItem('user'));
            this.$axios({
                method:"get",
                url:"/studyRoom/getAllStudyRoomWhichUserIn?userId="+this.user.userId
            })
            .then(res=>{
                this.studyRoomList=res.data.data
                console.log(this.studyRoomList)
            })

        },
        //进入自习室
        enterRoom(studyRoomId){
            this.dialogVisible=false;
            this.$router.push({
                path:"/studyRoom",
                query:{"studyRoomId":studyRoomId}
            })
        },
        //搜索建议
        querySearch(queryString, cb){
            var data=[];
            //发送搜索请求
            this.$axios({
                method:"get",
                url:"/search/autoComplementStudyRoom?keyword="+queryString
            })
            .then(res=>{
                var dataResult=res.data.data;
                var results=[];
                //重新封装数据
                for(let i=0;i<dataResult.length;i++){
                    var tem={
                        name:dataResult[0],
                        value:dataResult[0]
                    }
                    results.push(tem)
                }

            // 控制展示条数，防止数据太大加载时间过长，影响体验
            if(results.length > 10) {
               results.splice(10,results.length-1);
              }
             // 返回筛选出的结果数据到输入框下面的输入列表
              cb(results);
            })


        },
        //选中提交
        handleSubmit(item){
        },
        //查询
        queryStudyRoom(){
            if(this.input==""){
                this.$message.error("搜索内容不能为空")
            }
            else {
            //页面跳转
            this.$router.push({
                path:"/queryStudyRoom",
                query:{inputText:this.input}
            })
        }
        },
        //开始学习
        beginLearn(){
            this.dialogVisible = true;
            //刷新自习室
            this.getStudyRoom();

        }


    },
    mounted(){
        this.getStudyRoom();
        //添加事件监听
    //     document.onkeydown = (e)=> {   //按下回车提交
    //     let key = window.event.keyCode;
    //     //事件中keycode=13为回车事件
    //     if (key == 13) {
    //       this.queryStudyRoom();
    //     }
    //   };

    }
};
</script>

<style scoped>
.header {
    padding-left: 0%;
    padding-right: 10%;
}
.title {
    width: 50px;
    height: 40px;
}
.box{
    display: flex;
}
.el-link {
    font-size: 14px;
}
.header-left {
    float: left;
    width: fit-content;
    margin: 16px 10px 0px 10px;
    vertical-align: middle;
}
.header-right {
    float: right;
    width: fit-content;
    margin: 16px 10px 0px 10px;
}
.header-mid {
    margin-left: 30%;
    float: left;
    width: 30%;

}

</style>

<style>
.searchClass{
    top: 5px;
  border: 1px solid #c5c5c5;
  border-radius: 20px;
  background: #f4f4f4;
}
.searchClass .el-input-group__prepend {
  border: none;
  background-color: transparent;
  padding: 0 10px 0 30px;
}
.searchClass .el-input-group__append {
  border: none;
  background-color: transparent;
}
.searchClass .el-input__inner {
  height: 36px;
  line-height: 36px;
  border: none;
  background-color: transparent;
}
.searchClass .el-icon-search{
  font-size: 16px;
}
.searchClass .centerClass {
  height: 100%;
  line-height: 100%;
  display: inline-block;
  vertical-align: middle;
  text-align: right;
}
.searchClass .line {
  width: 1px;
  height: 26px;
  background-color: #c5c5c5;
  margin-left: 14px;
}
.searchClass:hover{
  border: 1px solid #D5E3E8;
  background: #fff;
}
.searchClass:hover .line {
  background-color: #D5E3E8;
}
.searchClass:hover .el-icon-search{
  color: #409eff;
  font-size: 16px;
}
.box{
    display: flex;
    border-style: hidden;
}
.title{
    font-size: 1.2em;
    margin-bottom: 20px;
}
.button{
    margin-left: 20px;
    height: 40px;
}
</style>
