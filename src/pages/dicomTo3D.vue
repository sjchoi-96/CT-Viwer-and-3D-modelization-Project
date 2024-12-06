<template>
  <div>
    <!-- DICOM 파일 폴더 업로드 버튼 -->
    <input type="file" webkitdirectory @change="handleFolderUpload" />
    <!-- 3D 렌더링이 표시될 Canvas 영역 -->
    <div
      id="vtk-canvas"
      ref="vtkCanvas"
      style="width: 100%; height: 500px; border: 1px solid black"
    ></div>
  </div>
</template>

<script setup lang="ts">
definePage({
  meta: {
    icon: 'mdi-clipboard-text',
    title: 'dcmTo3D',
    drawerIndex: 6,
  },
})

import { ref, onMounted } from 'vue'
import type * as vtk from '@kitware/vtk.js/vtk.js'
import vtkGenericRenderWindow from '@kitware/vtk.js/Rendering/Misc/GenericRenderWindow'
import vtkHttpDataSetReader from '@kitware/vtk.js/IO/Core/HttpDataSetReader'
import vtkVolume from '@kitware/vtk.js/Rendering/Core/Volume'
import vtkVolumeMapper from '@kitware/vtk.js/Rendering/Core/VolumeMapper'
import vtkColorTransferFunction from '@kitware/vtk.js/Rendering/Core/ColorTransferFunction'
import vtkPiecewiseFunction from '@kitware/vtk.js/Common/DataModel/PiecewiseFunction'

// 상태 관리
const vtkCanvas = ref<HTMLDivElement | null>(null) // Canvas 영역 참조
let genericRenderWindow: vtkGenericRenderWindow | null = null

// 폴더 업로드 이벤트 처리
const handleFolderUpload = async (event: Event) => {
  const target = event.target as HTMLInputElement
  const files = target.files

  if (!files || files.length === 0) {
    console.error('폴더에서 파일을 선택해주세요.')
    return
  }

  // DICOM 파일 배열로 변환
  const dicomFiles = Array.from(files)
  console.log('dicomFiles in handleFolderUpload', dicomFiles)

  // VTK.js를 사용하여 렌더링
  await render3DModel(dicomFiles)
  console.log('render3DModel 호출 완료')
}

// 3D 렌더링 함수
const render3DModel = async (files: File[]) => {
  console.log('render3DModel 호출')
  try {
    // Generic Render Window 초기화
    console.log('vtkCanvas.value in render3DModel :', vtkCanvas.value)
    if (!vtkCanvas.value) {
      console.error('Canvas 요소를 찾을 수 없습니다.')
      return
    }

    // GenericRenderWindow 생성
    console.log('genericRenderWindow in render3DModel :', genericRenderWindow)
    if (!genericRenderWindow) {
      genericRenderWindow = vtkGenericRenderWindow.newInstance()
      genericRenderWindow.setContainer(vtkCanvas.value)
      console.log('generated genericRenderWindow in render3DModel')
    }

    const renderer = genericRenderWindow.getRenderer()
    const renderWindow = genericRenderWindow.getRenderWindow()
    console.log('renderer in render3DModel', renderer)
    console.log('renderWindow in render3DModel', renderWindow)

    // DICOM 데이터 로드 및 처리
    const reader = vtkHttpDataSetReader.newInstance()
    reader.setUrl(files[0].name) // 첫 번째 DICOM 파일 사용 (추가 확장 가능)
    await reader.loadData()

    const imageData = reader.getOutputData()

    // Volume Mapper 및 Volume 생성
    const volumeMapper = vtkVolumeMapper.newInstance()
    volumeMapper.setInputData(imageData)

    const volume = vtkVolume.newInstance()
    volume.setMapper(volumeMapper)

    // 색상 및 투명도 매핑
    const colorTransferFunction = vtkColorTransferFunction.newInstance()
    colorTransferFunction.addRGBPoint(0, 0.0, 0.0, 0.0)
    colorTransferFunction.addRGBPoint(255, 1.0, 1.0, 1.0)

    const opacityFunction = vtkPiecewiseFunction.newInstance()
    opacityFunction.addPoint(0, 0.0)
    opacityFunction.addPoint(255, 1.0)

    volume.getProperty().setRGBTransferFunction(0, colorTransferFunction)
    volume.getProperty().setScalarOpacity(0, opacityFunction)

    // 렌더링 및 추가
    renderer.addVolume(volume)
    renderer.resetCamera()
    renderWindow.render()
  } catch (error) {
    console.error('3D 렌더링 중 오류 발생:', error)
  }
}

// 컴포넌트 마운트 시 초기화
onMounted(() => {
  if (vtkCanvas.value) {
    console.log('VTK.js Canvas 초기화 완료')
  }
})
</script>

<style>
#vtk-canvas {
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
