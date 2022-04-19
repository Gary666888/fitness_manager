<template>
  <div class="addvip">
   <div class="content">
      <!-- 标题 -->
    <div class="title">
      <i class="el-icon-edit"></i>
      <span>修改器材信息</span>

      <!-- 返回按钮 -->
      <div class="back">
        <el-button round size='mini' class="back-btn" icon='el-icon-arrow-left' @click="back">返回</el-button>
      </div>
    </div>
    

    <!-- 表单内容 -->
    <el-form :model="form" class="main" size="medium" status-icon label-width="120px" ref="ruleForm">
      <el-form-item label="器材名称:" prop="name" :rules="{ required: true, message: '标题不能为空', trigger: ['blur','change']}" >
        <el-input v-model="form.name" placeholder="请输入标题"></el-input>
      </el-form-item>
      <el-form-item label="器材数量:" prop="number" :rules="{ required: true, message: '器材数量不能为空', trigger: ['blur','change']}" >
        <el-input v-model="form.number" placeholder="请输入器材数量"></el-input>
      </el-form-item>
      <el-form-item label="器材描述:" prop="info" :rules="{ required: true, message: '内容不能为空', trigger: ['blur','change']}">
        <el-input v-model="form.info" placeholder="请输入公告内容" type="textarea" :autosize="{ minRows: 2, maxRows: 4}"></el-input>
      </el-form-item>
      <el-form-item label="器材图片:" prop="link">
          <el-upload
            name="file"
            class="avatar-uploader"
            action=''
            :show-file-list="false"
            :before-upload="beforeUpload"
            :on-change="handleChange"
            :http-request="uploadFile"
          >
            <img v-if="form.img" :src="this.$store.state.ip+this.form.img" class="avatar" />
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
          </el-upload>
        </el-form-item>

      <el-form-item class="control">
        <el-button type="danger" icon="el-icon-close" @click="back">取消</el-button>
        <el-button @click="resetForm('ruleForm')" class="reset-btn" icon="el-icon-refresh-left">重置</el-button>
        <el-button type="primary" @click="submitForm('ruleForm')" icon="el-icon-document-add">保存</el-button>
      </el-form-item>

      
    </el-form>
   </div>
  </div>
</template>
<script>
 export default {
    data() {
      return {
        file:{},
        form:{},
        data:{},
      }
    },
    methods: {
      //返回
      back(){
        this.$router.push({path:'/home/equipment'});
      },

      // 文件状态改变时的钩子，添加文件、上传成功和上传失败时都会被调用
      handleChange(file) {
        this.file = file.raw
        console.log(this.file);
      },

      // 上传文件之前的钩子
      beforeUpload(file) {
        console.log("上传前");
        const isJPG = file.type === "image/jpeg";
        const isLt2M = file.size / 1024 / 1024 < 2;
        if (!isJPG) {
          this.$message.error("上传头像图片只能是 JPG 格式!");
        }
        if (!isLt2M) {
          this.$message.error("上传头像图片大小不能超过 2MB!");
        }
        return isJPG && isLt2M;
      },

      // 自定义图片上传
      uploadFile() {
        // 创建表单对象
        let form = new FormData();
        // 后端接受参数 ，可以接受多个参数
        form.append("file",this.file)

        // 请求头设置文件上传
        let config = {
            //必须
          headers: {
              "Content-Type": "multipart/form-data"
          },
        }
        // 发送请求
        this.axios.post('/teacher/upload',form,config)
        .then(res=>{
          console.log(res.data)
          if(res.data.code=='200'){
            this.form.img = res.data.url;
          }
        })
        .catch(err=>{
          console.log(err)
        })
      },

      // 格式化数据
      formateData(item){
        var data={};
        data.img=item.eImg;
        data.id=item.eId;
        data.name = item.eName;
        data.number = item.eNum;
        data.info = item.eInfo?item.eInfo:'暂无';
        return data;
      },

      // 提交表单
      submitForm(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            this.$confirm("此操作会将数据永久性修改，确认是否继续操作?", "提示", {
              confirmButtonText: "确定",
              cancelButtonText: "取消",
              type: "warning"
            })
            .then(()=>{
              this.axios.post("/equipment/updateEquipment",{
                eId:this.form.id,
                eName:this.form.name,
                eNum:this.form.number,
                eInfo:this.form.info,
                eImg:this.form.img,
              })
              .then(res=>{
                console.log(res);
                if(res.data.code=='200'){
                  this.$message.success(res.data.msg);
                  this.$router.push({path:'/home/equipment'});
                }
              })
              .catch(err=>{
                console.log(err);
              })
            })
            .catch(()=>{
              this.$message.info("已取消修改");
            })
            
          } else {
            console.log('error submit!!');
            return false;
          }
        });
        
      },

      // 重置表单
      resetForm(formName) {
        this.form.img="";
        this.$refs[formName].resetFields();
      },

    },
    mounted() {//创建时获取数据
      this.axios.post('/equipment/queryEquipment',{
        eId:this.$route.params.id
      })
      .then(res=>{
        console.log(res.data)
        if(res.data.code=='200'){
          this.data = res.data.data;
          this.form=this.formateData(res.data.data)
        }
      });
    },
  }
</script>
<style lang="less">
@import "../assets/less/base.less";

.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: @greenColor;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>

<style lang="less" scoped>
@import "../assets/less/base.less";
.addvip {
  color: @fontColor;
  background-color: white;
  min-height: 500px;
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
.el-input--medium,.el-select,.time-input{
  width: 300px;
}
.reset-btn{
  background: @blueGreenColor;
  color: white;
  border-color: @blueGreenColor;
}
.el-form--inline .control{
  text-align: right;
  width: 100%;
}
</style>