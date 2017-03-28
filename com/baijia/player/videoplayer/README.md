# 百家云点播 Android SDK 集成文档

## 1、添加依赖(目前仅支持使用 Android Studio 的方式集成)
在工程的根 build.gradle 文件中添加百家云的 maven 源
```gradle
allprojects {
      repositories {
        jcenter()
        maven { url 'https://raw.github.com/baijia/maven/master/' }
     }
}
```

在需要使用点播播放器的 module 中添加 library 的依赖
```gradle
dependencies {
   compile 'com.baijia.player:video-player:1.1.0'
}
```

## 2、在 xml 布局文件中添加视频播放控件
```xml
<com.baijiahulian.player.BJPlayerView
  android:id="@+id/videoView"
  android:layout_width="match_parent"
  android:layout_height="wrap_content"
  app:top_controller="@layout/bjplayer_layout_top_controller"
  app:bottom_controller="@layout/bjplayer_layout_bottom_controller"
  app:center_controller="@layout/bjplayer_layout_center_controller"
  app:aspect_ratio="fit_parent_16_9">
</com.baijiahulian.player.BJPlayerView>

```

#### 以下配置均为可选， 不设置即使用默认值
- a) aspect_ratio: 播放窗口宽高比， fit_parent_16_9 表示适配布局为 16:9
- b) top_controller: 视频播放窗口顶部控制布局 
- c) bottom_controller: 视频播放窗口底部控制布局 
- d) center_controller: 视频播放窗口中部控制布局



## 3、初始化 BJPlayerView 配置
```java
playerView = (BJPlayerView) findViewById(R.id.videoView);
 playerView.setBottomPresenter(new BJBottomViewPresenter(playerView.getBottomView()));
 playerView.setTopPresenter(new BJTopViewPresenter(playerView.getTopView()));
 playerView.setCenterPresenter(new BJCenterViewPresenter(playerView.getCenterView()));

playerView.setOnVideoPlayerListener(new BJPlayerView.OnVideoPlayerListener() {

    @Override
    public void onVideoInfoInitialized(BJPlayerView playerView, HttpException exception) {
      if (exception != null) {
          // 视频信息初始化成功
          VideoItem videoItem = playerView.getVideoItem();
      }
  } 

  @Override
  public void onPlayCompletion(BJPlayerView playerView, VideoItem item, SectionItem nextSection) {
      if (nextSection != null) {
           // play next section
           playerView.setVideoId(nextSection.videoId, "theToken");
           playerView.playVideo();
      }
 }
});

```
- a) setXXXPresenter(): 设置对应(上、中、下) 区域布局的控制实现。 如果使用的是默认播放器样式， 请使用 SDK 提供的默认实现
- b) setOnVideoPlayerListener(): 监听播放器回调
- c) onVideoInfoInitialized(): 正式播放之前， 会从服务器获取视频信息。 初始化完成可以得到 VideoItem 对象
- d) onPlayCompletion(): 当一个视频播放完成，或者被切换之后，回调这个方法， 同时给出下一个应该播放的视频的建议(nextSection)

### 3.1 BJPlayerView 对应的 Activity 重载实现以下方法 

```java
@Override
public void onConfigurationChanged(Configuration newConfig) {
  super.onConfigurationChanged(newConfig);
  if (playerView != null) {
    playerView.onConfigurationChanged(newConfig);
  }
}


@Override
public void onBackPressed() {
  if (!playerView.onBackPressed()) {
    super.onBackPressed();
  }
}


@Override
protected void onResume() {
  super.onResume();
  if (playerView != null) {
    playerView.onResume();
  }
}

@Override
protected void onPause() {
  super.onPause();
  if (playerView != null) {
    playerView.onPause();
  }
}

@Override
protected void onDestroy() {
  super.onDestroy();
  if (playerView != null) {
    playerView.onDestroy();
  }
}
```

### 3.2 为了防止屏幕旋转导致界面重新绘制，Activity 增加 configChanges 配置, 如下
```xml
<activity android:name=".MainActivity"
  android:theme="@style/Theme.AppCompat.NoActionBar"
  android:configChanges="keyboardHidden|orientation|screenSize">
    <intent-filter>
      <action android:name="android.intent.action.MAIN"/>
      <category android:name="android.intent.category.LAUNCHER"/>
  </intent-filter>
</activity>

```

### 3.3 配置合作方ID
```java
/**
 * 设置 合作方 ID。 如果没有设置， 无法使用 SDK
 *
 * @param partnerId 合作方 ID
 * @param deploy    运行环境;
 *  <ul>
 *   <li>测试环境: PLAYER_DEPLOY_DEBUG</li>
 *   <li>Beta 环境: PLAYER_DEPLOY_BETA</li>
 *   <li>线上环境: PLAYER_DEPLOY_ONLINE</li>
 * </ul>
 */
playerView.initPartner(long partnerId, int deploy);
```


## 4、设置视频源
```java

/**
* <p>设置视频源</p>
*
* @param videoId   视频 id
* @param token     视频的token
*/
playerView.setVideoId(long videoId, String token);
```

```java
/**
*  <p>设置视频源</p>
* @param sectioNid 视频集 Id
* @param videoId 视频 id
* @param token   视频的token
*/
playerView.setVideoId(long sectionId, long videoId, String token);
```

### 4.1  设置视频集
```java
/**
 * <p>设置播放列表</p>
 *
 * @param customSectionList 用户自己提供的播放列表
 */
public void setCustomSectionList(SectionItem[] customSectionList);
```

```java
/**
 * 加载视频集（视频集信息托管在百家云平台）
 *
 * @param sectionId
 */
public void loadSections(long sectionId);
```

### 4.2 设置私有用户信息，作为最终统计报表的过滤标记。
```java
/**
 * <p>设置用户自定义信息</p>
 *
 * @param userInfo 用户自定义消息
 */
public void setUserInfo(String userInfo);
```
*字符串的具体格式可根据自己的业务需求自定义*

## 5、常用 apis
```java
/**
* <p>播放视频</p>
*/
@Override
public void playVideo();
```

```java
/**
* <p>暂停视频</p>
*/
@Override
public void pauseVideo()
```

```java
/**
* 调整进度
* @param position
*/
@Override
public void seekVideo(int position) 
```

```java
/**
* <p>设置播放视频清晰度</p>
* @param definition 清晰度
* <ul>
*      <li>标清: VIDEO_DEFINITION_STD</li>
*      <li>高清: VIDEO_DEFINITION_HIGH</li>
*      <li>超清: VIDEO_DEFINITION_SUPER</li>
* </ul>
*/
@Override
public void setVideoDefinition(int definition);
```

```java
/**
* <p>设置视频播放倍速</p>
*
* @param rate 播放速率
* <ul>
*      <li>1 倍正常速率: VIDEO_RATE_1_X</li>
*      <li>1.5 倍速率: VIDEO_RATE_1_5_X</li>
*      <li>2 倍速率: VIDEO_RATE_2_X</li>
* </ul>
*/
@Override
public void setVideoRate(int rate);
```

## 6、自定义播放器样式
播放器 SDK 能够支持灵活的样式调整。

### 6.1、调整播放器主题色

>  如果只是需要调整播放器的主题色，在宿主工程中添加一个颜色定义
```xml
<color name="bjplayer_color_primary">#ff9900</color>
```


### 6.2、完整的样式更换
>  如果不希望使用 SDK 默认提供的样式，可以完全自己实现一套布局。

#### 一: 将上文第二步中布局内的播放器控件的 xxx_controller 指向自定义的布局 
#### 二: 针对上、中、下三个布局，依次实现三个接口, 具体实现可参考 BJXXXViewPresenter 的实现。 如下：
```java
interface TopView {
  void onBind(IPlayer player);
  void setTitle(String title);
  void setOrientation(int orientation);
  void setOnBackClickListener(View.OnClickListener listener);
}
```

```java
interface CenterView {

  void onBind(IPlayer player);
  boolean onBackTouch();
  void setOrientation(int orientation);
  void showProgressSlide(int delta);
  void showLoading(String message);
  void dismissLoading();
  void showVolumeSlide(int volume, int maxVolume);
  void showBrightnessSlide(int brightness);
  void showError(int what, int extra);
  void showError(int code, String message);
  void showWarning(String warn);

  void onShow();
  void onHide();
  void onVideoInfoLoaded(VideoItem videoItem);

  boolean isDialogShowing();
}
```

```java
interface BottomView {

  void onBind(IPlayer player);

  void setDuration(int duration);

  void setCurrentPosition(int position);

  void setIsPlaying(boolean isPlaying);

  void setOrientation(int orientation);

  void onBufferingUpdate(int percent);
}
```
#### 三: 将 XXXView 接口的实现，传入 BJPlayerView 中
```java
playerView.setBottomPresenter(BottomViewImpl);
playerView.setTopPresenter(TopViewImpl);
playerView.setCenterPresenter(CenterViewImpl);
```
