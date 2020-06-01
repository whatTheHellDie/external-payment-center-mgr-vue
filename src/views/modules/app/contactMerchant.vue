<template>
  <el-dialog class="my-dialog mechant-dialog" title="联系商家" :close-on-click-modal="false" :visible.sync="dialogVisible" :append-to-body="true">
      <div class="withdraw-coin-wrap withdraw-customize">
  <div class="order-info">
    <div class="content">
      <div class="number">订单号：{{chat.order_no}}</div>
      <div class="number">订单金额：{{amount}}</div>
    </div>
    <div>
      <ul class="record" id="record">
        <li class="more" v-show="more" @click="getMoreOrder">加载更多</li>
        <li v-for="item in beforeMessages">
          <div class="status" v-if="item.all==1">{{item.content}}</div>
          <div v-else-if="item.user_id!=uid">
            <div class="time">{{item.add_time}}</div>
            <div class="avatar">{{chat.user.r_name}}</div>
            <div class="detail" v-if='item.content.slice(0,4)=="img["&&item.content.slice(item.content.length-6)=="][lay]"'><img :src="getBiliImg(item)" width="100%" /></div>
            <div v-else class="detail">{{item.content}}
            </div>
            <div class="ke"></div>
          </div>
          <div class="other mine" v-else-if="item.user_id==uid">
            <div class="ke"></div>
            <div class="time">{{item.add_time}}</div>
            <div class="detail" v-if='item.content.slice(0,4)=="img["&&item.content.slice(item.content.length-6)=="][lay]"'>
              <img :src="getBiliImg(item)" width="100%" />
            </div>
            <div v-else class="detail">{{item.content}}

            </div>
            <div class="avatar">{{chat.my.r_name}}</div>
          </div>
        </li>
        <li v-for="item in messages">
          <div class="status" v-if="item.all==1">{{item.content}}</div>
          <div v-else-if="item.user_id!=uid">
            <div class="time">{{item.add_time}}</div>
            <div class="avatar">{{chat.user.r_name}}</div>
            <div class="detail" v-if='item.content.slice(0,4)=="img["&&item.content.slice(item.content.length-6)=="][lay]"'><img :src="getBiliImg(item)" width="100%" /></div>
            <div v-else class="detail">{{item.content}}
            </div>
            <div class="ke"></div>
          </div>
          <div class="other mine" v-else-if="item.user_id==uid">
            <div class="ke"></div>
            <div class="time">{{item.add_time}}</div>
            <div class="detail" v-if='item.content.slice(0,4)=="img["&&item.content.slice(item.content.length-6)=="][lay]"'>
              <img :src="getBiliImg(item)" width="100%" />
            </div>
            <div v-else class="detail">{{item.content}}

            </div>
            <div class="avatar">{{chat.my.r_name}}</div>
          </div>
        </li>
      </ul>
      <div class="send-box">
        <textarea @keyup.enter="sendMsg" class="input" v-model="inputText" ref="inputBox" @keyup="toBottom" placeholder="请输入消息"></textarea>
        <!--<input type="text" />-->
        <div class="send" @click="sendMsg">发送</div>
        <img src="/static/img/upload.png" alt="" class="send1" />
        <input @change='add_img($event,0)' accept="image/png,image/jpeg,image/gif" type="file" id="saveImage" class="send1 opacity">
      </div>
    </div>
  </div>
  </div>
  </el-dialog>
</template>

<script>
  import io from 'socket.io-client'
  export default {
    name: 'Chat',
    data() {
      return {
        dialogVisible:true,
        showMessage: false,
        message: '',
        more: false,
        beforeMessages: [],
        messages: [],
        amount: '',
        uid: "",
        chat: {
          'my': {
            'id': '',
            'logo': '',
            'r_name': ''
          },
          'user': {
            'id': '',
            'logo': '',
            'r_name': ''
          },
          'order_id': '',
          'order_no': '',
          'id': 0,
          'user_id': '',
          'status': 0
        },
        getChatTime: 0,
        inputText: '',
        location: '',
        onlineTip: '',
        isShowTip: false,
        imgs: ['/static/img/zhan1.png'],
        imgSource: "",
        isReturn: false,
        applicationId:''
      }
    },


    methods: {
      init(params){
        this.beforeMessages=[]
        this.messages=[]
        this.more=false;
        this.applicationId=''
        
        
        this.dialogVisible=true
        if(params.chat) {
        var chatInfo = params.chat;

        var chatInfoStorage = {}
        window.socket = io(chatInfo.source, {
          forceNew: true
        });
        console.log(window.socket)
        //path: '/',
        this.amount = chatInfo.amount
        this.uid = this.chat.my.id = chatInfo.my_id //1842
        this.chat.my.r_name = chatInfo.my_rname

        this.chat.user_id = chatInfo.my_id //1842

        this.chat.user.r_name = chatInfo.to_rname
        this.chat.user.id = chatInfo.to_id //31

        this.chat.order_id = chatInfo.trade_id
        this.chat.order_no = chatInfo.trade_no
        this.imgSource = chatInfo.biliexUrl
        this.connectSocket()
        this.applicationId=params.applicationId
      } 
      },
      connectSocket(){
        // 建立socket.io通信
      window.socket.on('connection', (data) => {});
      window.socket.on('order_info', (data) => {});

      window.socket.on('order', (data) => {
        this.isReturn = true
        if(data.length < 10) {
          this.more = false;
        } else {
          this.more = true;
        }
        var id = this.chat.id;
        if(id > 0) {
          for(let i = 9; i >= 0; i--) {
            if(data[i]) {
              this.beforeMessages.unshift(data[i])
            }
          }
        } else {
          this.messages = data;
          this.toBottom();
        }
        this.chat.id = data[0].id;
        //              if(this.chat.my.id == data[0].to_user_id && this.chat.order_id == data[0].order_id){
        //                  var read_msg = {order_id:this.chat.order_id,to_user_id:data[0].to_user_id,data:{user2_read:1},order_no:this.chat.order_no};
        //                  window.socket.emit('read', read_msg);
        //              }
        this.saveMessages(data)

        this.sendOrder()
      });
      window.socket.on('chat', (data) => {
        //判断是否在聊天界面，

        if(typeof this.chat !== 'undefined') {
          if(this.chat.order_id == data.order_id) {
            this.messages.push(data)
            this.saveMessages(this.messages)
            this.toBottom()
            //                  tpl($('#chat_msg').html()).render({content: [data], chat: chat}, function (string) {
            //                      if(chat.my.id == data.to_user_id && chat.order_id == data.order_id){
            //                          var read_msg = {order_id:chat.order_id,to_user_id:data.to_user_id,data:{user2_read:1},order_no:chat.order_no};
            //                          socket.emit('read', read_msg);
            //                      }
            //
            //                      $('#chating').append(string);
            //                      setTimeout(function () {
            //                          var h = $('#chating').height() - 360;
            //                          $('#chating').parent().scrollTop(h);
            //                      }, 1)
            //                  });
          } else {
            this.sendOrder()
          }

        } else {
          this.sendOrder();
          //showTips(data.order_id);
        }
        this.toBottom()
      });
      window.socket.on('unread', (msg) => {
        //          var user_id = <?php echo $user['id'] ?>;
        //          if(msg['to_user_id'] == user_id){
        //              var href = '<?php echo OTC;?>'+ds.url({'s':'OtcOrders.infoView',id:msg['last_order_id']});
        //              $('#otc_unread_href').attr('href',href);
        //              $('#otc_unread_number').text(msg['count']);
        //          }
      });
      window.socket.on('online', (msg) => {
        //          var $this = $('.online-icon[data-id=' + msg.uid + ']');
        //          $this.find('p').addClass('off');
        //          if (msg.online) {
        //              $this.find('p').removeClass('off');
        //          }
      });
      // 后端推送来在线数据时
      window.socket.on('update_online_count', (online_stat) => {
        this.online();
        // $('#online_box').html(online_stat);
      });
      window.socket.on('login', (d) => {
        function isEmptyObject(e) {
          var t;
          for(t in e)
            return !1;
          return !0
        }
        //            orders = {};
        if(typeof this.chat !== 'undefined') {
          setTimeout(() => {
            window.socket.emit("order", this.chat)
          }, 100);
        } else {
          this.sendOrder();
        }

        this.online();
        //checkOrders();
      });

      window.socket.on('connect', () => {
        window.socket.emit('login', this.uid);

      });
      //    window.socket.emit("connect")
      //    window.socket.emit("order_info", '1514')
      //    window.orderInfoInterval=setInterval(()=>{
      //      if(!this.isReturn){
      //        window.socket.emit('order_info', this.uid);
      //        window.socket.emit('order_info', this.uid);
      //      }else{
      //        clearInterval(window.orderInfoInterval)
      //      }
      //    },1000)

      //    window.socket.emit("online", this.uid)
      this.toBottom()
      },
      add_img(event, index) {
        var img1 = event.target.files[0]

        if(index == 0 && this.imgs.length > 0) {
          this.imgs.splice(0, 1)
        }

        if(index == 0) {
          let x = document.getElementById('saveImage').files[0];
          let params = new FormData()
          params.append('applicationId', this.applicationId)
          params.append('file', x)
          this.$axios.post(this.$http.adornUrl('/paychatmsg/uploadImage'),
          params, {
            headers: {
              "Content-Type": "multipart/form-data",
              'token': this.$cookie.get('token')
            }
          }).then(({
            data
          }) => {
            console.log(data)
            if(data.code == 200) {
              var url = data.data.url;
              //            var cutIndex = url.indexOf("\/", 8)
              //            url = url.substr(cutIndex)
              this.inputText = "img[" + url + "][lay]"
              this.sendMsg()
              //            if (data && data.code === '0000') {
              //              this.form.idCardFrontPicUrl = data.data
              //            } else {
              //              // this.$message.error(data.msg)
              //              this.$message({ message: '上传失败', type: 'error' });
              //            }
            }else {
              this.$alert(data.msg, '提示', {
          confirmButtonText: '确定',
        });
            }
          }).catch(({
            error
          }) => {

          })
        }
      },
      getBiliImg(item) {

        //      return this.imgSource + item.content.slice(4, item.content.length - 6)
        return item.content.slice(4, item.content.length - 6)
      },
      saveMessages(data) {},
      sendOrder() {
        window.socket.emit('order_info', this.uid);
      },
      getMoreOrder() {
        window.socket.emit('order', this.chat);
      },
      online() {
        if(window.socket) {
          window.socket.emit('online', this.uid)
        }
      },
      toBottom() {
        this.$nextTick(() => {
          var defaultTop = document.body.scrollTop
          document.documentElement.scrollTop = document.body.scrollTop = document.documentElement.scrollHeight + document.body.scrollHeight
          //      alert(document.documentElement.scrollTop)
          //      document.querySelector("#record").scrollTop=document.querySelector("#record").scrollHeight
        })
      },
      sendMsg() {
        if(!this.inputText) {
          return
        }
        window.socket.emit('chat', {
          uid: this.uid,
          to_id: this.chat.user.id,
          content: this.inputText,
          order_id: this.chat.order_id,
          order_no: this.chat.order_no,
        });

        this.inputText = ''
      },
      fixedBottom() {
        clearTimeout(this.timer)
      }
    },

    computed: {
      clickable() {
        return this.inputText.length > 0
      },
    },

    watch: {
      messages: {
        handler() {
          this.$cookie.set('record_chat', JSON.stringify(this.messages))
          //        localStorage.record_chat = JSON.stringify(this.messages)
          this.fixedBottom()
        },
        deep: true
      },
      dialogVisible(val){
        if(!val){
          window.socket.disconnect()
        }
      }
    },
  }
</script>

<style scoped="scoped" lang="scss">

</style>