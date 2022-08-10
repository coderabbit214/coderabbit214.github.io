# 设计模式-适配器模式


# 设计模式-适配器模式

> 把一个接口转换成用户希望的另一个接口，使的原本不兼容而不能一起工作的类可以一起工作。

## 何时使用

1. 系统需要使用现有的类，而此类的接口不符合系统的需要。 
2. 想要建立一个可以重复使用的类，用于与一些彼此之间没有太大关联的一些类，包括一些可能在将来引进的类一起工作，这些源类不一定有一致的接口。 
3. 通过接口转换，将一个类插入另一个类系中。（比如老虎和飞禽，现在多了一个飞虎，在不增加实体的需求下，增加一个适配器，在里面包容一个虎对象，实现飞的接口。）

## 图

![20201204-adapter](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/20201204-adapter.png)

> 我们有一个 *MediaPlayer* 接口和一个实现了 *MediaPlayer* 接口的实体类 *AudioPlayer*。默认情况下，*AudioPlayer* 可以播放 mp3 格式的音频文件。
>
> 我们还有另一个接口 *AdvancedMediaPlayer* 和实现了 *AdvancedMediaPlayer* 接口的实体类。该类可以播放 vlc 和 mp4 格式的文件。
>
> 我们想要让 *AudioPlayer* 播放其他格式的音频文件。为了实现这个功能，我们需要创建一个实现了 *MediaPlayer* 接口的适配器类 *MediaAdapter*，并使用 *AdvancedMediaPlayer* 对象来播放所需的格式。
>
> *AudioPlayer* 使用适配器类 *MediaAdapter* 传递所需的音频类型，不需要知道能播放所需格式音频的实际类。*AdapterPatternDemo* 类使用 *AudioPlayer* 类来播放各种格式。

## 代码实现

```java
package com.designPattern.adapterPattern;

/**
 * 适配器模式
 * 把一个接口转换成用户希望的另一个接口，使的原本不兼容而不能一起工作的类可以一起工作。
 */
public class AdapterPattern {
    public static void main(String[] args) {
        MediaPlayer mediaPlayer = new AudioPlayer();
        mediaPlayer.play("mp4","111");
    }
}
//创建播放器接口
interface MediaPlayer{
    void play(String audioType, String fileName);
}
//创建高级播放器接口
interface AdvancedMediaPlayer {
    void playVlc(String fileName);
    void playMp4(String fileName);
}
//实现高级播放器接口
class VlcMediaPlayer implements AdvancedMediaPlayer {

    @Override
    public void playVlc(String fileName) {
        System.out.println("playVlc"+fileName);
    }

    @Override
    public void playMp4(String fileName) {

    }
}
class Mp4MediaPlayer implements AdvancedMediaPlayer {

    @Override
    public void playVlc(String fileName) {

    }

    @Override
    public void playMp4(String fileName) {
        System.out.println("playMp4"+fileName);
    }
}
//创建实现了 MediaPlayer 接口的适配器类
class MediaAdapter implements MediaPlayer{

    AdvancedMediaPlayer advancedMediaPlayer;
    MediaAdapter(String audioType){
        if(audioType.equalsIgnoreCase("vlc") ){
            advancedMediaPlayer = new VlcMediaPlayer();
        } else if (audioType.equalsIgnoreCase("mp4")){
            advancedMediaPlayer = new Mp4MediaPlayer();
        }
    }
    @Override
    public void play(String audioType, String fileName) {
        if(audioType.equalsIgnoreCase("vlc")){
            advancedMediaPlayer.playVlc(fileName);
        }else if(audioType.equalsIgnoreCase("mp4")){
            advancedMediaPlayer.playMp4(fileName);
        }
    }
}
//创建实现了 MediaPlayer 接口的实体类
class AudioPlayer implements MediaPlayer {
    MediaAdapter mediaAdapter;

    @Override
    public void play(String audioType, String fileName) {
        //播放 mp3 音乐文件的内置支持
        if(audioType.equalsIgnoreCase("mp3")){
            System.out.println("Playing mp3 file. Name: "+ fileName);
        }
        //mediaAdapter 提供了播放其他文件格式的支持
        else if(audioType.equalsIgnoreCase("vlc")
                || audioType.equalsIgnoreCase("mp4")){
            mediaAdapter = new MediaAdapter(audioType);
            mediaAdapter.play(audioType, fileName);
        }
        else{
            System.out.println("Invalid media. "+
                    audioType + " format not supported");
        }
    }
}
```


