<template>
  <div class="single-plane-viewer">
    <div
      v-if="dicomData"
      id="dicom-viewer"
      ref="viewerElement"
      @dragover.prevent
      @wheel.prevent="handleMouseWheel"
    ></div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'
import * as cornerstone from 'cornerstone-core'
import * as cornerstoneWADOImageLoader from 'cornerstone-wado-image-loader'
import * as dicomParser from 'dicom-parser'

// Cornerstone 초기화
cornerstoneWADOImageLoader.external.cornerstone = cornerstone
cornerstoneWADOImageLoader.external.dicomParser = dicomParser

// Props 정의

interface DicomDataProps {
  file: File
  totalFrames: number
}

const props = defineProps<{ dicomData: DicomDataProps }>()

// 상태 관리
const viewerElement = ref<HTMLDivElement | null>(null)
const imageIds = ref<string[]>([])
const currentFrameIndex = ref(0)

// 이미지 로드 및 표시 로직
const loadImageIds: (
  file: File,
  totalFrames: number,
) => Promise<string[]> = async (file, totalFrames) => {
  const baseImageId = cornerstoneWADOImageLoader.wadouri.fileManager.add(file)
  return Array.from(
    { length: totalFrames },
    (_, i) => `${baseImageId}?frame=${i}`,
  )
}

const renderImage: (frameIndex: number) => Promise<void> = async (
  frameIndex,
) => {
  if (!viewerElement.value || imageIds.value.length === 0) {
    console.warn('Viewer element is not ready or no images available.')
    return
  }

  try {
    const imageId = imageIds.value[frameIndex]
    const image = await cornerstone.loadImage(imageId)
    cornerstone.displayImage(viewerElement.value, image)
    // console.log(`Frame ${frameIndex} displayed.`)
  } catch (error) {
    console.error('Error rendering image:', error)
  }
}

const updateFrameIndex: (delta: number) => Promise<void> = async (delta) => {
  const newIndex = currentFrameIndex.value + delta

  if (newIndex >= 0 && newIndex < imageIds.value.length) {
    currentFrameIndex.value = newIndex
    await renderImage(newIndex)
  } else {
    console.warn('Frame index out of bounds:', newIndex)
  }
}

// 이벤트 핸들러
const handleDicomDataChange: (
  newData: DicomDataProps,
) => Promise<void> = async (newData) => {
  if (!newData) return

  console.log('Loading DICOM data...')
  imageIds.value = await loadImageIds(newData.file, newData.totalFrames)

  if (imageIds.value.length > 0) {
    currentFrameIndex.value = 0
    await renderImage(0)
  }
}

const handleMouseWheel: (event: WheelEvent) => void = (event) => {
  const delta = Math.sign(event.deltaY)
  updateFrameIndex(delta)
}

// Cornerstone 활성화
const initializeCornerstone: () => void = () => {
  if (viewerElement.value) {
    cornerstone.enable(viewerElement.value)
    console.log('Cornerstone viewer enabled.')
  }
}

// DICOM 데이터 변경 감지
watch(
  () => props.dicomData,
  async (newData) => {
    if (newData) {
      await handleDicomDataChange(newData)
    }
  },
  { immediate: true },
)

// 컴포넌트 마운트 시 초기화
onMounted(() => {
  initializeCornerstone()
  if (props.dicomData) {
    handleDicomDataChange(props.dicomData)
  }
})
</script>

<style scoped>
#dicom-viewer {
  width: 512px;
  height: 512px;
  border: 1px solid #000;
  margin: 20px auto;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #333;
  color: white;
  text-align: center;
  overflow: hidden;
}
</style>
