# guli-14-阿里云视频点播


# guli-14-阿里云视频点播

> 使用视频id+视频凭证加密播放

## 一.后端

> service_vod

controller

```java
    //根据视频id获取视频凭证
    @GetMapping("getPlayAuth/{id}")
    public R getPlayAuth(@PathVariable String id) throws ClientException {
        //根据视频id获取视频凭证
        //创建初始化对象
        DefaultAcsClient defaultAcsClient = InitVodCilent.initVodClient(ConstantVodUtils.KEY_ID, ConstantVodUtils.KEY_SECRET);
        //创建获取视频地址request和response
        GetVideoPlayAuthRequest request = new GetVideoPlayAuthRequest();
        GetVideoPlayAuthResponse response = new GetVideoPlayAuthResponse();
        //向request对象里面设置视频id
        request.setVideoId(id);
        //调用初始化对象里面的方法，传递request，获取视频凭证
        response = defaultAcsClient.getAcsResponse(request);
        //获取凭证
            String playAuth = response.getPlayAuth();
            return R.ok().data("playAuth",playAuth);


    }
```

## 二.前端

api/vod.js

```js
import request from '@/utils/request'
export default {
  getPlayAuth(vid) {
    return request({
      url: `/eduvod/video/getPlayAuth/${vid}`,
      method: 'get'
    })
  }

}
```

页面 player/_vid.vue

```html
<template>
  <div>

    <!-- 阿里云视频播放器样式 -->
    <link rel="stylesheet" href="https://g.alicdn.com/de/prismplayer/2.8.1/skins/default/aliplayer-min.css" >
    <!-- 阿里云视频播放器脚本 -->
    <script charset="utf-8" type="text/javascript" src="https://g.alicdn.com/de/prismplayer/2.8.1/aliplayer-min.js" />

    <!-- 定义播放器dom -->
    <div id="J_prismPlayer" class="prism-player" />
  </div>
</template>
<script>
import vod from '@/api/vod'

export default {
    layout: 'video',//应用video布局
    asyncData({ params, error }) {
       return vod.getPlayAuth(params.vid)
        .then(response => {
            return { 
                playAuth: response.data.data.playAuth,
                vid: params.vid
            }
        })
    },
    mounted() { //页面渲染之后  created
        new Aliplayer({
            id: 'J_prismPlayer',
            vid: this.vid, // 视频id
            playauth: this.playAuth, // 播放凭证
            encryptType: '1', // 如果播放加密视频，则需设置encryptType=1，非加密视频无需设置此项
            width: '100%',
            height: '500px',
            // 以下可选设置
            // cover: 'http://guli.shop/photo/banner/1525939573202.jpg', // 封面
            // qualitySort: 'asc', // 清晰度排序

            // mediaType: 'video', // 返回音频还是视频
            // autoplay: false, // 自动播放
            // isLive: false, // 直播
            // rePlay: false, // 循环播放
            // preload: true,
            // controlBarVisibility: 'hover', // 控制条的显示方式：鼠标悬停
            // useH5Prism: true, // 播放器类型：html5
        }, function(player) {
            console.log('播放器创建成功')
        })
    }

}
</script>

```

## 三.其他

1. ```html
   <a :href="'/player/'+video.videoSourceId" target="_blank">
   ```

   a标签+target="_blank"会产生新页面

2. 播放器设置

   ```
   https://player.alicdn.com/aliplayer/setting/setting.html
   https://player.alicdn.com/aliplayer/presentation/index.html?type=cover
   ```

   


