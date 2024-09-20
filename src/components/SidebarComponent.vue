<template>
  <el-drawer
    v-model="drawerVisible"
    title="字幕列表"
    :size="300"
    :with-header="true"
    direction="ltr"
    :modal="false"
    :append-to-body="true"
  >
    <div class="sidebar-content">
      <el-input
        v-model="searchQuery"
        placeholder="搜索字幕..."
        clearable
        prefix-icon="el-icon-search"
        @input="filterSubtitles"
      />
      <div 
        v-for="(subtitle, index) in filteredSubtitles" 
        :key="index"
        class="subtitle-group"
      >
        <div 
          v-for="(unit, unitIndex) in splitSubtitleText(subtitle.text)" 
          :key="`${index}-${unitIndex}`"
          class="subtitle-unit"
          @click="onSubtitleClick(subtitle.start)"
        >
          {{ unit }}
        </div>
      </div>
    </div>
  </el-drawer>
</template>

<script>
import { ref, watch, computed } from 'vue'
import { ElInput } from 'element-plus'

export default {
  name: 'SidebarComponent',
  components: {
    ElInput
  },
  props: {
    subtitles: {
      type: Array,
      required: true
    },
    visible: {
      type: Boolean,
      required: true
    }
  },
  emits: ['update:visible', 'subtitle-click'],
  setup(props, { emit }) {
    const drawerVisible = ref(props.visible)
    const searchQuery = ref('')

    const filteredSubtitles = computed(() => {
      if (!searchQuery.value) {
        return props.subtitles
      }
      return props.subtitles.filter(subtitle => 
        subtitle.text.toLowerCase().includes(searchQuery.value.toLowerCase())
      )
    })

    const onSubtitleClick = (time) => {
      emit('subtitle-click', time)
    }

    const splitSubtitleText = (text) => {
      return text.split(/\s+/)
    }

    const filterSubtitles = () => {
      // 这个函数可以为空，因为我们使用了计算属性
    }

    watch(() => props.visible, (newValue) => {
      drawerVisible.value = newValue
    })

    watch(drawerVisible, (newValue) => {
      emit('update:visible', newValue)
    })

    return {
      drawerVisible,
      searchQuery,
      filteredSubtitles,
      onSubtitleClick,
      splitSubtitleText,
      filterSubtitles
    }
  }
}
</script>

<style scoped>
.sidebar-content {
  height: 100%;
  overflow-y: auto;
  padding: 20px;
}

.subtitle-group {
  margin-bottom: 10px;
}

.subtitle-unit {
  cursor: pointer;
  padding: 5px;
  border-radius: 3px;
  transition: background-color 0.3s ease;
}

.subtitle-unit:hover {
  background-color: #e0e0e0;
}

:deep(.el-drawer__container) {
  position: absolute !important;
}

:deep(.el-drawer) {
  position: absolute !important;
}

:deep(.el-overlay) {
  position: absolute !important;
}

.el-input {
  margin-bottom: 15px;
}
</style>