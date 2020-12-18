<template>
  <div class="User">
    <div class="port-head">
      <div class="head-text">
        <span>管理员设置</span>
      </div>
      <div class="right-container">
        <el-button @click="userEdit"><i class="el-icon-refresh"></i></el-button>
        <el-button @click="deleteUser"> <i class="el-icon-delete"></i></el-button>
        <el-button @click="addDialog = true"><i class="el-icon-plus"></i></el-button>
      </div>
    </div>

  <div class="user-table">
    <el-table
      :data="tableData"
      border
      align="center">

      <el-table-column
        width="55px"
        label="序号">
        <template slot-scope="scope" @click="getRowData(scope)">
          <el-checkbox @change="getRowData(scope)" name="user">{{scope.$index+1}}</el-checkbox>
        </template>
      </el-table-column>

      <el-table-column
        v-for="(value,key) in tableHead"
        :prop="key"
        :label="value"
        :key="key"
        >
      </el-table-column>

    </el-table>


    <el-dialog
      title= "title"
      :visible.sync="addDialog"
      width="25%"
      center
      >
      <el-form
        label-position="right"
        label-width="80px"
        :model="addFrom"
        size="mini"
        :rules="rule"
        ref="userFrom">

          <el-form-item label="角色" prop="Role">
            <el-select v-model="addFrom.Role" placeholder="请选择角色">
              <el-option label="管理员" value="管理员"></el-option>
              <el-option label="操作员" value="操作员"></el-option>
            </el-select>
          </el-form-item>

          <el-form-item label="用户名" prop="UserName">
            <el-input v-model="addFrom.UserName" style="width: 60%"></el-input>
          </el-form-item>

          <el-form-item label="密码" prop="PassWord" v-if="this.editEvent!=true">
            <el-input v-model="addFrom.PassWord" show-password ref="password" style="width: 60%"></el-input>
          </el-form-item>

          <el-form-item label="确认密码" prop="ConfirmPW" v-if="this.editEvent!=true">
            <el-input v-model="addFrom.ConfirmPW" show-password ref="ConfirmPW" style="width: 60%"></el-input>
          </el-form-item>

          <el-form-item label="名字" prop="Name">
            <el-input v-model="addFrom.Name"  style="width: 60%"></el-input>
          </el-form-item>

          <el-form-item label="部门" prop="Department">
            <el-input v-model="addFrom.Department" style="width: 60%"></el-input>
          </el-form-item>

          <el-form-item label="职位" prop="Position">
            <el-input v-model="addFrom.Position"  style="width: 60%"></el-input>
          </el-form-item>

          <el-form-item label="电话" prop="Phone">
            <el-input v-model="addFrom.Phone"  style="width: 60%"></el-input>
          </el-form-item>

          <el-form-item label="电子邮箱" prop="Email">
            <el-input v-model="addFrom.Email"  style="width: 60%"></el-input>
          </el-form-item>

      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="cancelFrom">取 消</el-button>
        <el-button type="primary" @click="submitForm('userFrom')">确 定</el-button>
      </span>
    </el-dialog>
  </div>




  </div>


</template>

<script>
export default {
  name: "UserManagement",

  activated(){
    const role = window.sessionStorage.getItem('role');
    if(role=='操作员'){
      this.$message({
        type:'warning',
        message:'没有访问权限'
      });
      this.$router.push('/home');
      return
    }
  },
  mounted() {
    this.reload()
  },
  data () {
    return {
      tableData:[],
      tableHead: {
        Role:'角色',
        UserName:'用户名',
        Name:'名字',
        Department:'部门',
        Position:'职位',
        Phone:'电话',
        Email:'电子邮箱',
      },
      rowData: {
        Action:0,
      },
      addDialog:false,
      addFrom:{
        Action:0,
        Role:'',
        UserName:'',
        Name:'',
        Department:'',
        Position:'',
        Phone:'',
        Email:'',
        PassWord:'',
        ConfirmPW:'',
      },
      rule:{
        Role:[
          { required: true, message: '请输入角色', trigger: 'change' },
        ],
        UserName:[
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 6, max: 18, message: '长度在 6 到 18 个字符 ', trigger: 'blur' },
        ],
        Name:[
          { required: true, message: '请输入名字', trigger: 'blur' },
        ],
        Department:[
          { required: true, message: '请输入部门', trigger: 'blur' },
        ],
        Position:[
          { required: true, message: '请输入职位', trigger: 'blur' },
        ],
        Phone:[
          { required: true, message: '请输入电话', trigger: 'blur' },
        ],
        PassWord:[
          { required: true, message: '请输入密码', trigger: 'blur' },
        ],
        ConfirmPW:[
          { required: true, message: '请再次确认密码', trigger: 'blur' },
        ],

      },
      editEvent:false
    };
  },
  computed:{
    title:function (){
      if(this.rowData.Action==0){
        return '修改管理员信息';
      }else {
        return '添加管理员'
      }
    },
  },
  methods:{

    reload(){
      let param={
        url:'/cgi-bin/getuser.cgi',
        method:'post',
        bodyData:{
          Page:1,
          Role:'超级管理员'
        }
      };
      this.$http.requestPost(param).then((result)=>{
        this.tableData=result.data[1];
        this.addDialog = false;
      })
    },

    deleteUser(){
      this.rowData.Action=2;
      let param={
        url:'/cgi-bin/setuser.cgi',
        method:'post',
        bodyData:this.rowData,
      };
      this.$http.requestPost(param).then((result)=>{
        result.data.State=='success'?this.reload():this.$message({
          message:'用户删除失败',
          type:'warning'
        });
      })
    },

    getRowData(scope){
      Object.assign(this.rowData,scope.row);
    },

    addUser(){
      if(this.addFrom.PassWord===this.addFrom.ConfirmPW){
        delete this.addFrom.ConfirmPW;
      }else {
        return this.$message({
          message:'两次输入密码不一致',
          type:'warning'
        });
      }
      this.addFrom.Action=0
      let param={
        url:'/cgi-bin/setuser.cgi',
        method:'post',
        bodyData:this.addFrom
      };
      this.$http.requestPost(param).then((result)=>{
        if(result.data.State=='success'){
          this.reload();
          this.$message({
            type:'success',
            message:'用户创建成功'
          })
        }else if(result.data.State=='fail'){
          this.$message({
            message: result.data.Info,
            type:'warning'
          });
        }
        this.addDialog=false;
      })
    },

    submitForm(formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          this.editEvent==true?this.submitEdit():this.addUser()
        } else {
          // if(this.editEvent==true){
          //   this.editEvent=false;
          // }
          return false;
        }
      });
    },

    cancelFrom(){
      //关闭dialog时判断editEvent是否在true，是则改为false
      if(this.editEvent==true){
        this.addDialog=false;
        this.editEvent=false;
        return
      }else {
        return this.addDialog=false;
      }
    },

    userEdit(e){
      //转换为数组判断勾选个数,一次只能编辑一个用户
      // console.log(this.rowData);
      this.addFrom=JSON.parse(JSON.stringify(this.rowData));
      // console.log(this.addFrom)
      const userGrounp=Array.from(document.querySelectorAll('input[name="user"]'));
      let userVal= [];
      let checkOne=false;
      userGrounp.forEach(item=>{
        if(item.checked){
          checkOne = true;
          userVal.push(item)
        }
      });
      if(userVal.length==1){
        this.addDialog = true;
        this.editEvent=true;
      }else if(userVal==0){
        this.$message({
          message:'请勾选用户再进行编辑!',
          type:'warning'
        })
      }else{
        return this.$message({
          type:'warning',
          message:'一次只能编辑一个用户!'
        })
      }
    },

    submitEdit(){
      this.addFrom.OriginName=JSON.parse(JSON.stringify(this.addFrom.UserName));
      this.addFrom.Action=1;
      let param={
        url:'/cgi-bin/setuser.cgi',
        method:'post',
        bodyData:this.addFrom,
      };
      console.log(param)
      this.addDialog=false;
      this.$http.requestPost(param).then((result)=>{
        if(result.data.State=='success'){
          this.reload();
          this.editEvent=false;
        }
      })
    }
  }

}
</script>

<style scoped>
  .port-head{
    display: flex;
    flex: 1;
    justify-content: space-between;
    height: 80px;
    line-height: 80px;
    margin-top: 10px;
    border-top: 2px solid #EBEEF5;
    border-bottom: 2px solid #EBEEF5;
  }

  .head-text{
    height: 40px;
    line-height: 80px;
    text-align: center;
    font-size: 22px;
  }
</style>
