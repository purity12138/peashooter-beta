<template>
  <div id="recommend">
    <!-- 页首 -->
    <myhead></myhead>
    <!-- 搜索框代码 -->
    <div class="searchbody">
      <!-- <img src="../assets/teamlogo.png" style="padding-left: 20%; width: 60px; height: auto;"/> -->
      <span class="title" style="padding-top: 30px;margin-left: 20%;">豌豆</span>
      <span class="title" style="padding-top: 40px;">射手</span>
      <img src="../../../static/images/pea.png" style="width: 30px; height: auto;" />
      <!-- <input id="userinput" type="text" v-model="search" placeholder="搜索你感兴趣的推荐" @keyup.enter="start()" />
      <input id="startsearch" type="submit" value="搜索" v-on:click="start()" /> -->
    </div>
    <!-- 搜索框代码 -->

    <!-- 主题代码 -->
    <div class="mybody">
      <div class="bodyleft">
        <div class="item1" v-for="num in shares">
          <div id="itemleft">
            <el-popover
                placement="bottom"
                trigger="hover"
                @after-enter="getShareUser(num.user_id)"
                @after-leave="cleanUser">
              <div>
                <p>用户名：{{num.username}}</p>
                <p v-show="shareUserNickname === 'nickname'">用户昵称：该用户暂未设置昵称</p>
                <p v-show="shareUserNickname !== 'nickname'">用户昵称：{{shareUserNickname}}</p>
                <p v-show="shareUserProfile === null">这个用户很懒，没有个人简介</p>
                <p v-show="shareUserProfile !== null">个人简介：{{shareUserProfile}}</p>
                <el-button type="infor" v-show="hadfollow === 1" @click="nofollow(num.user_id)">取消关注</el-button>
                <el-button type="primary" v-show="hadfollow === 0" @click="setfollow(num.user_id)">关 注</el-button>
              </div>
              <div slot="reference">
                <img src="../../../static/images/pea.png"/>
                <!-- <img src="../../../static/images/pea.png" @click="tomypage(num.user_id)"/> -->
                <!-- <img src="../../../static/images/pea.png" @click="getShareImformation(num.user_id)"/> -->
                <p align="center"><strong>{{num.username}}</strong></p>
              </div>
            </el-popover>
          </div>
          <div id="itemright">
            <div class="name"> {{num.title}} </div>
            <div class="tag">
              <i class="el-icon-price-tag"></i>
              {{num.tags}}
            </div>
            <div style="float: right;margin: 10px;color: #CCCCCC;">
              <i class="el-icon-date">{{formatDate(new Date(num.create_time*1000))}}</i>
            </div>
            <div class="article">
              <strong>一句话分享：</strong>{{num.content}}
            </div>
            <div class="article" >
            <hr align="center" color="cadetblue" size="2" />
              <!-- 内容链接 -->
              <p style="float: left;
                padding: 3px;
                background-color: #6BC4E0;
                color: #F2F2F2;
                border-radius: 3px;"
                v-bind:title="'http://129.204.247.165/'+num.route">
              <i class="el-icon-link"></i>
              内容链接
              </p>

              <!-- 内容预览 -->
              <p style="float: left;
                padding: 3px;
                background-color: #6BC4E0;
                color: #F2F2F2;
                border-radius: 3px;
                margin-left: 8px;"
                @click="showimg(num.route)"
                v-show="num.points === 0">
              <i class="el-icon-view"></i>
              预览
              </p>

              <!-- 需要积分支付 -->
              <div v-if="num.points > 0">
                <p style="padding: 3px;
                    background-color: #D39819;
                    float: left;
                    color: #F2F2F2;
                    border-radius: 4px;
                    margin-left: 8px;"
                    @click="readpay(num.title, user_id, num.points, num.route, num.id)"
                    :title="'累计下载'+num.times+'次'">
                  <i class="el-icon-lock"></i>
                  需要 {{num.points}} 积分/
                  <i class="el-icon-sell"></i>点击支付
                </p>
              </div>
                <!-- 无需积分下载 -->
              <div v-if="num.points === 0">
                <a target="_blank" :href="'http://129.204.247.165/'+num.route">
                  <p style="padding: 3px;
                      background-color: #42B983;
                      float: left;
                      color: #F2F2F2;
                      border-radius: 4px;
                      margin-left: 8px;"
                      @click="downloadfile(num.id,user_id)"
                      :title="'累计下载'+num.times+'次'">
                  <i class="el-icon-download"></i>
                      无需积分
                  </p>
                </a>
              </div>
              <!-- <p style="float: right;color: #8F949A;">下载次数:{{num.times}}</p> -->
            </div>
          </div>

        </div>
        <!--总条数{{total}}-->
        <el-divider></el-divider>

        <!-- 支付积分提示窗口 -->
        <el-dialog
          :showClose="showClo"
          :closeOnClickModal="showClo"
          :closeOnPressEscape="showClo"
          :before-close = "cancel"
          title="当前进行积分支付"
          :visible.sync="payDialogVisible"
          width="30%">
          <div v-show="hadpayit === 0 && payit === 0">
            <div>
              资源标题：{{this.payDialogtitle}}<br />
              所需积分：{{this.payDialogpoints}}<br />
              我的积分：{{this.payDialoguserlast}}<br /><br />
            </div>
            <span slot="footer" class="dialog-footer">
              <el-button @click="cancel()">取 消</el-button>
              <el-button type="primary" @click="paypoints()">支 付</el-button>
            </span>
          </div>
          <div v-show="hadpayit === 1 && payit === 0" style="text-align: center; margin: auto;">
            <strong style="font-size: x-large;"> {{this.message}} </strong><br /><br />
            <span slot="footer" class="dialog-footer">
              <el-button type="primary" @click="cancel()">
                <a target="_blank" :href="'http://129.204.247.165/'+this.route">
                  <p style="color:#DDDDDD">点击下载</p>
                </a>
              </el-button>
              <el-button type="primary" @click="cancel()">
                <p style="color:#DDDDDD">关 闭</p>
              </el-button>
            </span>
          </div>
          <div v-show="hadpayit === 0 && payit === 1" style="text-align: center; margin: auto;">
            <strong style="font-size: x-large;"> {{this.message}} </strong><br /><br />
            <div v-if="candownload === 1">
              <span slot="footer" class="dialog-footer">
                <el-button type="primary" @click="cancel()">
                  <a target="_blank" :href="'http://129.204.247.165/'+this.route">
                    <p style="color:#DDDDDD" @click="downloadfile(payDialogdocuid, user_id)">点击下载</p>
                  </a>
                </el-button>
              </span>
            </div>
            <div v-if="candownload === 0">
              <span slot="footer" class="dialog-footer">
                <el-button type="primary" @click="cancel()">
                  <p style="color:#DDDDDD">关 闭</p>
                </el-button>
              </span>
            </div>
          </div>
        </el-dialog>

        <!-- 图片预览窗口 -->
        <el-dialog
          title="文件预览"
          :visible.sync="documentView"
          width="30%">
          <div v-if="isPicture == 1">
            <img width="50%" :src="imgroute" />
          </div>
          <div v-else>{{showInView}}</div>
          <span slot="footer" class="dialog-footer">
            <!-- <el-button @click="documentView = false">取 消</el-button> -->
            <el-button type="primary" @click="closeimg()">关 闭</el-button>
          </span>
        </el-dialog>

        <!-- 分页控件代码 -->
        <div class="pageblock">
          <el-pagination
            @current-change="handleCurrentChange"
            background
            layout="prev, pager, next"
            :page-size="20"
            :total="total" >
          </el-pagination>
        </div>
      </div>
      <div class="bodyright">
        <!-- <div class="mybody m-margin-top-large" style="background-color:white" > -->
        <myhotlist></myhotlist>
        <!-- </div> -->
      </div>
    </div>
    <!-- 主题代码 -->
    <!-- 页尾 -->
    <myfoot></myfoot>
  </div>
</template>

<script>
  import myhead from '../myhead.vue'
  import myfoot from '../myfoot.vue'
  import myhotlist from './body/myhotlist.vue'

  export default {
    name: 'recommend',
    components: {
      myhead,
      myfoot,
      myhotlist,
    },
    data:function(){
      return{
        drawer: false,
        showClo:false,
        search:"",//搜索框输入
        shares:[],//服务器返回的信息
        userfollow:[],
        hadfollow:0,
        hadpayit:0,
        payit:0,
        total:0,
        candownload:0,
        followid:-1,
        isPicture:1,
        payDialogVisible:false,
        documentView:false,
        imgroute:"",
        payDialogtitle:"",
        payDialogpoints:"",
        payDialoguserlast:"",
        payDialogdocuid:"",
        route:"",
        message:"",
        showInView:"",
        more:0,
        user_id:window.sessionStorage.getItem('user_id'),//登录用户id
        shareUserProfile:"",
        shareUserNickname:"",
        shareUserUsername:"",
        circleUrl: "https://cube.elemecdn.com/3/7c/3ea6beec64369c2642b92c6726f1epng.png",
      }
    },
    methods:{

      /**计算评论的创建时间
       * @param {Object} date
       */
      formatDate(date) {
        var year=date.getFullYear();
        var month=date.getMonth()+1;
        var day=date.getDate();
        var hour=date.getHours();
        var minute=date.getMinutes();
        return year + '-' +
          (String(month).length > 1 ? month : '0' + month) + '-' +
          (String(day).length > 1 ? day : '0' + day) + ' ' +
          (String(hour).length > 1 ? hour : '0' + hour) + ':' +
          (String(minute).length > 1 ? minute : '0' + minute)
      },

      /**
       * 设置未关注
       */
      nofollow(touser) {
        this.hadfollow = 0
        const _this = this
        var c = 0
        for (let c=0; c<_this.userfollow.length; c++) {
          if (_this.userfollow[c].to_user_id == touser) {
            _this.followid = _this.userfollow[c].id
            break
          }
        }
        if (c == _this.userfollow.length) {
          _this.$message.warning("无关注该用户")
        } else {
          //删除关注表里的记录
          this.$axios({
            url:'/api/follows/'+_this.followid,
            method:'delete',
            headers: {'Content-Type': 'application/x-www-form-urlencoded'},
          }).then(function(res) {
            console.log("取消关注返回信息")
            console.log(res)
            _this.$message.success("已取消关注")
          })
          _this.followid = -1
        }
      },
      /**
       * 设置关注
       */
      async setfollow(touser) {
        this.hadfollow = 1
        const _this = this
        var had = 0
        for (let c=0; c<_this.userfollow.length; c++) {
          if (_this.userfollow[c].to_user_id == touser) {
            had = 1
            break
          }
        }
        if (had == 0) {
          const formData = new FormData()
          formData.append('from_user_id', _this.user_id)
          formData.append('to_user_id',touser)
          await this.$axios({
            url:'/api/follows',
            method:'post',
            data:formData,
            headers: {'Content-Type': 'application/x-www-form-urlencoded'},
          }).then(function(res) {
            console.log(res)
            _this.$message.success("已关注")
          })
          //重新获取用户的关注列表
          const myuserid={id:window.sessionStorage.getItem('user_id')}
          await this.$axios({
            method: 'post',
            url: '/api/user/getfollowed',
            data: this.$qs.stringify(myuserid),
          }).then(function (res) {
            console.log('我的关注',res)
            _this.userfollow = res.data.data
          }).catch(function (res) {
            console.log('我的关注获取失败 ')
          })
          await this.$forceUpdate();
          // _this.$router.go(0);
        } else {
          _this.$message.warning("已关注该用户，无需重复关注")
        }
      },
      /**
       * 清除资源分享者信息
       */
      cleanUser() {
        this.shareUserNickname = ""
        this.shareUserProfile = ""
        this.shareUserUsername = ""
        this.hadfollow = 0
      },

      /**获取某个资源分享者的信息
       * @param {Object} userid
       */
      async getShareUser(shareuserid) {
        const _this = this
        const formData = new FormData()
        formData.append('id', shareuserid)
        this.$axios({
          url:'/api/user/searchbyid',
          method:'post',
          data:formData,
          headers: {'Content-Type': 'application/x-www-form-urlencoded'}
        }).then(function(res) {
          console.log(res)
          _this.shareUserNickname = res.data.data.nickname
          _this.shareUserProfile = res.data.data.profile
          _this.shareUserUsername = res.data.data.username
        })
        for (let c=0; c<_this.userfollow.length; c++) {
          if (_this.userfollow[c].to_user_id == shareuserid)
            _this.hadfollow = 1
        }
        console.log("hadfollow="+_this.hadfollow)
        await this.$forceUpdate();
      },

      /**
       * 支付弹窗的取消按钮
       */
      cancel() {
        this.payDialogVisible = false
        this.hadpayit = 0
        this.payDialogdocuid = ""
        this.payDialogpoints = ""
        this.payDialogtitle = ""
        this.payDialoguserlast = ""
        this.candownload = 0
        this.payit = 0
      },

      /**
       * 支付弹窗的确认支付按钮
       */
      paypoints() {
        this.payit = 1
        console.log("点击支付按钮后")
        console.log("this.payDialoguserlast = "+this.payDialoguserlast)
        console.log("this.payDialogpoints = "+this.payDialogpoints)
        if (this.payDialogpoints > this.payDialoguserlast) {
          this.message = "您的积分不足"
          this.candownload = 0
        } else {
          this.candownload = 1
          this.message = "进入下载页面之后将自动支付积分"
        }
      },

      /**点击资源的支付按钮
       * @param {Object} title 资源标题
       * @param {Object} userid 支付积分的用户id
       * @param {Object} point 资源消耗的积分
       */
      readpay(title, userid, point, route, id){
        const _this = this
        this.payDialogVisible = true
        this.payDialogtitle = title
        this.payDialogpoints = point
        this.route = route
        this.payDialogdocuid = id
        this.judgepay(id, userid)
        const formData = new FormData()
        formData.append('id', userid)
        this.$axios({
          url:'/api/user/searchbyid',
          method:'post',
          data:formData,
          headers: {'Content-Type': 'application/x-www-form-urlencoded'}
        }).then(function(res) {
          _this.payDialoguserlast = res.data.data.points
          console.log("获取用户积分")
          console.log(res)
          console.log("this.payDialogpoints = "+point)
          console.log("_this.payDialoguserlast = "+_this.payDialoguserlast)
        })
      },

      /**判断是否支付过改资源
       * @param {Object} id
       * @param {Object} userid
       */
      judgepay(id, userid) {
        const _this = this
        const formData = new FormData()
        formData.append('id', id)
        formData.append('user_id', userid)
        // console.log("id="+id+",userid="+userid+",this.userid="+this.user_id)
        this.$axios({
          url: '/api/shares/jud',
          method: 'post',
          data: formData,
          headers: {'Content-Type': 'application/x-www-form-urlencoded'}
        }).then(function (res) {
          console.log("id="+id+",user_id="+userid)
          _this.hadpayit = res.data.data
          _this.candownload = 1
          _this.message = "您已支付过该资源，可直接点击下载"
          console.log("_this.hadpayit = ")
          console.log(_this.hadpayit)
        })
      },

      /**请求下载文件，使times属性的值加一
       * @param {Object} id 文件id
       * @param {Object} userid 下载文件的用户id
       */
      downloadfile(id, userid) {
        const formData = new FormData()
        formData.append('id', id)
        formData.append('user_id', userid)
        console.log("id="+id+",userid="+userid+",this.userid="+this.user_id)
        this.$axios({
          url: '/api/shares/getfile',
          method: 'post',
          data: formData,
          headers: {'Content-Type': 'application/x-www-form-urlencoded'},
        }).then(function (res) {
          console.log("id="+id+",user_id="+userid)
          console.log(res)
        })
      },
      /**文件预览
       * @param {Object} route 文件路径
       */
      showimg(route) {
        var list = route.split(".")
        console.log(list)
        if (list[1] != "jpg" && list[1] != "JPG" &&
          list[1] != "png" && list[1] != "PNG" &&
          list[1] != "gif" && list[1] != "GIF") {
            this.isPicture = 0
            this.showInView = "该文件不是.jpg/.png/.gif的图片文件。当前文件格式为."+list[1]+",暂不支持显示"
            this.documentView = true;
            console.log("判断该文件不是图片文件,文件格式为："+list[1])
        } else {
          this.imgroute = "http://129.204.247.165/"+route;
          this.documentView = true;
        }
      },

      /**
       * 关闭文件预览，重置变量
       */
      closeimg() {
        this.isPicture = 1
        this.showInView = ""
        this.documentView = false
      },
      //处理页码改变后的shares数据
      async handleCurrentChange(current){
        this.page=current
        const a={page:this.page}
        console.log('当前页码为')
        console.log(this.page)
        const _this=this
        await this.$axios({
          method: 'post',
          url: '/api/shares/gethot',
          data:this.$qs.stringify(a)
        }).then(function (res) {
          const __this =_this
          const r=res.data
          _this.shares=r.data
          console.log("得到的某一页shares总信息")
          console.log(_this.shares)

        }).catch(function (res) {
          console.log(res)
          console.log("得到的某一页shares总信息发生异常！请稍后重试...")
        })
        console.log('该页下的数据')
        console.log(_this.shares)
      },

      start(){
        // console.log(this.message)
        // alert(this.message)
      },

      tomypage(userid) {
        alert("前往"+userid+"主页")
        //跳转到id对应的用户主页
        //window.sessionstorage.setItem（‘user_id’,id）
      }
    },
    async created() {
      const _this=this;
      // console.log(_this.hadpayit)
      this.$axios({
        method: 'post',
        url: '/api/shares/gethot',
      }).then(function (res) {
        console.log('shares取到了')
        console.log(res)
        const r=res.data
        _this.shares=r.data
        // console.log(_this.shares.item.content)
      }).catch(function (res) {
        console.log("数据获取成功。。。")
      })
      //获取全部shares个数
      await this.$axios({
        method: 'post',
        url: '/api/shares/getcount',
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded'
        },
      }).then(function (res) {
        console.log(res)
        _this.total= parseInt(res.data.data)
        console.log("查找全部shares条数成功")
        console.log(_this.total)
      }).catch(function (res) {
        console.log("查找全部shares条数失败")
      })
      const myuserid={id:window.sessionStorage.getItem('user_id')}
      this.$axios({
        method: 'post',
        url: '/api/user/getfollowed',
        data: this.$qs.stringify(myuserid),
      }).then(function (res) {
        console.log('我的关注',res)
        _this.userfollow = res.data.data
      }).catch(function (res) {
        console.log('我的关注获取失败 ')
      })
    }
  }

  //设置页面无法进行鼠标右键，防止预览图片时可以对图片进行下载
  if (window.Event)
    document.captureEvents(Event.MOUSEUP);
  function nocontextmenu() {
    event.cancelBubble = true
    event.returnValue = false;
    return false;
  }
  function norightclick(e) {
    if (window.Event) {
      if (e.which == 2 || e.which == 3)
        return false;
    }
    else if (event.button == 2 || event.button == 3) {
      event.cancelBubble = true
      event.returnValue = false;
      return false;
    }
  }
  document.oncontextmenu = nocontextmenu; // for IE5+
  document.onmousedown = norightclick; // for all others
</script>

<style scoped>
  .searchbody {
    overflow: hidden;
    background-color: #DDDDDD;
    height: 85px;
    background-color: white;
  }
  .searchbody img {
    float: left;
    display: block;
    padding-top: 1%;
  }
  .title{
    font-family:"幼圆";
    font-size: 30px;
    font-weight: bold;
    color: #42B983;
    float: left;
  }
  .searchbody input[type=text]{
    width: 30%;
    padding: 1%;
    margin: 2% 0 2% 0;
    display: inline-block;
    border: 1px solid #ccc;
    border-radius: 4px;
    box-sizing: border-box;
  }
  .searchbody input[type=submit] {
    width: 5%;
    background-color: #4CAF50;
    color: white;
    padding: 1%;
    margin: 2% 0 2% 0;
    border: none;
    border-radius: 4px;
    cursor: pointer;
  }
  .searchbody input[type=submit]:hover {
    background-color: #45a049;
  }
  .mybody{
    width: 80%;
    height: auto;
    margin: auto;
    overflow: hidden;
  }
  .bodyleft{
    float: left;
    margin-top: 1%;
    width: 77%;
    height: auto;
    border-radius: 10px;
    padding: 1%;
  }
  .bodyright{
    text-align: center;
    float: right;
    width: 20%;
    height: auto;
    padding-top: 1%;
    margin-top: 1%;
    padding-bottom: 1%;
    overflow: hidden;
  }
  #itemcon hr {
    width: 90%;
  }
  .item1 {
    width: 90%;
    margin: auto;
    margin-bottom: 3%;
    /* border: 1px solid; */
    background-color: white;
    /* opacity:0.9; */
    overflow: hidden;
    text-align: left;
    border-radius: 10px;
    /* background: url(../../../static/images/background1.jpg); */
  }
  .item1:hover {
    background-color: aliceblue;
    box-shadow: 0px 0px 15px 15px rgba(0,0,0,0.2);
    transition: box-shadow 0.5s;
  }
  #itemleft {
    float: left;
    width: 10%;
    overflow: hidden;
    padding: 1%;
  }
  #itemleft img {
    float: left;
    width: 100%;
    height: 100%;
    border-radius: 5px;
    margin: 10%;
  }
  #itemright {
    float: right;
    width: 86%;
    overflow: hidden;
    padding: 1%;
  }
  #itemright .name{
    text-align: left;
    float: left;
    padding: 2%;
    font-size: x-large;
    font-family: "bookman old style";
    font-weight: bold;
  }
  #itemright .tag{
    text-align: left;
    float: left;
    padding: 3px;
    font-family: "幼圆";
    font-weight: bold;
    font-size: x-small;
    background-color: orangered;
    border-radius: 5px;
    color: #FFFFFF;
    margin-top: 2%;
  }
  #itemright .article{
    text-align: left;
    float: left;
    width: 96%;
    /* background-color: #CCCCCC; */
    padding: 2%;
  }
  #itemright .shareto{
    text-align: right;
    float: left;
    width: 96%;
    padding: 1% 1% 2% 1%;
  }
  #itemright textarea{
    width: 50%;
    height: 15px;
    padding: 1%;
    border-radius: 5px;
    resize: none;
    background-color: blanchedalmond;
  }
</style>
