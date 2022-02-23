# FFmpeg备忘单

> [FFmpeg](https://ffmpeg.org)中常见视频处理操作的备忘单

## 操作

*如果文件已存在，使用 `-y` 标记来覆盖*

### 音频-视频 同步

> [参考](https://superuser.com/questions/982342/in-ffmpeg-how-to-delay-only-the-audio-of-a-mp4-video-without-converting-the-au)

```sh
# 音频延后 3 秒
$ ffmpeg -i input.mov -itsoffset 3 -i input.mov -map 0:v -map 1:a -codec:a copy -codec:v copy output.mov

# 视频延后 3 秒 (即音频提前 3 秒)
$ ffmpeg -i input.mov -itsoffset 3 -i input.mov -map 1:v -map 0:a -codec:a copy -codec:v copy output.mov
```

- 第二个 `-i` 标记必须 *紧跟着* `-itsoffset` 标记后面。

### 裁剪

> [参考](https://ffmpeg.org/ffmpeg-filters.html#crop)

```sh
# 裁剪到 360 宽, 640 高
$ ffmpeg -i input.mov -filter:v 'crop=360:640:0:0' -codec:a copy output.mov

# 裁剪到 360 宽,  640 高, 从坐标 (10, 20) 开始
$ ffmpeg -i input.mov -filter:v 'crop=360:640:10:20' -codec:a copy output.mov
```

### 格式

> [参考](https://stackoverflow.com/questions/8075992/using-ffmpeg-convert-a-file-from-one-output)

```sh
# 转换到 GIF
$ ffmpeg -i input.mov output.gif

# 从 GIF 转换
$ ffmpeg -i input.gif output.mov

# 两个非-GIF 文件转换
$ ffmpeg -i input.mov -codec:v copy -codec:a copy output.mp4
```

### 帧率

> [参考](https://trac.ffmpeg.org/wiki/ChangingFrameRate)

```sh
# 调整帧率到 12
$ ffmpeg -i input.mov -filter:v 'fps=fps=12' -codec:a copy output.mov
```

### 剥离音频

> [参考](https://superuser.com/questions/268985/remove-audio-from-video-file-with-ffmpeg)

```sh
# 移除音频
$ ffmpeg -i input.mov -codec:v copy -an output.mov
```

### 调整大小

> [Reference](https://trac.ffmpeg.org/wiki/Scaling)

```sh
# 调整到 360 宽,  640 高
$ ffmpeg -i input.mov -filter:v 'scale=360:640' -codec:a copy output.mov

# 调整为 360 宽，保持长宽比
$ ffmpeg -i input.mov -filter:v 'scale=360:-1' -codec:a copy output.mov

# 调整到 640 高, 保持长宽比
$ ffmpeg -i input.mov -filter:v 'scale=-1:640' -codec:a copy output.mov
```

- 设置 `width` 或 `height` 任意一个到 `-1` 来保持长宽比。

### 倒放

> [参考](https://video.stackexchange.com/questions/17738/how-to-use-ffmpeg-command-for-reverse-video)

```sh
# 倒放
$ ffmpeg -i input.mov -filter:v 'reverse' -filter:a 'areverse' output.mov
```

### 旋转

> [参考](https://stackoverflow.com/questions/3937387/rotating-videos-with-ffmpeg)

```sh
# 顺时针旋转90度
$ ffmpeg -i input.mov -filter:v 'transpose=1' -codec:a copy output.mov

# 逆时针旋转90度
$ ffmpeg -i input.mov -filter:v 'transpose=2' -codec:a copy output.mov

# 旋转180度
$ ffmpeg -i input.mov -filter:v 'transpose=1,transpose=1' -codec:a copy output.mov
```

### 速度

> [参考](https://trac.ffmpeg.org/wiki/How%20to%20speed%20up%20/%20slow%20down%20a%20video)

```sh
# 四分之一的速度
$ ffmpeg -i input.mov -filter:v 'setpts=4*PTS' -filter:a 'atempo=0.5,atempo=0.5' output.mov

# 速度减半
$ ffmpeg -i input.mov -filter:v 'setpts=2*PTS' -filter:a 'atempo=0.5' output.mov

# 速度加倍
$ ffmpeg -i input.mov -filter:v 'setpts=0.5*PTS' -filter:a 'atempo=2' output.mov

# 四倍的速度
$ ffmpeg -i input.mov -filter:v 'setpts=0.25*PTS' -filter:a 'atempo=2,atempo=2' output.mov
```

- 使用共识 `1 ÷ speed` 来计算 `setpts` 的值。
  - 速度减半: `setpts=2*PTS` 因为 `1 ÷ 0.5 = 2`。
  - 速度加倍: `setpts=0.5*PTS` 因为 `1 ÷ 2 = 0.5`。

- 每个 `atempo` 选项必须介于 0.5 和 2。
  - 四分之一的速度: `atempo=0.5,atempo=0.5` 因为 `0.5 × 0.5 = 0.25`.
  - 四倍的速度: `atempo=2,atempo=2` 因为 `2 × 2 = 4`.

### 字幕

> [参考](https://stackoverflow.com/questions/57869367/ffmpeg-subtitles-alignment-and-position)

```sh
# 字幕写入视频
$ ffmpeg -i input.mov -filter:v 'subtitles=subtitles.srt' -codec:a copy output.mov

# 字幕写入视频, 使用自定义样式
$ ffmpeg -i input.mov -filter:v "subtitles=subtitles.srt:force_style='FontName=Menlo Bold,Fontsize=18'" -codec:a copy output.mov
```

### 修剪

> [参考](https://trac.ffmpeg.org/wiki/Seeking#Cuttingsmallsections)

```sh
# 剪去 0:05 到 0:10
$ ffmpeg -ss 0:05 -to 0:10 -i input.mov -codec:v copy -codec:a copy output.mov

# 减去 0:05 到视频结束
$ ffmpeg -ss 0:05 -i input.mov -codec:v copy -codec:a copy output.mov
```

-  `-ss` 和 `-to` 标签必须放在  `-i` 标签 *前面*。

### 音量

> [参考](https://trac.ffmpeg.org/wiki/AudioVolume)

```sh
# 音量减半
$ ffmpeg -i input.mov -codec:v copy -filter:a 'volume=0.5' output.mov

# 音量加倍
$ ffmpeg -i input.mov -codec:v copy -filter:a 'volume=2' output.mov
```

## 相关命令行标签

> [参考](https://ffmpeg.org/ffmpeg.html)

```
ffmpeg
  -an
  -ss <timestamp>
  -to <timestamp>
  -itsoffset <offset>
  -i <input>
  -map <stream>
  -codec:a <codec>
  -codec:v <codec>
  -filter:a <filtergraph>
  -filter:v <filtergraph>
  -y
  <output>
```

## 另见

- [vdx](https://github.com/yuanqing/vdx)
