<template>
  <div class="ct-viewer-container">
    <!-- 파일 입력 필드 -->
    <input
      id="file-input"
      type="file"
      accept=".dcm"
      multiple
      @change="handleFileChange"
    />

    <!-- SinglePlaneViewer 컴포넌트 -->
    <SinglePlaneViewer v-if="dicomData" :dicom-data="dicomData" />
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import SinglePlaneViewer from '../components/CT-Viewer/SinglePlaneViewer.vue'
import * as dicomParser from 'dicom-parser'

definePage({
  meta: {
    icon: 'mdi-clipboard-text',
    title: 'DICOM Viewer',
    drawerIndex: 5,
  },
})

// DICOM 데이터 상태 관리
const dicomData = ref<{ file: File; totalFrames: number } | null>(null)
console.log('초기 dicomData:', dicomData.value)

// DICOM 파일 로드 이벤트
const handleFileChange = async (event: Event) => {
  try {
    const target = event.target as HTMLInputElement
    const files = target.files
    if (!files || files.length === 0) {
      throw new Error('선택된 파일이 없습니다.')
    }
    await loadDicomFiles(files)
    console.log('DICOM 파일 로드 완료')
    console.log('로드 dicomData:', dicomData.value)
  } catch (error) {
    console.error('파일 로드 에러:', error)
  }
}

// DICOM 파일 로드 및 프레임 수 확인
const loadDicomFiles = async (files: FileList) => {
  try {
    const file = files[0]
    const arrayBuffer = await readFileAsArrayBuffer(file)
    console.log('arrayBuffer :', arrayBuffer)
    const dataSet = dicomParser.parseDicom(new Uint8Array(arrayBuffer))
    console.log('dataSet :', dataSet)
    if (!dataSet) {
      throw new Error('DICOM 파일 파싱 실패')
    }

    const totalFrames = parseInt(dataSet.string('x00280008') || '1')
    if (totalFrames <= 0) {
      throw new Error('유효하지 않은 프레임 수')
    }

    dicomData.value = { file, totalFrames }
  } catch (error) {
    console.error('DICOM 파일 처리 에러:', error)
    throw error
  }
}

// FileReader Promise 래퍼
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
    console.log('파일 읽기 시작')
  })
}
</script>

<style scoped>
.ct-viewer-container {
  padding: 20px;
}

input[type='file'] {
  display: block;
  margin: 20px auto;
}
</style>
