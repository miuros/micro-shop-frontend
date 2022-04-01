<template>
<div class="outer-wrapper">

    <div class="table">

      <div v-for="(item,index) in toList" :key="index" @click="selectUser(item.name)" class="item">
        {{item.name}}
      </div>
    </div>

    <div class="wrapper">
        <h3>聊</h3>
    
      <div class="message-panel">
        <msg-box v-for="(item, index) of msgList" :key="index" :uname="item.userName" :content="item.content" :isself="item.userUuid==info.uuid"></msg-box>

      </div>
      <div class="input-area">
          <textarea class="input" v-model="msg" @keyup.enter="sendMsg"></textarea>
          <!-- <button class="send" @click="sendMsg">发送</button> -->
          <el-button class="send-btn" @click="sendMsg">发送</el-button>

      </div>
    </div>
  </div>
</template>

<script>
import msgBox from './msgbox.vue';

export default {
    
  name: 'chat',
//   props: {
//     msg: String
//   }
  data() {
    return {
      ws: null,
      toUserUuid: '',
      page:1,
      limit:10,
      content: 'hahhahaha',
      userName: '',
      info:{},
      class: new Map(),
      toList: new Array(),
      msg: '',
      msgList: [],
      num:0,
    }
  },
  components: {
      msgBox,   
  },
  created(){
      if(!localStorage.getItem('token')){
        this.notifyError("login plz"); 
        return;

      }
      this.initWs();
       // disable onclose handler first
      this.$axios.get("/api/nt/v1/list?type=chat&page="+this.page+"&limit="+this.limit).then(res=>{
        if (res.data.code==='200'){

          this.msgList=res.data.data.data.reverse();
          for(var i=0;i<this.msgList.length;i++){
            if (this.msgList[i].toUserUuid==this.info.uuid) {
                if(!this.class[this.msgList[i].userUuid]){
                  this.toList.push({name:this.msgList[i].userUuid});
                  this.class[this.msgList[i].userUuid]=new Array();
                }
                this.class[this.msgList[i].userUuid].push(this.msgList[i]);
            }else{
              if(!this.class[this.msgList[i].toUserUuid]){
                  this.toList.push({name:this.msgList[i].toUserUuid});
                  this.class[this.msgList[i].toUserUuid]=new Array();
                }
                this.class[this.msgList[i].toUserUuid].push(this.msgList[i]);
            }
          }
          console.log(this.class);
          this.msgList=[];
          this.num=res.data.data.num;
        }
      }) 
      
  },
  activated() {
    if (this.$route.query.id != undefined) {
      this.toUserUuid = this.$route.query.id;
      if(this.class[this.toUserUuid]){
        this.msgList=this.class[this.toUserUuid];
      }else{
        this.toList.push(this.toUserUuid);
      }
    }
  },
  mounted() {
      this.info = JSON.parse(localStorage.getItem('user'));

  },
  methods: {
    initWs(){
      this.ws=new WebSocket('ws://miuros.fun:10000/api/chat/v1/chat?token='+localStorage.getItem('token'));

      this.ws.onerror=function(err){
        console.log(err);
      }
      this.ws.onclose = function () {
        this.notifySucceed("chat close");
        this.ws.close();
      };
      this.ws.onmessage=this.message;
    },
    selectUser(id){
      this.toUserUuid=id;
      this.msgList=this.class[id];
    },
    handleSelect(key, keyPath) {
        console.log(key,keyPath);
    }, 
    message(data) {
      data=data.data;
      
        console.log(data)
        // this.msgList.push(`${data.name}说:${data.msg}`)
        //this.msgList.push()
        if(data.toUserUuid==this.info.userUuid){
            if(!this.class[data.userUuid]){
              this.toList.push({name:data.userUuid});
              this.class[data.userUuid]=new Array();
            }
          this.class[data.userUuid].push(data);
        }else{
            if(!this.class[data.toUserUuid]){
              this.toList.push({name:data.toUserUuid});
              this.class[data.toUserUuid]=new Array();
            }
          this.class[data.toUserUuid].push(data);
        }
        this.msgList.push(JSON.parse(data));
    },
    sendMsg() {
        // $socket is socket.io-client instance
        const data = {
          type: 'chat',
          toUserUuid: this.toUserUuid,
          content: this.msg,
        }
        this.msg="";
        this.ws.send(JSON.stringify(data));
    }
    },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.table{
  top: 100px;
  position: relative;
  float: left;
  width: 200px;
  left: 300px;

}
.item{
  text-align: center;
  width: 200px;
  border: 0 ;
  border-radius: 4px;
  background-color: antiquewhite;
}
    .outer-wrapper {
      width :100%;
      height: 100%;
      padding: 0;
      margin: 0;
     }
    .wrapper {
        position: relative;
        width: 650px;
        background-color: #fff;
        opacity: 0.85;
        height: 610px;
        border-radius: 4px;
        border: 1px #ebebeb solid;
        box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
        margin: 0 auto;
        display: flex;
        top: 80px; 
        flex-direction: column;
        align-items: center;
        padding-top: 10px;
    }
    .message-panel {
        width: 85%;
        height: 350px;
        border-top: 1px #ebebeb solid;
        border-bottom: 1px #ebebeb solid;
        overflow: scroll;
        overflow-x: hidden;
        padding: 10px;
    }
    .input-area {
        width: 85%;
        border-radius: 4px;
        height: 80px;
        margin-top: 20px;
        border:0 solid black;
    }
    .send-btn {
        position: absolute;
        right: 10px;
        bottom: 10px;
    }
    .input {
        width: 100%;
        height: 100%;
        border:  0px #ebebeb solid;
        border-radius: 4px;
        outline: none;
        padding: 5px;
    }
    .fix-top {
      position: fixed;
      width: 100%;
      height: 50px;
      top: 0;
    }
  
</style> 