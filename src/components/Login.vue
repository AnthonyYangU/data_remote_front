<template>
    <div class="login-wrap">
        <div class="ms-login">
            <div class="ms-title">数据采集管理平台</div>

            <div class="myLogin" v-if="registerFlag">
                <el-form :model="ruleForm2" ref="ruleForm2" label-width="0px" class="ms-content">
                    <el-form-item prop="userName">
                        <el-input v-model="ruleForm2.userName" placeholder="用户名">
                            <el-button slot="prepend" icon="el-icon-s-check"></el-button>
                        </el-input>
                    </el-form-item>
                    <el-form-item prop="password">
                        <el-input type="password" placeholder="密码" v-model="ruleForm2.userPwd">
                            <el-button slot="prepend" icon="el-icon-edit-outline"></el-button>
                        </el-input>
                    </el-form-item>
                    <el-row>
                        <el-col :span="14"
                            ><el-input
                                placeholder="验证码"
                                v-model="code"
                                clearable
                                prop="security"
                            >
                            </el-input>
                        </el-col>
                        <el-col :span="10">
                            <div class="code" @click="refreshCode">
                                <security :identifyCode="identifyCode"></security>
                            </div>
                        </el-col>
                    </el-row>            
                    <br>
                    <div class="login-btn">
                        <el-button type="primary" @click="login">登录</el-button>
                        <el-button type="primary" @click="toRegister">注册</el-button>
                        <!-- <el-button type="primary" @click="toRegister">找回密码</el-button> -->
                    </div>
                    <p class="login-tips" v-show="errorTip">用户名或者密码错误</p>
                </el-form>
            </div>

            <div class="ms-register" v-if="!registerFlag">
                <el-form :model="ruleForm" status-icon :rules="rules" ref="ruleForm"  class="ms-content">
                    <el-form-item label="" prop="username">
                        <el-input v-model="ruleForm.username" clearable placeholder="用户邮箱">
                           <el-button slot="prepend" icon="el-icon-s-check"></el-button> 
                        </el-input>
                    </el-form-item>                    
                    <!-- <el-form-item label="" prop="email">
                        <el-input v-model="ruleForm.email" clearable placeholder="邮箱">
                           <el-button slot="prepend" icon="el-icon-eleme"></el-button> 
                        </el-input>
                    </el-form-item> -->
                    <el-form-item label="" prop="pass">
                        <el-input type="password" v-model="ruleForm.pass" autocomplete="off" placeholder="密码" clearable>
                            <el-button slot="prepend" icon="el-icon-s-goods"></el-button> 
                        </el-input>
                    </el-form-item> 
                    <el-form-item label="" prop="checkPass">
                        <el-input type="password" v-model="ruleForm.checkPass" autocomplete="off" placeholder="确认密码" clearable>
                            <el-button slot="prepend" icon="el-icon-goods"></el-button> 
                        </el-input>
                    </el-form-item>
                    <el-row>
                        <el-col :span="14"
                            ><el-input
                                placeholder="验证码"
                                v-model="code2"
                                clearable
                                prop="security"
                            >
                            </el-input>
                        </el-col>
                        <el-col :span="10">
                            <div class="code" @click="refreshCode2">
                                <security :identifyCode="identifyCode2"></security>
                            </div>
                        </el-col>
                    </el-row>            
                    <br>        
                    <div class="login-btn">
                        <el-button type="primary" @click="toLogin">返回登录</el-button>
                        <el-button type="primary" @click="submitForm('ruleForm')">确认注册</el-button>
                    </div>
                    <br>
                </el-form>
            </div>
        </div>
    </div>
</template>

<script>
    import axios from 'axios';
    import security from './security'
    export default {
        data: function(){
            var validatePass = (rule, value, callback) => {
                let reg = /(?!^[0-9]+$)(?!^[A-z]+$)(?!^[^A-z0-9]+$)^[^\s\u4e00-\u9fa5]{6,16}$/;
                if (value === '') {
                    callback(new Error('请输入密码'));
                }else if(!reg.test(value)){
                    callback(new Error('密码长度需6-16位，且包含字母和字符'))
                } else {
                    if (this.ruleForm.checkPass !== '') {
                        this.$refs.ruleForm.validateField('checkPass');
                    }
                    callback();
                }
            };
            var validatePass2 = (rule, value, callback) => {
                if (value === '') {
                    callback(new Error('请再次输入密码'));
                } else if (value !== this.ruleForm.pass) {
                    callback(new Error('两次输入密码不一致!'));
                } else {
                    callback();
                }
            };
            return {
                code: '',
                code2:'',
                // 随机抽取四位数
                identifyCodes: '1234567890abcdef',
                identifyCode: '',
                identifyCode2: '',
                checked: false,
                swiperOptions: {
                    autoplay:{
                        delay:2000,
                    },
                    pagination: {
                        el: '.swiper-pagination'
                    }
                },
                errorTip:false,
                registerFlag:this.$store.state.registerState,
                userExist:false,
                emaiExist:false,
                ruleForm2: {
                    userName: '',
                    userPwd: ''
                },
                ruleForm: {
                    username:'',
                    checkPass: '',
                    pass: '',
                    //email: '847124730@qq.com'
                },
                rules: {
                    pass: [
                        { validator: validatePass, trigger: 'blur' }
                    ],
                    checkPass: [
                        { validator: validatePass2, trigger: 'blur' }
                    ],
                    // email: [
                    //     { validator: checkEmail, trigger: 'blur' }
                    // ]
                },
                // load:false
            }
        },
        components:{
            security
        },
        computed:{
            swiper() {
                return this.$refs.mySwiper.$swiper
            }
        },
        mounted(){
            this.identifyCode = ''
            this.makeCode(this.identifyCodes, 4)
            this.refreshCode2()
            // this.swiper.slideTo(3,1000,false)
        },
        methods: {
            randomNum(min, max) {
                return Math.floor(Math.random() * (max - min) + min)
            },
            // 刷新验证码
            refreshCode() {
                this.identifyCode = ''
                this.makeCode(this.identifyCodes, 4)
            },
            refreshCode2() {
                this.identifyCode2 = ''
                this.makeCode2(this.identifyCodes, 4)
            },
            makeCode(o, l) {
                for (let i = 0; i < l; i++) {
                    this.identifyCode += this.identifyCodes[
                    this.randomNum(0, this.identifyCodes.length)
                    ]
                }
                console.log(this.identifyCode)
            },
            makeCode2(o, l) {
                for (let i = 0; i < l; i++) {
                    this.identifyCode2 += this.identifyCodes[
                    this.randomNum(0, this.identifyCodes.length)
                    ]
                }
                console.log(this.identifyCode2)
            },
            login(){
                
                if(!this.ruleForm2.userName || !this.ruleForm2.userPwd){
                  this.errorTip = true;
                  sessionStorage.setItem("loginstate","1")
                  return;
                }

                if(this.identifyCode != this.code){
                    
                    this.$alert("验证码错误")
                    this.refreshCode() 
                    sessionStorage.setItem("loginstate","1")
                    return
                }
                // console.log(this.ruleForm2)
                // console.log(this.ruleForm2.userName)
                axios.get('/api/userfind',{
                            params: {
                                userName:this.ruleForm.username
                            }
                }).then(response=>{
                    let res = response.data
                    if(res.status==0){
                        axios.post("/api/login",{
                            userName:this.ruleForm2.userName,
                            userPwd:this.ruleForm2.userPwd
                        }).then((response)=>{
                            let res = response.data;
                            console.log(res)
                            if(res.status==0){
                            this.errorTip = false;
                            sessionStorage.setItem("loginstate",0)
                            sessionStorage.setItem("ms_username",this.ruleForm2.userName);
                            this.$router.push('/table');
                            }else{
                            //   this.$store.commit("updateUserSignal",false)
                                sessionStorage.setItem("loginstate","1")
                                this.errorTip = true;
                            }
                        })
                    }
                }).catch(()=>{
                    sessionStorage.setItem("loginstate","1")
                    this.errorTip = true;
                });
                
                
            },
            toRegister(){
                this.registerFlag = false;
            },
            toLogin(){
                this.registerFlag = true;
            },
            //注册
            submitForm(formName) {
                // this.load=true
                this.$refs[formName].validate((valid) => {
                    if(valid){
                        // console.log('code2',this.code2)
                        if(this.identifyCode2 != this.code2){
                            this.$alert("验证码错误")
                            this.refreshCode()
                            return
                        }
                        // console.log(this.ruleForm)
                        axios.get('/api/userfind',{
                            params: {
                                userName:this.ruleForm.username
                            }
                        }).then((response)=>{
                            let res = response.data;
                            console.log(res)
                            if(res.status==1){
                                this.$alert("用户名已经被注册")
                            }else{
                                console.log(this.ruleForm.username)
                                axios.post('/api/register',{
                                    userName:this.ruleForm.username,
                                    userPwd:this.ruleForm.pass
                                }).then((response)=>{
                                    let res = response.data;
                                    // console.log(res)
                                    if(res.status=='0'){
                                        this.$message("注册成功")
                                        //this.toLogin();
                                        // console.log("Register successfully")
                                    }
                                    
                                })
                            }
                        }).finally(()=>{
                            this.refreshCode2()
                        })
                    }else{
                        console.log('error submit!!');
                        return false;
                    }
                });
            }
        }
    }
</script>

<style scoped>
    .login-wrap{
        position: relative;
        width:100%;
        height:100%;
        background-image: url(../assets/img/train3.jpg);
        background-size: 100%;
    }

    .ms-title{
        width:100%;
        line-height: 50px;
        text-align: center;
        font-size:20px;
        font-weight: 700;
        color: rgba(65, 131, 215, 1);
        border-bottom: 1px solid #ddd;
    }
    .ms-login{
        position: absolute;
        float:left;
        left:50%;
        top:50%;
        width:350px;
        margin:-190px 0 0 -175px;
        border-radius: 5px;
        background: rgba(255,255,255, 0.3);
        overflow: hidden;
    }
    .ms-content{
        padding: 30px 30px;
    }
    .login-btn{
        text-align: center;
    }
    .login-tips{
        font-size:12px;
        text-align: center;
        line-height:30px;
        color:rgb(211, 19, 19);
    }
</style>