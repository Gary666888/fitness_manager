<template>
  <div id="show-house">
    <div class="content" v-loading="loading">

      <!-- 标题 -->
      <div class="title">
        <i class="el-icon-s-management"></i>
        <span>教练信息详情</span>

        <!-- 返回按钮 -->
        <div class="back">
          <el-button round size='mini' class="back-btn" icon='el-icon-arrow-left' @click="back">返回</el-button>
        </div>
      </div>

      <!-- 显示信息 -->
      <div class="main" v-show="data" v-loading="loading">
        <el-row :gutter="10">
          <el-col :xs="24" :sm="6" :md="3" :lg="3"><div class="item-title">头像:</div></el-col>
          <el-col :xs="24" :sm="18" :md="9" :lg="9"><div>
              <el-avatar shape="square" :size="100" :src="data.img" class="serve-img"></el-avatar>
          </div></el-col>
          <el-col :xs="24" :sm="6" :md="3" :lg="3"><div class="item-title">教练编号:</div></el-col>
          <el-col :xs="24" :sm="18" :md="9" :lg="9"><div class="msg">{{data.no}}</div></el-col>
        </el-row>

        <el-row :gutter="10">
          <el-col :xs="24" :sm="6" :md="3" :lg="3" ><div class="item-title">名字:</div></el-col>
          <el-col :xs="24" :sm="18" :md="9" :lg="9" ><div class="msg">{{data.name}}</div></el-col>
          <el-col :xs="24" :sm="6" :md="3" :lg="3" ><div class="item-title">电话:</div></el-col>
          <el-col :xs="24" :sm="18" :md="9" :lg="9" ><div class="msg">{{data.tel}}</div></el-col>
        </el-row>

        <el-row :gutter="10">
          <el-col :xs="24" :sm="6" :md="3" :lg="3" ><div class="item-title">性别:</div></el-col>
          <el-col :xs="24" :sm="18" :md="9" :lg="9" ><div class="msg">{{data.sex}}</div></el-col>
          <el-col :xs="24" :sm="6" :md="3" :lg="3" ><div class="item-title">年龄:</div></el-col>
          <el-col :xs="24" :sm="18" :md="9" :lg="9" ><div class="msg">{{data.age}}</div></el-col>
        </el-row>
        <el-row :gutter="10">
          <el-col :xs="24" :sm="6" :md="3" :lg="3" ><div class="item-title">生日:</div></el-col>
          <el-col :xs="24" :sm="18" :md="9" :lg="9" ><div class="msg">{{data.birth}}</div></el-col>
          <el-col :xs="24" :sm="6" :md="3" :lg="3" ><div class="item-title">个人描述:</div></el-col>
          <el-col :xs="24" :sm="18" :md="9" :lg="9" ><div class="msg">{{data.info}}</div></el-col>
        </el-row>
      </div>
    </div>
  </div>
</template>
<script>

export default {
  data() {
    return { 
      value:3.7 ,
      loading:true,
      data: {},//信息    
    };
  },
  methods: {
    //返回信息列表
    back(){
      this.$router.push({path:'/home/teacher'});
    },

    // 格式化时间
    switchTimeFormat(time) {
      const dateTime = new Date(time);
      const year = dateTime.getFullYear();
      const month = dateTime.getMonth() + 1;
      const date = dateTime.getDate();
      return `${year}-${this.PrefixInteger(month,2)}-${this.PrefixInteger(date,2)}`;
    },

    //格式化数字
    PrefixInteger(num, n) {
      return (Array(n).join(0) + num).slice(-n);
    },

    // 通过生日获取年龄
    GetAge(strBirthday){       
      var returnAge,
      strBirthdayArr=strBirthday.split("-"),
      birthYear = strBirthdayArr[0],
      birthMonth = strBirthdayArr[1],
      birthDay = strBirthdayArr[2],  
      d = new Date(),
      nowYear = d.getFullYear(),
      nowMonth = d.getMonth() + 1,
      nowDay = d.getDate();   
      if(nowYear == birthYear){
        returnAge = 0;//同年 则为0周岁
      }
      else{
        var ageDiff = nowYear - birthYear ; //年之差
        if(ageDiff > 0){
          if(nowMonth == birthMonth) {
            var dayDiff = nowDay - birthDay;//日之差
            if(dayDiff < 0) {
              returnAge = ageDiff - 1;
            }else {
              returnAge = ageDiff;
            }
          }else {
            var monthDiff = nowMonth - birthMonth;//月之差
            if(monthDiff < 0) {
              returnAge = ageDiff - 1;
            }
            else {
              returnAge = ageDiff ;
            }
          }
        }else {
          returnAge = -1;//返回-1 表示出生日期输入错误 晚于今天
        }
      } 
      return returnAge;//返回周岁年龄
    },

    // 格式化数据
    formateData(item){
      var data={};
      data.id=item.tId;
      data.no=this.PrefixInteger(item.tId,10);
      data.img = item.tImg?this.$store.state.ip+item.tImg:'/img/avator.jpg';
      data.name = item.tName;
      data.tel = item.tTel;
      data.birth = item.tBirth?this.switchTimeFormat(item.tBirth):'待完善';
      data.age =  item.tBirth?this.GetAge(data.birth):'待完善';
      data.sex = item.tSex?'女':'男';
      data.info = item.t_info?item.t_info:"待完善";
      console.log(data);
  
      return data;
    }
  },
  mounted() {//创建时获取数据
    console.log(this.$route.params.id);
    this.axios.post("/teacher/queryTeacher", {
      tId:this.$route.params.id
    })
    .then((res) => {
      console.log(res);
      if(res.data.code=='200'){
        this.loading=false;
        this.data=this.formateData(res.data.data)
      }
    })
    .catch(err=> {
      console.log(err)
    })
  },
};
</script>
<style lang="less">
.el-avatar>img{
  height:100%;
  width: 100%;
}
</style>
<style lang="less" scoped>
@import "../assets/less/base.less";
#show-house {
  color: @fontColor;
  background-color: white;
  height: 100%;
}
.msg{
  height: 40px;
}
.content {
  background: white;
}
.title {
  padding: 15px 20px;
  border-top: 3px solid #e7eaec;
  border-bottom: 1px solid #e7eaec;
  i {
    margin-right: 2px;
  }
  .back {
    float: right;
    .back-btn{
      padding: 5px;
    }
  }
}
.main{
  padding: 20px;
}
.choose .el-button--text{
  margin-top: 10px;
  color: #A7B1C2;
  padding: 15px 20px;
}
.btn-choose.el-button--text{
  color: @fontColor;
  border: 1px solid @lineColor;
}
.tab-content{
  border: 1px solid @lineColor;
}
.item-title{
  color: @fontColor;
  font-weight: 600;
  text-align: right;
  line-height: 45px;
  font-size: 14px;
}
.msg{
  color: @fontColor;
  line-height: 45px;
  font-size: 14px;
}
@media screen and (max-width: 768px) {
  .item-title{
    text-align: left;
  }
}
.el-tabs--border-card>.el-tabs__content{
  padding: 30px;
}
.add-btn{
  text-align: right;
  padding: 5px 20px;
  padding-top:20px;
  margin-top: 30px;
  border-top: 1px solid @lineColor;
}
.searchTel-btn{
  background: @blueColor;
  border-color: @blueColor;
}
.rate{
  height: 45px;
  align-items: center;
  display: flex;
}
</style>