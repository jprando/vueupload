<script setup lang="ts">

// TODO validar arquivos com tamanho 0 bytes
// TODO validar newName duplicados

import type { Component } from 'vue';
import { ref, reactive, computed, nextTick } from 'vue';
import { FileCheck, FileDocument, FileMusic, FileSpreadsheet, FileStar, FileText, FileZip } from "./components/icons"

// TYPEs
type UploadFile = { editMode?: boolean, sizeHuman?: string; newName?: string; editNewName?: string; extension?: string; originalFileName: string; file: File };
type StateType = {
  dragover: boolean;
  files: UploadFile[];
}
// CONSTs & DATA & REFs
const units = ['B', 'KB', 'MB', 'GB'];
const fileTypes: { [typeName: string]: Component } = {
  "application/json": FileText,
  "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet": FileSpreadsheet,
  "application/x-zip-compressed": FileZip,
  "video/mp4": FileMusic,
  "video/webm": FileMusic,
  "image/jpeg": FileStar,
  "image/png": FileStar,
  "image/svg+xml": FileStar,
}
const fileNameInput = ref(null);
const state = reactive<StateType>({
  dragover: false,
  files: [],
})
// FUNCTIONs
function fileSizeHuman(value: number): string {
  let idx = 0;
  while (value > 1024) {
    idx++;
    value = value / 1024;
  };
  const unit = units[idx] || 'GB'
  return `${idx === 0 ? value : value.toFixed(2)} ${unit}`;
}
function dropHandler(event: DragEvent) {
  event.preventDefault();
  state.dragover = false;
  if (!event.dataTransfer) return;
  const dataTransfer = event.dataTransfer;
  if (dataTransfer.items) {
    for (let idx = 0; idx < dataTransfer.items.length; idx++) {
      const dataItem: DataTransferItem = dataTransfer.items[idx];
      if (dataItem.kind !== 'file') continue;
      const fileItem = dataItem;
      // console.log("#24 fileItem", fileItem)
      const file = fileItem.getAsFile()
      if (!file || !file.size) continue; // file.size igual a zero é uma pasta
      let fileNameSplit: string[] = file.name.split('.');
      const fileInfo: UploadFile = {
        editMode: false,
        originalFileName: file.name,
        newName: fileNameSplit.slice(0, -1).join('.'),
        extension: fileNameSplit.slice(-1).join(''),
        sizeHuman: fileSizeHuman(file.size),
        file
      }
      fileInfo.editNewName = fileInfo.newName
      if (fileInfo.newName === fileInfo.originalFileName && fileInfo.newName === fileInfo.extension) {
        fileInfo.extension = "";
      }
      state.files.push(fileInfo)
      // console.log("#48 file.type", file.type)
    }
  } else {
    for (let idx = 0; idx < dataTransfer.files.length; idx++) {
      const fileItem = dataTransfer.files[idx];
      // console.log("#33 fileItem", fileItem)
      let fileNameSplit: string[] = fileItem.name.split('.');
      const fileInfo: UploadFile = {
        editMode: false,
        originalFileName: fileItem.name,
        newName: fileNameSplit.slice(0, -1).join('.'),
        extension: fileNameSplit.slice(-1).join(''),
        sizeHuman: fileSizeHuman(fileItem.size),
        file: fileItem,
      }
      fileInfo.editNewName = fileInfo.newName
      state.files.push(fileInfo)
    }
  }
}
function dragoverHandle(event: DragEvent) {
  event.preventDefault();
  state.dragover = true;
}
function dragenterHandler(event: DragEvent) {
  event.preventDefault();
  state.dragover = true;
}
function dragleaveHandler(event: DragEvent) {
  event.preventDefault();
  state.dragover = false;
}
function enterEditMode(file: UploadFile) {
  file.editNewName = file.newName;
  file.editMode = true;
  nextTick(() => {
    // console.log("#101 fileNameInput.value", fileNameInput.value)
    if (fileNameInput.value && Array.isArray(fileNameInput.value) && fileNameInput.value[0]) {
      (fileNameInput.value[0] as any).focus();
    }
  })
}
// COMPUTEDs
const dropZoneStyle = computed(() => ({
  borderStyle: state.dragover ? 'dashed' : 'solid',
  borderWidth: state.dragover ? '2px' : '2px',
  borderColor: state.dragover ? 'red' : 'transparent',
}))
</script>

<template>
  <main style="height:300px;width:600px" :style="dropZoneStyle">

    <section style="height:100%;width:100%;overflow-y: auto;" @drop="dropHandler($event)"
      @dragover="dragoverHandle($event)" @dragenter="dragenterHandler($event)" @dragleave="dragleaveHandler($event)">

      <p> Arraste e solte um ou mais arquivos aqui.</p>

      <p v-for="file in state.files" :key="`${file.newName}#${file.editMode}`"
        style="display:flex;flex-direction: row;align-items: center;">

        <component :is="fileTypes[file.file.type] || FileDocument" style="padding-right:4px;width:32px"></component>
        <span style="padding-right:12px;width:120px;text-align:left">{{ file.sizeHuman }}</span>

        <input ref="fileNameInput" v-if="file.editMode" type="text" style="width:100%" v-model="file.editNewName"
          @keydown.enter="file.editMode = false; if (file.editNewName?.trim()) { file.newName = file.editNewName }"
          @keydown.esc="file.editMode = false;" @blur="file.editMode = false;" />
      <div v-else style="width:100%;text-align:left;cursor: pointer;user-select: none;"
        title="clique para editar o nome do arquivo" @click="enterEditMode(file)">
        {{ file.newName }}{{ file.extension ? '.' : '' }}{{ file.extension ? file.extension : '' }}
      </div>
      <!-- <button @click="file.editMode = true">renomear</button> -->

      </p>

    </section>

  </main>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}

.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}

.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
