<template>
  <div class="home">
    <el-container>
      <el-header class="editor-header" height="60px">
        <el-row>
          <el-col :span="8">
            <el-button type="danger" round size="small" @click="init"
              >初始化</el-button
            >
            <el-button type="success" round size="small" @click="createPicture"
              >下载图片</el-button
            >
          </el-col>
          <el-col :span="8"></el-col>
          <el-col :span="8"></el-col>
        </el-row>
      </el-header>
      <el-container width="300px">
        <el-aside class="toolbar">
          <el-form ref="form" :model="configuration" label-width="80px">
            <el-form-item label="宽度:">
              <el-input
                v-model="configuration.width"
                clearable
                placeholder="图片宽度"
              ></el-input>
            </el-form-item>
            <el-form-item label="高度:">
              <el-input
                v-model="configuration.height"
                clearable
                placeholder="图片高度"
              ></el-input>
            </el-form-item>
            <el-form-item label="背景色">
              <span slot="label"
                >背景色
                <el-tooltip
                  class="item"
                  effect="dark"
                  content="背景色清空会生成透明底的图片"
                  placement="top-start"
                >
                  <i class="el-icon-question" style="color: #f56c6c"></i>
                </el-tooltip>
                :
              </span>
              <el-color-picker
                v-model="configuration.color"
                :predefine="predefineColors"
              >
              </el-color-picker>
            </el-form-item>
            <el-form-item label="文字:">
              <el-input
                v-model="configuration.text"
                placeholder="显示在图片内的文字"
                clearable
              ></el-input>
            </el-form-item>
            <el-form-item label="文字颜色:">
              <el-color-picker
                v-model="configuration.text_color"
              ></el-color-picker>
            </el-form-item>
          </el-form>
        </el-aside>
        <el-main class="picture-area">
          <div>
            <div
              id="diy-dom"
              v-if="isShowDom"
              :style="{
                width: configuration.width + 'px',
                height: configuration.height + 'px',
                lineHeight: configuration.height + 'px',
                backgroundColor: configuration.color,
                color: configuration.text_color,
                overflow: 'hidden',
              }"
            >
              {{ configuration.text }}
            </div>
          </div>
        </el-main>
      </el-container>
    </el-container>
  </div>
</template>

<script>
// @ is an alias to /src
import html2canvas from "html2canvas";

export default {
  name: "Home",
  data() {
    return {
      configuration: {},
      predefineColors: [
        "#ff4500",
        "#ff8c00",
        "#ffd700",
        "#90ee90",
        "#00C7BF",
        "#1e90ff",
        "#c71585",
      ],
    };
  },
  methods: {
    handleCanvasOptions(config) {
      let options = {
        backgroundColor: config.color,
      };
      return options;
    },
    init() {
      this.configuration = {
        width: "100",
        height: "100",
        color: "#ff8c00",
        text: "Good",
        text_color: "#ffffff",
      };
    },
    createPicture() {
      let _this = this;
      let options = this.handleCanvasOptions(this.configuration);
      html2canvas(document.getElementById("diy-dom"), options)
        .then(function (canvas) {
          canvas.style.display = "none";
          document.body.appendChild(canvas);
          return canvas;
        })
        .then((canvasDom) => {
          _this.downloadPicture(canvasDom);
        });
    },
    downloadPicture(canvasDom) {
      // 接下来将canvas转换成base64的url
      var url = canvasDom.toDataURL("image/png"); // 该方法获得的base64和上面提到echarts的API得到的一样
      // 注意该方法属于canvas元素，而不是创建的context对象
      var arr = url.split(","),
        mime = arr[0].match(/:(.*?);/)[1], // 此处得到的为文件类型
        bstr = atob(arr[1]), // 此处将base64解码
        n = bstr.length,
        u8arr = new Uint8Array(n);
      while (n--) {
        u8arr[n] = bstr.charCodeAt(n);
      }

      // 通过以下方式将以上变量生成文件对象，三个参数分别为文件内容、文件名、文件类型
      var file = new File([u8arr], "filename", { type: mime });
      var aDom = document.createElement("a"); // 创建一个 a 标签
      aDom.download = file.name; // 设置文件名
      let href = URL.createObjectURL(file); // 将file对象转成 UTF-16 字符串
      aDom.href = href; // 放入href
      document.body.appendChild(aDom); // 将a标签插入 body
      aDom.click(); // 触发 a 标签的点击
      document.body.removeChild(aDom); // 移除刚才插入的 a 标签
      document.body.removeChild(canvasDom); // 移除刚才插入的 a 标签
      URL.revokeObjectURL(href); // 释放刚才生成的 UTF-16 字符串
    },
  },
  components: {},
  computed: {
    isShowDom() {
      let verifyWidth = /^\+?[1-9][0-9]*$/.test(
        Number(this.configuration.width)
      );
      let verifyHeight = /^\+?[1-9][0-9]*$/.test(
        Number(this.configuration.height)
      );
      return verifyWidth && verifyHeight;
    },
  },
  mounted() {
    this.init();
  },
};
</script>
<style lang="scss" scoped>
.home {
  width: 100vw;
  // height: 100vh;
  overflow: hidden;
}
.editor-header {
  border-bottom: 1px solid #dcdfe6;
  padding: 10px 0;
}
.toolbar {
  height: calc(100vh - 60px);
  padding: 20px 10px 0px;
}
.picture-area {
  background-color: #f5f5f5;
  height: calc(100vh - 60px);
}
</style>
