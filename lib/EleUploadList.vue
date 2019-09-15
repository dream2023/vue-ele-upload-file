<template>
  <transition-group
    class="ele-upload-file-list el-upload-list el-upload-list--text"
    name="el-list"
    tag="ul"
  >
    <li
      :key="file.uid"
      class="el-upload-list__item ele-upload-list__item-content"
      v-for="(file, index) in files"
    >
      <el-link
        :href="file.url"
        :underline="false"
        target="_blank"
      >
        <img
          :src="getExtension(file.name)"
          class="el-upload-list__item-content-name-icon"
        />
        <span>{{file.name}}</span>
      </el-link>
      <div class="ele-upload-list__item-content-action">
        <el-link
          :underline="false"
          v-if="isShowSize"
        >{{getSize(file.size)}}</el-link>
        <el-link
          :underline="false"
          @click="handleDownload(file)"
          v-if="isCanDownload"
        >下载</el-link>
        <el-link
          :underline="false"
          @click="handleDelete(index, file)"
          type="danger"
          v-if="!disabled && isCanDelete"
        >删除</el-link>
      </div>
    </li>
  </transition-group>
</template>
<script>
import download from 'ly-downloader'
import iconList from './iconList'
const prettyBytes = require('pretty-bytes')

export default {
  name: 'EleUploadList',
  props: {
    // 文件列表
    files: {
      type: Array,
      default() {
        return [];
      }
    },
    // 是否禁用
    disabled: {
      type: Boolean,
      default: false
    },
    // 是否显示文件大小
    isShowSize: {
      type: Boolean,
      default: true
    },
    // 是否显示下载
    isCanDownload: {
      type: Boolean,
      default: true
    },
    // 是否可删除
    isCanDelete: {
      type: Boolean,
      default: true
    }
  },
  methods: {
    // 获取扩展
    getExtension (name) {
      if (name.lastIndexOf('.') > -1) {
        const extension = name.slice(name.lastIndexOf('.') + 1).toLowerCase()
        return iconList[extension] || iconList['file']
      } else {
        return iconList['file']
      }
    },
    // 获取文件大小
    getSize (size) {
      return size && !isNaN(Number(size)) ? prettyBytes(size) : ''
    },
    // 上传进度
    parsePercentage(val) {
      return parseInt(val, 10)
    },
    // 下载文件
    handleDownload (file) {
      if (file.url) {
        download(1, file.url, file.name)
      }
    },
    // 删除文件
    handleDelete(index, file) {
      this.$confirm('确认删除该文件吗? ').then(() => {
        this.$emit('remove', index, file)
      }).catch(() => {})
    }
  }
}
</script>

<style>
.ele-upload-file-list .el-upload-list__item {
  border: 1px solid #e4e7ed;
  line-height: 2;
  margin-bottom: 10px;
  position: relative;
}

.ele-upload-file-list .ele-upload-list__item-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: inherit;
}

.ele-upload-file-list .el-upload-list__item-content-name-icon {
  padding-left: 8px;
  padding-right: 5px;
  width: 20px;
  vertical-align: text-top;
  border-radius: 4px;
}

.ele-upload-list__item-content-action .el-link {
  margin-right: 10px;
}
</style>