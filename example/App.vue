<template>
  <div style="max-width: 800px;margin: 50px auto">
    <ele-upload-file
      :disabled="false"
      :file-type="['svg', 'html', 'js', 'css', 'doc', 'xls', 'zip']"
      :fileSize="0.2"
      :isCanDelete="true"
      :isCanDownload="true"
      :isCanUploadSame="true"
      :isShowDeleteConfim="false"
      :isShowSize="true"
      :isShowTip="true"
      :limit="3"
      :multiple="true"
      :responseFn="handleResponse"
      @remove="handleRemove"
      action="https://jsonplaceholder.typicode.com/posts/"
      placeholder="上传附件"
      v-model="file"
    />
  </div>
</template>
<script>
export default {
  data () {
    return {
      file: { name: 'test.jpg', url: 'https://www.baidu.com' }
    }
  },
  methods: {
    handleRemove (file, fileList) {
      // eslint-disable-next-line no-console
      console.log(file, fileList)
    },
    handleResponse (response, file) {
      return {
        url: URL.createObjectURL(file.raw),
        name: file.name,
        size: file.size
      }
    }
  }
}
</script>
