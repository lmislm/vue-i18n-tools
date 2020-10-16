<template>
  <div class="toolsBox">
    <el-button
      type="primary"
      size="mini"
      round
      @click="handleOpen"
      icon="el-icon-question"
      style="user-select: none"
      >查看readme</el-button
    >
  </div>
</template>

<script>
export default {
  computed: {
    infoList() {
      const infos = [
        [
          "输入文件源码",
          "源代码粘贴到文本框中，点击“寻找中文字符串”，工具会将vue文件中所有“中文”字符串全部找出。",
        ],
        [
          "选择国际化的文本",
          "点击右侧的复选框，选择是否要国际化。点击输入框，定位源代码处，输入国际化的文本。",
        ],
        ["设置vue-i18n的key", "为选择好的国际化文本设置vue-i18n的key。"],
        [
          "生成文件和源码",
          "点击替换，右侧内容即可生成国际化文件，同时左侧还会生成国际化后的源码。",
        ],
      ];
      const fInfos = infos.map((item, index) => {
        return [`${index + 1}. ${item[0]}`, item[1]];
      });
      return fInfos;
    },
  },
  methods: {
    handleOpen() {
      //   this.isReadmeVisible = true
      const h = this.$createElement;
      this.$msgbox({
        title: "Readme",
        message: h(
          "ul",
          null,
          this.infoList.map((item) => {
            return [
              h("li", null, [h("h5", null, item[0]), h("span", null, item[1])]),
            ];
          })
        ),
        confirmButtonText: "确定",
      });
    },
  },
};
</script>

<style scoped>
.toolsBox {
  padding: 10px 20px 0;
}
.github-btn-group {
  vertical-align: middle;
  line-height: 0;
  float: right;
}
</style>