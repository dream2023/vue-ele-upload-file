<template>
  <div class="ele-upload-file">
    <el-upload
      :before-upload="handleBeforeUpload"
      :drag="false"
      :file-list="fileList"
      :limit="limit"
      :multiple="multiple"
      :on-change="handleChange"
      :on-error="handleUploadError"
      :on-exceed="handleExceed"
      v-bind="$attrs"
      :on-success="handleUploadSuccess"
      :show-file-list="false"
      class="ele-upload-file-uploader"
      ref="upload"
      v-if="!disabled"
    >
      <!-- 上传按钮 -->
      <el-button size="medium" type="primary">{{ btnText }}</el-button>
      <!-- 上传提示 -->
      <div class="el-upload__tip" slot="tip" v-if="showTip">
        请上传
        <template v-if="fileSize">
          大小不超过
          <b style="color: #f56c6c;">{{ fileSize }}MB</b>
        </template>
        <template v-if="fileType">
          格式为
          <b style="color: #f56c6c;">{{ fileType.join("/") }}</b>
        </template>
        的文件
      </div>
    </el-upload>

    <!-- 列表 -->
    <ele-upload-list
      :disabled="disabled"
      :files="list"
      :isCanDelete="isCanDelete"
      :isCanDownload="isCanDownload"
      :isShowDeleteConfim="isShowDeleteConfim"
      :isShowSize="isShowSize"
      @remove="handleRemove"
    />
  </div>
</template>

<script>
import EleUploadList from "./EleUploadList";
export default {
  inheritAttrs: false,
  name: "EleUploadFile",
  components: {
    EleUploadList
  },
  props: {
    // 值
    value: [String, Object, Array],
    // 大小限制(MB)
    fileSize: Number,
    // 响应处理函数
    responseFn: Function,
    // 文件类型, 例如['png', 'jpg', 'jpeg']
    fileType: Array,
    // 提示
    placeholder: String,
    // 是否禁用
    disabled: Boolean,
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
    },
    // 是否显示删除确认弹窗
    isShowDeleteConfim: {
      type: Boolean,
      default: true
    },
    // 是否可上传相同文件
    isCanUploadSame: {
      type: Boolean,
      default: true
    },
    // 是否显示提示
    isShowTip: {
      type: Boolean,
      default: true
    },
    // 是否显示上传成功的提示
    isShowSuccessTip: {
      type: Boolean,
      default: true
    },
    // 是否支持多选文件
    // 同 element-ui upload 组件
    multiple: {
      type: Boolean,
      default: true
    },
    // 最大允许上传个数
    // 同 element-ui upload 组件
    limit: Number,
    // 删除前的操作
    // 同 element-ui upload 组件
    beforeRemove: Function
  },
  data() {
    return {
      fileList: []
    };
  },
  computed: {
    // 按钮文本
    btnText() {
      if (this.placeholder) {
        return this.placeholder;
      } else {
        if (this.multiple) {
          return "上传文件";
        } else {
          return "上传单个文件";
        }
      }
    },
    // 是否显示提示
    showTip() {
      return this.isShowTip && (this.fileType || this.fileSize);
    },
    // 列表
    list() {
      let temp = 1;
      if (this.value) {
        // 首先将值转为数组
        const list = Array.isArray(this.value) ? this.value : [this.value];
        // 然后将数组转为对象数组
        return list.map(item => {
          if (typeof item === "string") {
            item = { name: item, url: item };
          }
          item.uid = item.uid || new Date().getTime() + temp++;
          return item;
        });
      } else {
        return [];
      }
    }
  },
  methods: {
    // 文件改变
    handleChange(file, fileList) {
      this.fileList = fileList;
    },
    // 上传前校检格式和大小
    handleBeforeUpload(file) {
      // 校检文件类型
      if (this.fileType) {
        let fileExtension = "";
        if (file.name.lastIndexOf(".") > -1) {
          fileExtension = file.name.slice(file.name.lastIndexOf(".") + 1);
        }
        const isTypeOk = this.fileType.some(type => {
          if (file.type.indexOf(type) > -1) return true;
          if (fileExtension && fileExtension.indexOf(type) > -1) return true;
          return false;
        });

        if (!isTypeOk) {
          this.$message.error(
            `文件格式不正确, 请上传${this.fileType.join("/")}格式文件!`
          );
          return false;
        }
      }

      // 校检文件大小
      if (this.fileSize) {
        const isLt = file.size / 1024 / 1024 < this.fileSize;
        if (!isLt) {
          this.$message.error(`上传文件大小不能超过 ${this.fileSize} MB!`);
          return false;
        }
      }

      // 校检相同文件
      if (!this.isCanUploadSame) {
        const isSame = this.list.some(
          item => item.name + item.size === file.name + file.size
        );
        if (isSame) {
          this.$message.error(`此文件已上传!`);
          return false;
        }
      }
      return true;
    },
    // 文件个数超出
    handleExceed() {
      this.$message.error(`最多上传${this.limit}个文件`);
    },
    // 上传失败
    handleUploadError(err) {
      this.$message.error("上传失败, 请重试");
      this.$emit("error", err);
    },
    // 上传成功回调
    handleUploadSuccess(response, file) {
      if (this.isShowSuccessTip) {
        this.$message.success("上传成功");
      }
      if (this.responseFn) {
        response = this.responseFn(response, file, this.list);
      }
      if (this.multiple) {
        this.$emit("input", [...this.list, response]);
      } else {
        this.$emit("input", response);
      }

      // 上传成功
      this.$emit("success", response, this.list);
    },
    handleRemove(index) {
      if (!this.beforeRemove) {
        this.doRemove(index);
      } else if (typeof this.beforeRemove === "function") {
        const before = this.beforeRemove(this.list[index], this.list);
        if (before && before.then) {
          before.then(
            () => {
              this.doRemove(index);
            },
            () => {}
          );
        } else if (before !== false) {
          this.doRemove(index);
        }
      }
    },
    // 删除
    doRemove(index) {
      this.$emit("remove", this.list[index], this.list);
      this.fileList.splice(index, 1);
      if (this.multiple) {
        const data = [...this.list];
        data.splice(index, 1);
        this.$emit("input", data || []);
      } else {
        this.$emit("input", null);
      }
    }
  },
  created() {
    this.fileList = this.list;
  }
};
</script>

<style>
.ele-upload-file-uploader {
  margin-bottom: 5px;
}
</style>
