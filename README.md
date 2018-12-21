#ASRemote
Android ADB Simple Remote Screen
通过ADB连接，在mac上进行Android屏幕简单的控制

参考项目：https://github.com/Genymobile/scrcpy

## 使用
- ADB
使用 adb forward 进行接口转发。通过Socket进行连接
- FFMPEG
接受Android端发送的H264 Naul，解码成YUV
- SDL2
接受解码后的YUV。提供屏幕渲染
接受键盘和点击事件。并通过socket 进行发送

## 线程模型
- client端(PC)
    - event_loop
        SDL的EventLoop。复制渲染上屏和分发事件
    - event_sender(Socket send)
        接受SDL分发的事件。并把对应的事件通过Socket分发给Android手机。
    - screen_receiver(Socket recv)
        通过Socket接受的 H264 Naul,使用FFmpeg进行解码。

- server端(Android)
    - screen record (Socket InputStream)
        使用SurfaceControl和MediaCodec进行屏幕录制，录制的结果通过Socket发送
    - event_loop (Socket OutputStream)
        接受Socket发送过来的事件。并调用对应的API进行事件的注入(InputManager)

### 线程通信
- frames
两块缓存区域。
   - decode_frame
        解码放置的frame
   - render_frame
        渲染需要的frame.使用该frame 进行render
数据流动
   - 生产的过程
     screen_receiver 负责生产。
   - 消费的过程
     event_loop 负责消费。将两块缓存区域进行交换，并把render_frame上屏

- event
一个event_queue队列来接受。可以使用链表
数据流动
   - 生产的过程
     event_loop 负责生产。并把数据送入队列当中
   - 消费的过程
     event_sender 负责消费。如果队列不为空，则进行发送

## 设计类
- client端
    SDL_Screen 负责SDL的显示和事件的接受
    FFmpegDecoder 负责数据的解码。和送入缓存
    EventSender 发送事件