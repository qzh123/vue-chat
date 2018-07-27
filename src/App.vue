<template>
  <div id="app">
    <el-container class="chatroom">
      <el-aside width="200px">
        <div class="card">
          <div class="header">
              <img class="avatar" src="http://tva2.sinaimg.cn/crop.0.0.1022.1022.50/005Fj2RDgw1ex5pinpg65j30sg0sgdg9.jpg" v-bind:alt="account">
              <p class="name">{{ account }} </p>
              <div class="dot" v-bind:class="[ online ? 'dot-green' : 'dot-red' ]"></div>
          </div>
          <li v-for="(room,index) of rooms" :key="index" v-bind:id="user.id" @click="changeChat('room',room)">
              <img v-bind:src="room.avatar" v-bind:alt="room.name">
              <p>{{ room.name }} <span>({{ room.memberCount }})</span></p>
              <div v-bind:class="[ room.has_message ? 'dot' : '' ]"></div>
          </li>
          <li v-for="(user,index) of users" :key="index" v-bind:id="user.name" @click="changeChat('user',user)">
              <img v-bind:src="user.avatar" v-bind:alt="user.name">
              <p>{{ user.name }}
              <div v-bind:class="[ user.has_message ? 'dot' : '' ]"></div>
          </li>
        </div>
      </el-aside>
      <el-container>
        <el-header height="10%">
            <!-- <el-dropdown @command="changeRoom">
              <span class="el-dropdown-link">
                选择房间<i class="el-icon-arrow-down el-icon--right"></i>
              </span>
              <el-dropdown-menu slot="dropdown">
                <el-dropdown-item v-for="(room,index) of rooms" :key="index" command="room.id">{{ room.name }}</el-dropdown-item>
              </el-dropdown-menu>
            </el-dropdown> -->
        </el-header>
        <el-main class="message" height="65%">
          <ul>
              <div v-for="(item,index) of msgs" :key="index">
                <li v-if="item.msgType == 'joinRoom'">
                    <p>{{item.time}}</p>
                    <div class="text">{{ item.msg }}</div>
                </li>
                <li v-else-if="item.msgType == 'outRoom'">
                    <p>{{item.time}}</p>
                    <div class="text">{{ item.msg }}</div>
                </li>
                <li v-else>
                    <p>{{item.time}}</p>
                    <div class="msg" v-bind:class="item.from == account ? 'self' : ''">
                        <img src="http://tva2.sinaimg.cn/crop.0.0.1022.1022.50/005Fj2RDgw1ex5pinpg65j30sg0sgdg9.jpg" alt="item.from">
                        <div class="text">{{ item.msg }}</div>
                    </div>
                </li>
              </div>
          </ul>
        </el-main>
        <el-footer height="25%">
          <div class="user-text">
              <textarea name="" id="" placeholder="Press enter to send" v-model="msg" v-on:keyup.enter="send"></textarea>
          </div>
          <div class="text-footer">
              <button v-on:click="send">发送</button>
          </div>
        </el-footer>
      </el-container>
    </el-container>
    
    <el-dialog
      title="登录"
      :visible.sync="showDialog"
      width="30%"
      center>
      <el-input v-model="account" placeholder="请输入账号"></el-input>
      <el-input v-model="passwd" placeholder="请输入密码"></el-input>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="login">登录</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
var moment = require("moment");
export default {
  name: 'App',
  components: {},
  data() {
    return {
      msg: "",
      msgs: {
        room:[],
        user:[],
      },
      rooms: [
        {
          id: 1000,
          chatname: '群聊A',
          avatar: 'http://58pic.ooopic.com/58pic/12/25/04/02k58PICVwf.jpg',
          has_message: false,
          memberCount: 0,
          has_join: false
        },
        {
          id: 1001,
          chatname: '群聊B',
          avatar: 'http://58pic.ooopic.com/58pic/12/25/04/02k58PICVwf.jpg',
          has_message: false,
          memberCount: 0,
          has_join: false
        }
      ],
      users: [],
      currentChat : {
        chatType: null,
        chatObj: null
      },
      clientNum: 0,
      showEmoji: false,
      emoji: [],
      showDialog: false,
      account: '',
      passwd: '',
      roomId: 0,
      count: 0,
      online: false
    };
  },
  mounted: function() {
    this.initWebpack()
  },
  updated: function() {
    this.$nextTick(function() {
      var oBody = document.querySelector(".message")
      oBody.scrollTop = oBody.scrollHeight
    })
  },
  methods: {
    initWebpack(){//初始化websocket
      const wsuri = "ws://localhost:9501"
      this.websock = new WebSocket(wsuri)//这里面的this都指向vue
      this.websock.onopen = this.websocketopen
      this.websock.onmessage = this.websocketonmessage
      this.websock.onclose = this.websocketclose
      this.websock.onerror = this.websocketerror
    },
    websocketopen(){//打开
      console.log('链接成功')
      // 检测用户是否登录
      if (this.account != '') {
        this.login()
      } else {
        // 用户未登录，提醒用户登录并跳转到登录页面
        this.showDialog = true;
      }
      
    },
    websocketonmessage(e){ //数据接收
      console.log(e)
      let res = e.data==''?undefined:JSON.parse(e.data)
      if (res == undefined){

      }else{
        let data = res.data
        let status = res.status
        if(status == 200){
          switch(data.msgType){
            case 'login':
              console.log('login')
              this.$message(data.msg)
              this.showDialog = false
              this.online = true;
              break
            case 'joinRoom':
              console.log('joinRoom')
              data.time = moment().format("HH:mm:ss")
              this.msgs.room.push(data)
              break
            case 'roomChat':
              console.log('roomChat')
              data.time = moment().format("HH:mm:ss")
              this.msgs.room.push(data)
              break
            case 'userChat':
              console.log('userChat')
              data.time = moment().format("HH:mm:ss")
              this.msgs.user.push(data)
              break
            default:
              break
          }
        }else{
          this.$message.error(data.msg)
        }
      }
    },
    websocketclose(){  //关闭
      console.log("WebSocket关闭")
    },
    websocketerror(e){  //失败
      console.log(e)
      console.log("WebSocket连接失败")
    },
    login(){
      // 用户已登录，客户端发送上线信息给服务器
      let user = {
        "controller": "Index",   // 请求控制器
        "action": "login",   // 请求方法
        "data":{    // 请求参数
            "account": this.account,
            "passwd": this.passwd
        }
      }
      let data = JSON.stringify(user)
      this.websock.send(data)
    },
    // changeRoom(roomId) {
    //   this.roomId = roomId
    //   this.$message('您选择了' + roomId)
    //   this.joinRoom()
    // },
    send() {
      if(this.msg != ''){
        if(this.currentChat.chatType == null){
          alert('请选择一个发送对象')
        }else if(this.currentChat.chatType=='room'){
          let info = {
            "controller": "Index",   // 请求控制器
            "action": "sendToRoom",   // 请求方法
            "data":{    // 请求参数
                "message": this.msg,
                "from": this.account,
                "roomId": this.currentChat.chatObj.id
            }
          }
        }else{
          let info = {
            "controller": "Index",   // 请求控制器
            "action": "sendToRoom",   // 请求方法
            "data":{    // 请求参数
                "message": this.msg,
                "from": this.account,
                "to": this.currentChat.chatObj.account
            }
          }
        }
        let data = JSON.stringify(info)
        this.websock.send(data)
        this.msg = ''
      } else {
        alert('消息为空！')
      }
    },
    changeChat(type,chatObj) {
      this.currentChat.chatType = type
      this.currentChat.chatObj = chatObj
      if(type == 'room'){
        if(!chatObj.has_join){
          this.joinRoom(chatObj.id)
        }
      }
    },
    joinRoom(roomId){
      let info = {
        "controller": "Index",   // 请求控制器
        "action": "intoRoom",   // 请求方法
        "data":{    // 请求参数
            "roomId": roomId,
            "account": this.account
        }
      }
      let data = JSON.stringify(info)
      this.websock.send(data) 
    },
  }
}
</script>

<style lang="less">

body{
    background: url('./assets/bg.jpg');
    background-size: 100%;
    font-family: Helvetica, sans-serif;
    color: #4d4d4d;
    margin: 0;
    padding: 0;
}

#app{
    width: 800px;
    height: 600px;
    position: absolute;
    top: 50%;
    left: 50%;
    margin-left: -400px;
    margin-top: -300px;

    display: flex;
    flex-wrap: nowrap;
    flex-direction: row;
    border-radius: 3px;
    

    .slide-bar{
      width: 200px;
      background:#2e3238;
      color: #f4f4f4;
    }
    .main{
      width: 600px;
      background: #eee;
      display: flex;
      flex-direction: column;
    }
}

.card{
  display: flex;
  flex-direction: column;
  padding: 20px 15px;
  border-bottom: 1px solid #24272c;

  .header{
      display: flex;
      flex-direction: row;

      img{
          width: 40px; 
          height: 40px;
      }

      p{
          font-size: 16px;
          align-self:center;
          margin-left: 15px;
      }
      .dot{
          width: 8px;
          height: 8px;
          border-radius: 50%;
          align-self: center;
          margin-left: 10px;
          background: #eee;
      }
      .dot-green{
          background: #00ff00;
      }
      .dot-red{
          background: #ff0000;
      }
  }
}

.list{
  height: 479px;
  overflow-y: scroll;
  overflow-x: hidden;
  &::-webkit-scrollbar-button{
    display: none;
  }
  &::-webkit-scrollbar{  
    width: 8px;  
    background-color: #4d4d4d;  
  }  
    
  /*定义滚动条轨道 内阴影+圆角*/  
  &::-webkit-scrollbar-track{   
    background-color: #2e3238;  
  }  
    
  /*定义滑块 内阴影+圆角*/  
  &::-webkit-scrollbar-thumb{  
    border-radius: 10px;  
    background: rgba(255,255,255,0.1);
  }  

  ul{
    margin: 0; padding: 0;
  }
  li{
    display: flex;
    flex-direction: row;
    align-items: center;
    height: 60px;
    cursor: pointer;
    border-bottom: 1px solid #292c33;
    padding: 15px;
    &:hover{
        background: rgba(255,255,255,0.03);
    }
    &.active{
        background: rgba(255,255,255,0.1);
    }
    
    img{
        width: 30px; 
        height: 30px;
    }   
    p{
        margin-left: 15px;
        font-size: 16px;
    }
    .dot{
        width: 8px;
        height: 8px;
        border-radius: 50%;
        align-self: center;
        margin-left: 10px;
        background: #ff0000;
    }
  }
}

.message{
  overflow-y: scroll;
  padding: 10px 15px;
  height: 400px;
  &::-webkit-scrollbar-button{
      display: none;
  }
  &::-webkit-scrollbar{  
      width: 8px;  
      background-color: #eee;  
  }  
    
  /*定义滚动条轨道 内阴影+圆角*/  
  &::-webkit-scrollbar-track{   
      background-color: #eee;  
  }  
    
  /*定义滑块 内阴影+圆角*/  
  &::-webkit-scrollbar-thumb{  
      border-radius: 10px;  
      background: #ccc;
  }  

  li{
      display: flex;
      flex-direction: column;
      margin-bottom: 15px;

      p{
          align-self: center;
          padding: 0 18px;
          font-size: 12px;
          color: #fff;
          border-radius: 2px;
          background-color: #dcdcdc;
          margin: 5px 0px;
      }

      .msg{
          display: flex;
          flex-direction: row;


          img{
              width: 30px;
              height: 30px;
              margin-right: 10px;
              flex-shrink: 0;
          }
          .text{
              padding: 10px;
              min-height: 20px;
              font-size: 14px;
              text-align: left;
              word-break: break-all;
              background: #fafafa;
              border-radius: 4px;
              line-height: 1.4;
          }
      }
      .self{
          flex-direction: row-reverse;
          .text{
              background: #b2e281;
          }
          img{
              margin-right: 0px; margin-left: 10px;
          }
      }
  }
}

.el-header {
  background-color: cadetblue;
  margin: 0 0;
  height: 20%;
}

.user-text{
    height: 100px;
    border-top: 1px solid #ddd;

    textarea{
        resize: none;
        width: 99%;
        height: 100%;
        border: none;
        outline: none;
    }
}
.text-footer{
    display: flex;
    padding: 10px 20px;
    background: #fff;
    justify-content: flex-end;
    align-items: flex-end;
    button{
        background: #fff;
        padding: 3px 20px;
        color: #222;
        border: 1px solid #c1c1c1;
        border-radius: 3px;
    }
    span{
        font-size: 14px;
        color: #999;
        margin-right: 10px;
    }
}

.el-footer {
  background-color: #B3C0D1;
  color: #333;
  text-align: center;
  height: 30%;
  padding: 0 0;
}

.el-aside {
  background-color: #D3DCE6;
  color: #333;
  text-align: center;
}

.el-main {
  background-color: #E9EEF3;
  color: #333;
  text-align: center;
  height: 50%;
}

body > .el-container {
  margin-bottom: 40px;
}
</style>
