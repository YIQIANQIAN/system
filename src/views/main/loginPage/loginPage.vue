<template>
  <!--登录页面-->
  <div id="login" :style="{ backgroundImage: 'url(' + img1 + ')' }">
    <!--背景图片不能在css中设置，否则可能导致打包图片失败-->
    <div id="loginDiv" :style="{ backgroundImage: 'url(' + img2 + ')' }">
      <a href="http://www.samsph.com/hsxx/1092/1/"><img id="schoolImg" src="../../../assets/images/title.png" :alt="imgAlt" ></a>
      <p id="hospitalMottoP">厚德 至善 求精 图强</p>
      <p id="loginTitleP">用户登录</p>
      <span id="loginTipSpan">不清楚登录规则？请点击<span id="tipSpan" @click="modal1 = true">这里</span>查看</span>
      <div id="userNumDiv" class="inputDiv">
        <span class="inputSpan">帐号</span><br/>
        <input id="userNumInput" class="loginInput" type="text" v-model="userId">
      </div>
      <div id="PasswordDiv" class="inputDiv">
        <span class="inputSpan">密码</span><br/>
        <input id="passwordInput" class="loginInput" type="password" v-model="passwordValue" @keyup.enter="loginClick()">
      </div>
      <p></p>
      <p></p>
      <div>
        <button id="loginButton" @click="loginClick()" class="am-btn am-btn-success am-radius">登录</button>
        <span id="forgetSpan" @click="forgetPwClick">忘记密码？</span>
      </div>
    </div>
    <Modal
        v-model="modal"
        width="400"
        :mask-closable="false"
        id="modalBody"
        :styles="{top:'10rem'}">
      <div style="font-size: 1.1rem;text-align: center;">
        <p>{{ errorMessage }}</p>
      </div>
      <div slot="footer" style="text-align: center">
        <button id="modalBtn" @click="modal = false">确定</button>
      </div>
    </Modal>
    <Modal
        v-model="modal1"
        width="400"
        id="modalBody">
      <div slot="header" style="font-size: 1rem;text-align: center;padding: 0.5rem 0;" id="modalHeader">
        <span>登录规则</span>
      </div>
      <div style="font-size: 0.9rem;text-align: left;">
        <p>123456</p>
        <p>123456</p>
        <p>123456</p>
      </div>
      <div slot="footer" style="text-align: center">
        <button id="modalBtn" @click="ok1">确定</button>
        <button id="modalBtn" @click="cancel1">取消</button>
      </div>
    </Modal>
  </div>
</template>

<script>
    var CryptoJS = require("crypto-js");
    export default {
        name: 'login',
        data () {
            return {
              imgAlt: '四川省医科科学院·四川省人民医院·护士学校',//学校图标
              img1: require('../../../assets/images/login-background.jpg'),//页面背景图片
              img2: require('./images/loginDiv.jpg'),//登录背景图片
              userId: '',//用户名
              passwordValue: '',//用户密码
              modal1: false,//登录规则
              modal: false, //错误提示对话框
              errorMessage: ""//错误提示
            }
        },
        beforeMount: function () {
          if(sessionStorage.getItem("userType") == "1"){
            location.href = '#/login/main/studentHome';
          }else if(sessionStorage.getItem("userType") != null){
            location.href = '#/login/main/eduAdminHome';
          }
        },
        mounted: function() {
          window.onresize = function(){
            var clientWidth = document.body.clientWidth;
            if(clientWidth > 319 && clientWidth < 1025){
              document.documentElement.style.fontSize = "24px";
            }
          };
          window.onresize();
          var dom = document.getElementById("login");
          dom.style.height = window.innerHeight + "px";
        },
        methods: {
          ok1: function(){
            this.modal1 = false;
          },
          cancel1: function(){
            this.modal1 = false;
          },
          loginClick: function(){
//            登录验证
            if(this.userNumValue == "" || this.passwordValue == "" ) {
//              this.$Message.warning("帐号或密码不能为空！");
              this.errorMessage = "帐号或密码不能为空!";
              this.modal = true;
            }else{
              var a = CryptoJS.MD5(this.passwordValue + this.userId + "护士学校");
              a = a.toString().toUpperCase();
              function encrypt(data) {
                var key  = CryptoJS.enc.Latin1.parse('uestc2017nurse01');
                var iv   = CryptoJS.enc.Latin1.parse('10esrun7102ctseu');
                return CryptoJS.AES.encrypt(data, key, {iv:iv,mode:CryptoJS.mode.CBC,padding:CryptoJS.pad.ZeroPadding}).toString();
              }
//              this.$http.post('../testPhp/loginCheck.php', {
              this.$http.post('./login', {
                "userId": this.userId,
                "loginPwd": JSON.stringify(encrypt(a))
              }, {
                "Content-Type": "application/json"
              }).then(function (response) {
                console.log(response.body);
                if(response.body.result == "1"){
                  sessionStorage.setItem("userType", response.body.userType);
                  if(response.body.userType == "1"){
                    location.href = '#/login/main/studentHome';
                  }else{
                    location.href = '#/login/main/eduAdminHome';
                  }
                }else{
//                  this.$Message.error("帐号或密码有误，请确认重试！");
                  this.errorMessage = "帐号或密码有误，请确认重试！";
                  this.modal = true;
                }
              }, function (error) {
                this.$Message.error('连接失败，请重试！',3);
              });
            }
          },
          forgetPwClick: function(){
//            忘记密码
            location.href='#/login/operation/forgetPassword';
          }
        }
    }
</script>

<style scoped>
    #login{
      /*页面*/
      /*text-align: center;*/
      font-family: 微软雅黑;
      background-repeat: no-repeat;
      background-size: cover;
      display: flex;
      min-height: 45.9rem;
      align-items: center;
      justify-content: center;
    }
    #loginDiv{
      /*登录模块*/
      width: 50%;
      height: 70%;
      min-height: 27rem;
      background-repeat: no-repeat;
      background-size: cover;
      box-shadow: 0 0 0.5rem;
      display: flex;
      flex-direction: column;
      padding: 2rem 3rem;
      text-align: left;
    }
    #schoolImg{
      /*学校图标*/
      width: 23rem;
      height: 3rem;
    }
    #hospitalMottoP{
      /*医训*/
      color: #34B064;
    }
    #loginTitleP{
      /*用户登录*/
      font-size: 1.4rem;
    }
    #loginTipSpan{
      /*规则查看*/
      color: #727272;
      font-size: 0.8rem;
      position: relative;
      bottom: 1rem;
    }
    .inputDiv{
      /*帐号密码div*/
      border: none;
      text-align: left;
      height: 4rem;
      width: 15rem;
    }
    .loginInput{
      /*帐号密码输入框*/
      outline: none;
      border: thin solid #CECECE;
      height: 2.5rem;
      width: 15rem;
      font-size: 1rem;
    }
    #tipSpan{
      /*规则查看*/
      color: #00A64A;
    }
    #tipSpan:hover{
      cursor: pointer;
    }
    .inputSpan{
      /*帐号密码*/
      position: relative;
      top: 0.7rem;
      left: 1.5rem;
      background-color: white;
    }
    #loginButton{
      /*登录按钮*/
      min-width: 5rem;
      min-height: 2.3rem;
      width: 8rem;
    }
    #loginButton:hover{
      cursor: pointer;
    }
    #loginButton:active{
      background-color: green;
    }
    #forgetSpan{
      /*忘记密码*/
      font-size: 0.75rem;
      position: relative;
      left: 1rem;
    }
    #forgetSpan:hover{
      cursor: pointer;
    }
    @media screen and (min-width:320px) and (max-width:1025px) {
      #schoolImg{
        width: 19rem;
        height: 2.5rem;
        position: relative;
        right: 2.5rem;
      }
      #login{
        align-items: flex-start;
      }
      #loginDiv {
        margin-top: 3rem;
        background-size: 100% 100%;
        width: 75%;
        height: 50%;
      }
    }
</style>
