<template>
  <!-- DICOM 뷰어 -->
  <div
    id="dicom-viewer"
    ref="viewerElement"
    @dragover.prevent
    @wheel.prevent="handleMouseWheel"
  ></div>

  <!-- 파일 입력 필드 -->
  <input
    type="file"
    id="file-input"
    accept=".dcm"
    multiple
    @change="handleFileChange"
  />
</template>

<script setup lang="ts">
definePage({
  meta: {
    icon: 'mdi-clipboard-text',
    title: 'Multi-Plane Viewer',
    drawerIndex: 5,
  },
})
import { ref, onMounted } from 'vue'
import * as cornerstone from 'cornerstone-core'
import * as cornerstoneWADOImageLoader from 'cornerstone-wado-image-loader'
import * as dicomParser from 'dicom-parser'

// TypeScript 타입 정의 확장
declare module 'cornerstone-wado-image-loader' {
  interface External {
    cornerstone: typeof cornerstone
    dicomParser: typeof dicomParser
  }
  const external: External
}

// Cornerstone 초기화
cornerstoneWADOImageLoader.external.cornerstone = cornerstone
cornerstoneWADOImageLoader.external.dicomParser = dicomParser

// DOM 요소 참조
const viewerElement = ref<HTMLDivElement | null>(null)

// 이미지 스택 관리
const imageIds = ref<string[]>([])
const currentIndex = ref(0)
const totalFrames = ref(0)

// DICOM 파일 로드 이벤트 (필요한 변수 : event), (생성되는 변수 : target, files)
const handleFileChange = async (event: Event) => {
  try {
    const target = event.target as HTMLInputElement
    const files = target.files
    if (!files || files.length === 0) {
      throw new Error('선택된 파일이 없습니다.')
    }
    await loadDicomFiles(files)
  } catch (error) {
    console.error('파일 로드 에러:', error)
    // 여기에 사용자에게 에러를 표시하는 로직 추가 가능
  }
}

// DICOM 파일 로드 및 프레임 수 확인 (필요한 변수 : files), (생성되는 변수 : file, arrayBuffer, dataSet, totalFrames)
const loadDicomFiles = async (files: FileList) => {
  try {
    const file = files[0]
    const arrayBuffer = await readFileAsArrayBuffer(file)

    const dataSet = dicomParser.parseDicom(new Uint8Array(arrayBuffer))
    if (!dataSet) {
      throw new Error('DICOM 파일 파싱 실패')
    }

    totalFrames.value = parseInt(dataSet.string('x00280008') || '1')
    if (totalFrames.value <= 0) {
      throw new Error('유효하지 않은 프레임 수')
    }
    console.log('Total frames:', totalFrames.value)

    // 기본 imageId 생성
    const baseImageId = (
      cornerstoneWADOImageLoader as any
    ).wadouri.fileManager.add(file)
    if (!baseImageId) {
      throw new Error('imageId 생성 실패')
    }

    // 각 프레임별 imageId 생성
    imageIds.value = Array.from({ length: totalFrames.value }, (_, i) => {
      return `${baseImageId}?frame=${i}`
    })

    console.log('Generated imageIds:', imageIds.value)
    await displayCurrentImage()
  } catch (error) {
    console.error('DICOM 파일 처리 에러:', error)
    throw error // 상위 핸들러로 에러 전파
  }
}

// FileReader Promise 래퍼 (필요한 변수 : file), (생성되는 변수 : reader.result)
const readFileAsArrayBuffer = (file: File): Promise<ArrayBuffer> => {
  return new Promise((resolve, reject) => {
    const reader = new FileReader()
    reader.onload = () => {
      if (reader.result instanceof ArrayBuffer) {
        resolve(reader.result)
      } else {
        reject(new Error('파일 읽기 실패'))
      }
    }
    reader.onerror = () => reject(reader.error)
    reader.readAsArrayBuffer(file)
  })
}

// 현재 이미지 표시 (필요한 변수 : viewerElement, imageIds, currentIndex), (생성되는 변수 : imageId, image)
const displayCurrentImage = async () => {
  try {
    if (!viewerElement.value) {
      throw new Error('뷰어 엘리먼트가 없습니다.')
    }
    if (imageIds.value.length === 0) {
      throw new Error('표시할 이미지가 없습니다.')
    }

    const imageId = imageIds.value[currentIndex.value]
    const image = await cornerstone.loadImage(imageId)
    if (!image) {
      throw new Error('이미지 로드 실패')
    }

    cornerstone.displayImage(viewerElement.value, image)
    console.log('Successfully displayed frame:', currentIndex.value)
  } catch (error) {
    console.error('이미지 표시 에러:', error)
    throw error
  }
}

// 마우스 휠 이벤트로 이미지 전환 (필요한 변수 : totalFrames, currentIndex), (생성되는 변수 : newIndex)
const handleMouseWheel = async (event: WheelEvent) => {
  try {
    if (totalFrames.value === 0) return

    event.preventDefault()
    const delta = Math.sign(event.deltaY)

    const newIndex = currentIndex.value + delta
    if (newIndex >= 0 && newIndex < totalFrames.value) {
      currentIndex.value = newIndex
      await displayCurrentImage()
    } else {
      console.log('currentIndex out of range:', newIndex)
    }
  } catch (error) {
    console.error('마우스 휠 이벤트 처리 에러:', error)
  }
}

// DOM 요소 Cornerstone 활성화 (필요한 변수 : viewerElement)
onMounted(() => {
  try {
    if (!viewerElement.value) {
      throw new Error('viewerElement가 null입니다.')
    }
    console.log('Enabling Cornerstone...')
    cornerstone.enable(viewerElement.value)
  } catch (error) {
    console.error('Cornerstone 초기화 에러:', error)
  }
})
</script>

<style>
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
input[type='file'] {
  display: block;
  margin: 20px auto;
}
</style>
