<template>
    <div class="article-container" ref="articleContainer">
      <div v-if="!subtitles.length" class="no-subtitles">
        暂无字幕数据
      </div>
      <div v-else class="subtitle-line">
        <span
          v-for="(subtitle, index) in subtitles"
          :key="index"
          class="subtitle-unit"
          :class="{ 'active': isSubtitleActive(subtitle) }"
          @click="handleSubtitleClick(subtitle)"
        >
          {{ subtitle.text }}
        </span>
      </div>
    </div>
</template>

<script>
export default {
  name: 'ArticleComponent',
  props: {
    subtitles: {
      type: Array,
      required: true
    },
    currentTime: {
      type: Number,
      default: 0
    }
  },
  setup(props, { emit }) {
    const handleSubtitleClick = (subtitle) => {
      emit('subtitle-click', subtitle.start)
    }
    const isSubtitleActive = (subtitle) => {
      return props.currentTime >= subtitle.start && props.currentTime < subtitle.end
    }
    return {
      handleSubtitleClick,
      isSubtitleActive
    }
  }
}
</script>

<style scoped>
.no-subtitles {
  text-align: center;
  padding: 20px;
  color: #999;
}
.article-container {
  height: 100%;
  overflow-y: auto;
  padding: 10px;
}

.subtitle-line {
  display: flex;
  flex-wrap: wrap;
}
.subtitle-unit.active {
  color: red;
}
.subtitle-unit {
  display: inline-block;
  cursor: pointer;
  padding: 2px 4px;
  transition: background-color 0.3s ease;
  overflow-wrap: break-word;
  word-wrap: break-word;
  margin: 0;
  border-radius: 0;
  font-size: 16px; /* 添加这一行来设置字体大小 */
}

.subtitle-unit:hover {
  background-color: #e0e0e0;
}

.subtitle-unit:first-child {
  border-top-left-radius: 3px;
  border-bottom-left-radius: 3px;
}

.subtitle-unit:last-child {
  border-top-right-radius: 3px;
  border-bottom-right-radius: 3px;
}

.subtitle-unit + .subtitle-unit {
  border-left: 1px solid #e0e0e0;
}

.popover-content {
  display: flex;
  padding: 0;
}

:deep(.el-button--text) {
  padding: 2px 5px !important; /* 增加按钮的可点击区域 */
  margin: 0 2px !important; /* 增加按钮之间的间距 */
}

:deep(.el-popover) {
  padding: 3px;
  min-width: auto;
}

/* 添加新的样式 */
:deep(.el-button--text:first-child) {
  margin-right: 1px;
}

:deep(.el-button--text:hover) {
  background-color: #f0f0f0;
  border-radius: 2px;
}

:deep(.subtitle-popover) {
  position: fixed !important;
  padding: 5px !important; /* 添加内边距 */
}
</style>