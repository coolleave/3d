<template>
  <div class="video-container">
    <video
      ref="videoRef"
      class="video-player"
      :src="currentVideo"
      controls
      @ended="handleEnded"
    ></video>

    <!-- 显示选择按钮 -->
    <div v-if="showChoices" class="video-overlay">
      <button @click="switchVideo('/videos/branch1.mp4')">选择剧情 1</button>
      <button @click="switchVideo('/videos/branch2.mp4')">选择剧情 2</button>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, onMounted } from 'vue'

const props = defineProps({
  videoSrc: {
    type: String,
    required: true,
  },
})

const videoRef = ref(null)
const currentVideo = ref(props.videoSrc)
const showChoices = ref(false)

// 自动在第一个视频结束时展示选择
const handleEnded = () => {
  showChoices.value = true
}

// 切换到新的视频
const switchVideo = (newSrc) => {
  currentVideo.value = newSrc
  showChoices.value = false

  // 自动播放新视频
  nextTick(() => {
    videoRef.value.load()
    videoRef.value.play()
  })
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
}

.video-overlay button {
  padding: 0.5rem 1.2rem;
  font-size: 1rem;
  cursor: pointer;
}
</style>
