# 视频字幕播放器

这是一个基于 Electron 和 Vue.js 的视频字幕播放器应用程序。它允许用户加载视频和字幕文件，并提供同步播放和交互功能。

## 主要功能

1. 视频播放：支持加载和播放本地视频文件。
2. 字幕同步：自动查找并加载与视频文件同名的字幕文件。
3. 字幕显示：在视频下方或旁边显示字幕文本。
4. 布局切换：支持水平和垂直布局，适应不同的观看需求。
5. 字幕导航：点击字幕文本可以跳转到视频的相应时间点。
6. 字幕列表：侧边栏显示完整的字幕列表，支持搜索功能。

## 技术栈

- Electron
- Vue.js 3
- Element Plus
- Video.js

## 安装和运行

1. 克隆仓库：
   ```bash
   git clone [仓库URL]
   ```

2. 安装依赖：
   ```bash
   npm install
   ```

3. 运行开发服务器：
   ```bash
   npm run serve
   ```

4. 构建生产版本：
   ```bash
   npm run build
   ```

## 使用说明

1. 启动应用后，点击"选择视频"按钮加载本地视频文件。
2. 应用将自动尝试加载同名的字幕文件（��持 .srt 和 .txt 格式）。
3. 使用视频播放器控件来控制视频播放。
4. 点击字幕文本可以跳转到视频的相应位置。
5. 使用"垂直布局"/"水平布局"按钮来切换界面布局。
6. 点击"打开字幕列表"按钮查看完整字幕，并可以搜索特定内容。

## 待办事项

- [ ] 支持更多视频格式（如 .avi, .mkv 等）
- [ ] 添加字幕编辑功能
- [ ] 实现字幕时间轴调整功能
- [ ] 添加多语言支持
- [ ] 优化字幕搜索算法，支持模糊搜索
- [ ] 实现字幕样式自定义（字体、颜色、大小等）
- [ ] 添加快捷键支持
- [ ] 实现自动更新功能
- [ ] 添加用户偏好设置（如默认字幕语言、播放速度等）
- [ ] 集成在线字幕搜索和下载功能

## 贡献

欢迎提交问题和拉取请求。对于重大更改，请先开issue讨论您想要改变的内容。

## 许可证

[MIT](https://choosealicense.com/licenses/mit/)