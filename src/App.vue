<template>
  <div id="app">
    <div class="logo">
      <img src="http://7xu2yc.com1.z0.glb.clouddn.com/logo.96ec8ad.png">
    </div>
    <div class="mdui-container">
      <div class="mdui-row">
        <div class="mdui-col-sm-2"></div>
        <div class="mdui-col-sm-8 search-bar">
          <div class="mdui-col-sm-1 mdui-text-left">
            <div class="mdui-textfield">
              <select class="mdui-select" v-model="downloadType">
                <option value="AV">AV</option>
                <option value="EP">EP</option>
              </select>
            </div>
          </div>
          <div class="mdui-col-sm-9">
            <div class="mdui-textfield">
              <input class="mdui-textfield-input" type="text" v-model="videoUrl" placeholder="请输入视频URL..."/>
            </div>
          </div>
          <div class="mdui-col-sm-2 mdui-text-right">
            <div class="mdui-textfield">
              <button class="mdui-btn mdui-btn-raised mdui-ripple mdui-color-theme-accent" @click="checkIsAvNum">下载</button>
            </div>
          </div>
        </div>
        <div class="mdui-col-sm-2"></div>
      </div>
      <transition name="fade">
        <w-result :video="video" v-if="flag"></w-result>
      </transition>

    </div>
  </div>
</template>

<script>
  import { jsonp } from './common/jsonp'
  import axios from 'axios'
  import result from './components/Result.vue'
  import mdui from 'mdui'
  export default {
    name: 'app',
    components: {
      'w-result': result
    },
    data () {
      return {
        downloadType: 'AV',
        videoUrl: '',
        flag: false,
        video: {
          videoImg: '',
          videoTitle: '',
          videoUpName: '',
          videoUpLink: '',
          videoInfo: [
            {
              videoItem: '',
              icon: 'sd_storage'
            },
            {
              videoItem: '',
              icon: 'access_time'
            },
            {
              videoItem: '',
              icon: 'ondemand_video'
            }
          ],
          videoDownloadUrl: '',
          videoBarrage:''
        }
      }
    },
    methods: {
      urlToAvOrEp(url){
        let check = url.split("/");
        for( let i=0,len=check.length; i<len; i++ ){
          if(check[i].indexOf('av') >= 0 && !isNaN(parseInt(check[i].replace('av','')))){
            let av = check[i].replace('av','');
            return av;
          }
          if(check[i].indexOf('ep') >= 0 && !isNaN(parseInt(check[i].replace('ep','')))){
            let ep = check[i].replace('ep','');
            return ep;
          }
        }
      },
      epToav(Num){
        let url = '/bangumi/play/ep' + Num;
        axios.get(url)
          .then(result => {
            let str = (result.data).replace(/[\r\n ]/g,"");
            let str1 = str.match(/"epInfo":(\S*),"epList"/)[1];
            let res = JSON.parse(str1);
            this.checkAvNum(res.aid);
          })
          .catch(error => {
            console.log('ep转av失败!!!');
          })
      },
      checkIsAvNum(){
        let Num = this.urlToAvOrEp(this.videoUrl);
        if (this.downloadType === 'AV'){
          this.checkAvNum(Num);
        }else {
          this.epToav(Num);
        }
      },
      checkAvNum(AVNUM){
        let that = this;
        let url = 'http://api.bilibili.com/playurl';
        let data = {
          aid: AVNUM,
          page: 2,
          platform: 'html5',
          quality: 1,
          vtype: 'mp4',
          type: 'jsonp',
          token: 'd3bd9275f0f2cda07f2406740db06c5d'
        };
        console.log(data);
        let options = {
          param: 'callback'
        };
        jsonp(url, data,options)
          .then(res => {
            console.log(res)
            if (res.durl){
              console.log('多p');
              mdui.prompt('这是一个分P视频，请输入分P号',
                function (value) {
                  that.getAllInfo(value,AVNUM)
                },
                function (value) {},
                {
                  confirmText: '确定',
                  cancelText: '取消'
                }
              );
            }else {
              console.log('单p');
              that.getAllInfo(1,AVNUM);
            }
          })
          .catch(err => {
            console.log('检查av号出错!');
          })
      },
      getAllInfo(value,avNum){
        let that = this;
        let url = 'http://api.bilibili.com/view';
        let data = {
          type: 'jsonp',
          appkey: '8e9fc618fbd41e28',
          page: 1,
          id: avNum
        };
        let options = {
          param: 'callback'
        };
        jsonp(url,data,options)
          .then(res => {
            console.log(res);
            let userId = res.mid,
                videoTitle = res.title,
                upName = res.author,
                videoImg = res.pic,
                videoDanmu = res.cid;
            let realimg = 'https://images.weserv.nl/?url=' + videoImg.replace(/https\:/,'');
            let upLink = `https://space.bilibili.com/${ userId }/#!/`;
            that.video.videoImg = realimg;
            that.video.videoTitle = '[p' + value + ']' + videoTitle;
            that.video.videoUpName = upName;
            that.video.videoUpLink = upLink;
            let url = 'http://api.bilibili.com/playurl';
            let data = {
              aid: avNum,
              page: value,
              platform: 'html5',
              quality: 1,
              vtype: 'mp4',
              type: 'jsonp',
              token: 'd3bd9275f0f2cda07f2406740db06c5d'
            };
            let options = {
              param: 'callback'
            };
            jsonp(url, data,options)
              .then(result => {
                console.log(result);
                let videoDownloadUrl = result.durl[0].url,
                    videoTime = result.durl[0].length,
                    videoFormat = result.format,
                    videoSize = result.durl[0].size;
                that.video.videoInfo[0].videoItem = (videoSize/1048576).toFixed(2) + 'mb';
                that.video.videoInfo[1].videoItem = (videoTime/60000).toFixed(2) + 'min';
                that.video.videoInfo[2].videoItem = videoFormat.toUpperCase();
                that.video.videoDownloadUrl = videoDownloadUrl;
                that.getVideoBarrage(videoDanmu);
              })
              .catch(err => {
                console.log('获取视频下载链接失败!')
              })
          })
          .catch(error => {
            console.log('获取视频信息失败!')
          })
      },
      getVideoBarrage(videoDanmu){
        let that = this;
        axios.get('/comment/'+videoDanmu+'.xml')
          .then(response => {
            console.log(response);
            let xmlData = that.strToXml(response.data);
            let danMu = xmlData.getElementsByTagName('d');
            let str = "";
            for(let i=0;i<danMu.length;i++){
              str += danMu[i].innerHTML + '\n';
            }
            that.video.videoBarrage = str;
            console.log(str);
            that.flag = true;
          })
          .catch(errors => {
            console.log('获取弹幕信息出错!');
          })
      },
      strToXml(str){
        if(document.all){
          var xmlDom=new ActiveXObject("Microsoft.XMLDOM");
          xmlDom.loadXML(str);
          return xmlDom;
        }
        else
          return new DOMParser().parseFromString(str, "text/xml");
      }
    }
  }
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
.logo{
  padding-top: 60px;
  text-align: center;
}
.fade-enter-active,.fade-leave-active{
  transition: all 0.5s ease;
}

.fade-enter,.fade-leave-active {
  opacity: 0
}
  .search-bar{
    background: #fff;
    border-radius: 5px;
    padding-bottom: 7px;
    -moz-box-shadow:1px 1px 5px #333333;
    -webkit-box-shadow:1px 1px 5px #333333;
    box-shadow:1px 1px 5px #333333;
  }
</style>
