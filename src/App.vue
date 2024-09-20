<template>
  <div id="app">
    <sidebar-component 
      :subtitles="parsedSubtitles"
      v-model:visible="sidebarVisible"
      @subtitle-click="onSubtitleClick"
    />
    <div class="main-content">
      <div class="control-panel">
        <el-button @click="selectVideo" type="primary">选择视频</el-button>
        <el-button @click="toggleLayout" type="info">
          {{ isHorizontalLayout ? '垂直布局' : '水平布局' }}
        </el-button>
        <el-button @click="toggleSidebar" type="success">
          {{ sidebarVisible ? '关闭字幕列表' : '打开字幕列表' }}
        </el-button>
        <span v-if="videoFileName" class="video-file-name">{{ videoFileName }}</span>
      </div>
      <div :class="['content-wrapper', { 'horizontal-layout': isHorizontalLayout }]">
        <div class="video-container">
          <video-player 
            v-if="videoSrc" 
            ref="videoPlayer"
            :src="videoSrc" 
            :subtitles="parsedSubtitles"
            class="video-player"
            @timeupdate="onTimeUpdate"
          ></video-player>
        </div>
        <article-component 
          :subtitles="parsedSubtitles" 
          :currentTime="currentTime"
          @subtitle-click="onSubtitleClick"
          class="article-component"
        ></article-component>
      </div>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue'
import { ElButton } from 'element-plus'
import VideoPlayer from './components/VideoPlayer.vue'
import ArticleComponent from './components/ArticleComponent.vue'
import SidebarComponent from './components/SidebarComponent.vue'

export default {
  name: 'App',
  components: {
    ElButton,
    VideoPlayer,
    ArticleComponent,
    SidebarComponent
  },
  setup() {
    const sidebarVisible = ref(false)

    const toggleSidebar = () => {
      sidebarVisible.value = !sidebarVisible.value
    }

    return {
      sidebarVisible,
      toggleSidebar
    }
  },
  data() {
    return {
      currentTime: 0,
      videoSrc: null,
      subtitleContent: '',
      parsedSubtitles: [],
      videoDirectory: null,
      isHorizontalLayout: false,
      videoFileName: null
    }
  },
  methods: {
    onTimeUpdate(time) {
      this.currentTime = time;
    },

    async selectVideo() {
      if ('showOpenFilePicker' in window) {
        try {
          const [fileHandle] = await window.showOpenFilePicker({
            types: [{
              description: 'Video Files',
              accept: { 'video/*': ['.mp4', '.webm', '.ogg'] }
            }],
            multiple: false
          });
          
          const file = await fileHandle.getFile();
          
          const fileURL = URL.createObjectURL(file);
          this.videoSrc = fileURL;

          this.videoFileName = file.name; // 添加这行来保存文件名

          // 自动查找并加载字幕文件
          await this.findAndLoadSubtitle(fileHandle);

          // 强制更新视频播放器
          this.$nextTick(() => {
            this.updateVideoPlayerSize();
          });
        } catch (error) {
          console.error('选择视频失败:', error);
        }
      } else {
        // 如果浏览器不支持 FileSystemAccess API,使用传统方法
        const input = document.createElement('input');
        input.type = 'file';
        input.accept = 'video/*';
        input.onchange = async (event) => {
          const file = event.target.files[0];
          this.videoSrc = URL.createObjectURL(file);
          this.videoFileName = file.name; // 添加这行来保存文件名

          this.videoDirectory = null;
          
          // 尝试在同一目录下查找字幕文件（传统方法无法访问文件系统）
          console.warn('您的浏览器不支持自动查找字幕文件。请手动选择字幕文件。');
        };
        input.click();
      }
    },
    
    async findAndLoadSubtitle(videoFileHandle) {
      const videoFile = await videoFileHandle.getFile();
      const videoFileName = videoFile.name;
      const baseName = videoFileName.substring(0, videoFileName.lastIndexOf('.'));
      const subtitleExtensions = ['.srt', '.txt'];
      
      for (const ext of subtitleExtensions) {
        try {
          const subtitleFileName = baseName + ext;
          console.log("尝试加载字幕文件:", subtitleFileName);
          // 尝试在同一目录下获取字幕文件
          const subtitleFileHandle = await window.showOpenFilePicker({
            startIn: videoFileHandle,
            suggestedName: subtitleFileName,
            types: [{
              description: 'Subtitle Files',
              accept: { 'text/plain': [ext] }
            }],
            multiple: false,
          });
          
          if (subtitleFileHandle && subtitleFileHandle.length > 0) {
            const subtitleFile = await subtitleFileHandle[0].getFile();
            const content = await subtitleFile.text();
            this.subtitleContent = content;
            this.parseSubtitles(content);
            console.log('成功加载字幕文件:', subtitleFileName);
            return;
          }
        } catch (error) {
          // 文件不存在或用户取消选择，继续尝试下一个扩展名
        }
      }
      
      console.warn('未找到匹配的字幕文件');
    },
    
    parseSubtitles(content) {
      console.log("字幕内容:", content);
      if (!content || typeof content !== 'string') {
        console.error("无效的字幕内容");
        return;
      }
      const lines = content.trim().split('\n');
      this.parsedSubtitles = [];
      
      // 判断字幕格式
      const isSRT = lines[1].includes('-->');
      
      if (isSRT) {
        this.parseSRT(lines);
      } else {
        this.parseTXT(lines);
      }
          },
    
    parseSRT(lines) {
      let index = 0;
      while (index < lines.length) {
        if (lines[index + 1] && lines[index + 1].includes('-->')) {
          const timeRange = lines[index + 1].split(' --> ');
          const text = lines[index + 2];
          this.parsedSubtitles.push({
            id: lines[index],
            start: this.timeToSeconds(timeRange[0]),
            end: this.timeToSeconds(timeRange[1]),
            text: text.trim()
          });
          index += 4; // 跳过空行
        } else {
          index++;
        }
      }
    },
    
    parseTXT(lines) {
      const units = [];
      let currentUnit = null;

      lines.forEach(line => {
        const match = line.match(/\[(\d+)\]\[(.+?)->(.+?)\](.+)/);
        if (match) {
          if (currentUnit) {
            units.push(currentUnit);
          }
          currentUnit = {
            id: match[1],
            start: this.timeToSeconds(match[2]),
            end: this.timeToSeconds(match[3]),
            text: match[4].trim()
          };
        } else if (currentUnit) {
          currentUnit.text += ' ' + line.trim();
        }
      });

      if (currentUnit) {
        units.push(currentUnit);
      }

      this.parsedSubtitles = units;
    },
    
    timeToSeconds(timeString) {
      if (!timeString || typeof timeString !== 'string') {
        console.error(`无效的时间符串: ${timeString}`);
        return 0;
      }
      // 处理SRT格式的时间戳 (00:00:00,000)
      timeString = timeString.replace(',', '.');
      const [time, milliseconds] = timeString.split('.');
      const [hours, minutes, seconds] = time.split(':');
      return parseFloat(hours) * 3600 + parseFloat(minutes) * 60 + parseFloat(seconds) + parseFloat(milliseconds || 0) / 1000;
    },
    
    onSubtitleClick(time) {
      if (this.$refs.videoPlayer) {
        this.$refs.videoPlayer.setCurrentTimeAndPlay(time);
      } else {
        console.error('VideoPlayer 组件未找到');
      }
    },
    toggleLayout() {
      this.isHorizontalLayout = !this.isHorizontalLayout;
      this.$nextTick(() => {
        this.updateVideoPlayerSize();
      });
    },
    updateVideoPlayerSize() {
      if (this.$refs.videoPlayer) {
        this.$refs.videoPlayer.updateSize();
      }
    }
  }
}
</script>

<style>
@import "element-plus/dist/index.css";

#app {
  font-family: 'Helvetica Neue', Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #333;
  width: 100%;
  max-width: 1600px;
  margin: 0 auto;
  padding: 20px;
  box-sizing: border-box;
}

.control-panel {
  margin-bottom: 20px;
}
.video-file-name {
  margin-left: 10px;
  font-size: 14px;
  color: #606266;
}
.content-wrapper {
  display: flex;
  flex-direction: column;
  gap: 20px;
  height: calc(100vh - 100px);
  width: 100%;
  transition: all 0.3s ease;
}

.content-wrapper.horizontal-layout {
  flex-direction: row;
}

.video-container {
  width: 100%;
  height: 0;
  padding-top: 56.25%;
  position: relative;
  transition: all 0.3s ease;
}

.horizontal-layout .video-container {
  width: 70%;
  height: auto;
  padding-top: 39.375%;
  order: 2;
}

.video-player {
  position: absolute;
  top: 0;
  left: 0;
  width: 100% !important;
  height: 100% !important;
}

.article-component {
  width: 100%;
  height: calc(40vh - 50px);
  overflow-y: auto;
  border: 1px solid #ddd;
  border-radius: 5px;
  padding: 15px;
  box-sizing: border-box;
  transition: all 0.3s ease;
}

.horizontal-layout .article-component {
  width: 30%;
  height: calc(100vh - 100px);
  order: 1;
}

@media (max-width: 1200px) {
  #app {
    padding: 15px;
  }

  .horizontal-layout .video-container {
    width: 65%;
    padding-top: 36.5625%;
  }

  .horizontal-layout .article-component {
    width: 35%;
  }
}

@media (max-width: 992px) {
  .content-wrapper {
    height: auto;
  }

  .horizontal-layout .video-container {
    width: 60%;
    padding-top: 33.75%;
  }

  .horizontal-layout .article-component {
    width: 40%;
  }
}

@media (max-width: 768px) {
  #app {
    padding: 10px;
  }

  .content-wrapper,
  .content-wrapper.horizontal-layout {
    flex-direction: column;
    height: auto;
  }

  .video-container,
  .horizontal-layout .video-container {
    width: 100%;
    padding-top: 56.25%;
    order: 1;
  }

  .article-component,
  .horizontal-layout .article-component {
    width: 100%;
    height: 300px;
    order: 2;
  }
}

@media (max-width: 480px) {
  .el-button {
    padding: 8px 16px;
    font-size: 14px;
  }
}

.main-content {
  width: 100%;
  transition: none; /* 移除过渡效果 */
  margin-left: 0 !important; /* 强制保持左边距为0 */
}

.el-drawer__container {
  position: absolute !important;
}

.el-drawer {
  position: absolute !important;
}
</style>
