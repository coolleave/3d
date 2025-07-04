<template>
  <div class="video-container">
    <video
      v-if="currentVideoNode"
      ref="videoRef"
      class="video-player"
      :src="currentVideoNode.src"
       controls
       @ended="handleEnded"
    ></video>




    <!-- 当前节点的分支选项 -->
    <div v-if="showChoices && currentVideoNode.choices" class="video-overlay">
      <button
        v-for="choice in currentVideoNode.choices"
        :key="choice.label"
        @click="switchToNode(choice.target)"
      >
        {{ choice.label }}
      </button>
    </div>

    <!-- 黑色转场动画 -->
    <transition name="fade">
      <div v-if="isTransitioning" class="black-overlay"></div>
    </transition>

    <!-- 预加载视频资源 -->
   <!-- 预加载：使用 template 包裹 -->
    <template v-for="(node, key) in storyMap">
      <video
        v-if="currentVideoNode.src && node.src !== currentVideoNode.src"
        :key="key"
        :src="node.src"
        preload="auto"
        style="display: none;"
      />
    </template>

  </div>
</template>

<script setup>
import { ref, computed, nextTick } from 'vue'
// v-if="currentVideoNode && node.src !== currentVideoNode.src"

// 播放视频
const playVideo = () => {
  if (videoRef.value) {
    videoRef.value.play()
  }
}
// 剧情图谱：定义视频分支
const storyMap = {
  intro: {
    src: '/start.mp4',
    choices: [
      { label: '揭秘真相', target: '1' },
    ],
  },
  '1': {
    src: '/1.mp4',
    choices: [
      { label: '返回选择', target: 'intro' },
      { label: '进入剧情 2', target: '2' },
    ],
  },
  '2': {
    src: '/2.mp4',
    choices: [
      { label: '返回选择', target: 'intro' },
      { label: '进入剧情 3', target: '3' },
    ],
  },
}

// 当前剧情节点的 key
const currentNodeKey = ref('intro')
const currentVideoNode = computed(() => storyMap[currentNodeKey.value] || {})


const videoRef = ref(null)
const showChoices = ref(false)
const isTransitioning = ref(false)

// 播放结束后展示选择
const handleEnded = () => {
  if (currentVideoNode.value.choices) {
    showChoices.value = true
  }
}

// 切换到新节点
const switchToNode = async (targetKey) => {
  isTransitioning.value = true
  await new Promise((resolve) => setTimeout(resolve, 500))

  currentNodeKey.value = targetKey
  showChoices.value = false

  await nextTick()

  if (videoRef.value) {
    videoRef.value.load()
    const onReady = () => {
      videoRef.value.play()
      videoRef.value.removeEventListener('canplay', onReady)
      setTimeout(() => {
        isTransitioning.value = false
      }, 300)
    }
    videoRef.value.addEventListener('canplay', onReady)
  }
}

</script>

<style scoped>
.video-container {
  position: relative;
  max-width: 800px;
  margin: auto;
}

.video-player {
  width: 100%;
  height: auto;
}

.video-overlay {
  position: absolute;
  top: 30%;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 1rem;
  z-index: 2;
}

.video-overlay button {
  padding: 0.5rem 1.2rem;
  font-size: 1rem;
  cursor: pointer;
}

/* 黑色遮罩淡入淡出 */
.black-overlay {
  position: absolute;
  inset: 0;
  background: black;
  z-index: 3;
}

.fade-enter-active, .fade-leave-active {
  transition: opacity 0.5s ease;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
}
</style>
