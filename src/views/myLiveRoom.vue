<template>
  <div>
    <div className="">
      <div v-for="item in this.userIdList" :key="item">
        <video :id="'pull_player_' + item" autoPlay controls muted></video>
        <button @click="play(item)">播放</button>
      </div>

      <!--      <video  autoplay controls muted ref="video1"></video>-->

      <video id="rtc_media_player" controls autoPlay muted></video>
      <!--      <button class="" id="btn_publish" >开始推流</button>-->
    </div>

    <button @click="navigateToBasicAbilityTest">开始测试</button>

    <div style="text-align : left ;font-size: 50px">
      测试题为：
    </div>
    <img :src="require(`./R.jpg`)"  v-if="isshow" alt="Displayed Image" />
    <ImageDisplay v-if="isshow"></ImageDisplay>
    <image-display></image-display>

  </div>
</template>


<script>
//引入flc
import flv from 'flv.js'
import ImageDisplay from "./ImageDisplay";
export default {
  name: 'myLiveRoom',//对外暴露的接口的名字
  data() {
    return {
      isshow: false,
      basePullUrl:"https://livestream.chiichen.cn/rtc/v1/whep/?app=live&stream=",
      pushUrl: "",
      basePushUrl:"https://livestream.chiichen.cn/rtc/v1/whip/?app=live&stream=",
      roomId:"",
      userId:"",
      userIdList:[]
    }
  },
  components:[ImageDisplay],
  methods: {


    navigateToBasicAbilityTest() {
      this.$data.isshow=true
      // 假设这是要展示的图片 URL
      const imageUrl = './R.jpg'

      // // 导航到展示图片的页面
      // this.$router.push({
      //   //path: '/basic-ability-test',
      //   path:'/ImageDisplay',
      //   query: { imageUrl }
      // })
    },
    publish() {
      var sdk = null; // Global handler to do cleanup when republishing.
      var startPublish = function (pushUrl) {
        // $('#rtc_media_player').show();


        // Close PC when user replay.
        if (sdk) {
          sdk.close();
        }

        sdk = new SrsRtcWhipWhepAsync();
        // User should set the stream when publish is done, @see https://webrtc.org/getting-started/media-devices
        // However SRS SDK provides a consist API like https://webrtc.org/getting-started/remote-streams
        $('#rtc_media_player').prop('srcObject', sdk.stream);
        // Optional callback, SDK will add track to stream.
        // sdk.ontrack = function (event) { console.log('Got track', event); sdk.stream.addTrack(event.track); };

        // https://developer.mozilla.org/en-US/docs/Web/Media/Formats/WebRTC_codecs#getting_the_supported_codecs
        sdk.pc.onicegatheringstatechange = function (event) {
          if (sdk.pc.iceGatheringState === "complete") {
            $('#acodecs').html(SrsRtcFormatSenders(sdk.pc.getSenders(), "audio"));
            $('#vcodecs').html(SrsRtcFormatSenders(sdk.pc.getSenders(), "video"));
          }
        };

        // For example: webrtc://r.ossrs.net/live/livestream
        var url = pushUrl;
        sdk.publish(url).then(function (session) {
          $('#sessionid').html(session.sessionid);
          $('#simulator-drop').attr('href', session.simulator + '?drop=1&username=' + session.sessionid);
        }).catch(function (reason) {
          console.log(reason)
          // Throw by sdk.
          if (reason instanceof SrsError) {
            if (reason.name === 'HttpsRequiredError') {
              alert(`WebRTC推流必须是HTTPS或者localhost：${reason.name} ${reason.message}`);
            } else {
              alert(`${reason.name} ${reason.message}`);
            }
          }
          // See https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia#exceptions
          if (reason instanceof DOMException) {
            if (reason.name === 'NotFoundError') {
              alert(`找不到麦克风和摄像头设备：getUserMedia ${reason.name} ${reason.message}`);
            } else if (reason.name === 'NotAllowedError') {
              alert(`你禁止了网页访问摄像头和麦克风：getUserMedia ${reason.name} ${reason.message}`);
            } else if (['AbortError', 'NotAllowedError', 'NotFoundError', 'NotReadableError', 'OverconstrainedError', 'SecurityError', 'TypeError'].includes(reason.name)) {
              alert(`getUserMedia ${reason.name} ${reason.message}`);
            }
          }

          sdk.close();
          //$('#rtc_media_player').hide();
          console.error(reason);
        });
      };

      // $('#rtc_media_player').hide();
      // var query = parse_query_string();
      // srs_init_rtc("#txt_url", query);

      // $("#btn_publish").click(startPublish);
      startPublish(this.pushUrl);


      // Never play util windows loaded @see https://github.com/ossrs/srs/issues/2732
      // if (query.autostart === 'true') {
      //   window.addEventListener("load", function(){ startPublish(); });
      // }

      if (true) {
        window.addEventListener("load", function () {
          startPublish();
        });
      }
    },
    play(userId) {

      var startPlay = function (basePullUrl,userId) {
        // $('#rtc_media_player').show();
        var sdk = null; // Global handler to do cleanup when republishing.
        // Close PC when user replay.
        if (sdk) {
          sdk.close();
        }
        sdk = new SrsRtcWhipWhepAsync();

        // User should set the stream when publish is done, @see https://webrtc.org/getting-started/media-devices
        // However SRS SDK provides a consist API like https://webrtc.org/getting-started/remote-streams
        $('#pull_player_'+userId).prop('srcObject', sdk.stream);
        // Optional callback, SDK will add track to stream.
        // sdk.ontrack = function (event) { console.log('Got track', event); sdk.stream.addTrack(event.track); };

        // For example: webrtc://r.ossrs.net/live/livestream
        var url = basePullUrl+userId;
        sdk.play(url).then(function (session) {
          $('#sessionid').html(session.sessionid);
          $('#simulator-drop').attr('href', session.simulator + '?drop=1&username=' + session.sessionid);
        }).catch(function (reason) {
          sdk.close();
          // $('#rtc_media_player').hide();
          console.error(reason);
        });
      };

      // $('#rtc_media_player').hide();
      var query = parse_query_string();
      srs_init_whep("#txt_url", query);

      startPlay(this.basePullUrl,userId)
      // if (true) {
      //   window.addEventListener("load", function () {
      //     startPlay();
      //   });
      // }

    }


  },


  mounted() {
    // this.pullUrl=this.$route.query.pullUrl
    this.roomId=this.$route.query.roomId
    this.userId=localStorage.getItem("userId")

    this.pushUrl = this.basePushUrl+this.userId
    console.log(this.pushUrl)
    //加载视频
    // this.playVedio();  //拉
    this.play()
    this.publish() //推
    this.timer = setInterval(async () => {
      await this.$axios({
        method:"get",
        url:"/liveRoom/getRoomPersons?roomId="+this.roomId
      })
        .then(res=>{
          console.log(res.data)
          var tempArr =[]
          for (let i = 0; i < res.data.data.length; i++) {
            tempArr.push(res.data.data[i].userId)
            tempArr.concat(res.data.data[i].userId)
          }
          this.userIdList=tempArr
          console.log(this.userIdList)
        })
    }, 5000); // 每隔 5 秒发送一次请求
  },
  beforeRouteLeave() {
    clearInterval(this.timer)
  }
}
</script>

<!-- 只在当前页面生效的样式 -->
<style scoped>
.vedio {
  height: 100%;
  white-space: inherit;
}
</style>

