<template>
  <div class="main">
    <template v-if="keyCode">
      <el-row :gutter="16">
        <el-col :span="12">
          <pre class="content" v-html="resultCode"></pre>
        </el-col>
        <el-col :span="12">
          <pre class="content" v-if="!pageName">{{ keyCode }}</pre>
          <pre
            class="content"
            v-else
            v-html="JSON.stringify(keyCode, null, 2)"
          ></pre>
        </el-col>
      </el-row>
      <el-row>
        <el-button type="danger" style="width: 100%" @click="goback2"
          >返回</el-button
        >
      </el-row>
    </template>
    <template v-else-if="findWordArr.length > 0">
      <div style="user-select: none">
        <el-input v-model="pageName" size="small"
          ><span slot="prepend">页面namespace</span></el-input
        >
      </div>
      <el-row :gutter="16">
        <el-col :span="12">
          <pre class="result" v-html="showReplaceCodeHTML" key="result"></pre>
        </el-col>
        <el-col :span="12">
          <ul class="word-list">
            <li
              class="word-list-item"
              v-for="word in findWordArr"
              :key="word.index"
            >
              <el-checkbox v-model="word.used"
                >{{ word.index + 1 }}. {{ word.word }}</el-checkbox
              >
              <p class="word-list-item-key">
                <el-input
                  size="small"
                  v-if="word.used"
                  @focus="focus(word.index)"
                  @blur="blur(word.index)"
                  placeholder="请输入该国际化文本的key"
                  v-model="word.key"
                />
              </p>
            </li>
          </ul>
        </el-col>
      </el-row>
      <el-row :gutter="16">
        <el-col :span="12">
          <el-button type="danger" @click="goback" style="width: 100%"
            >返回</el-button
          >
        </el-col>
        <!-- <el-col :span="8">
          <el-button @click="getKey"> 生成未完成的KEY </el-button>
        </el-col> -->
        <el-col :span="12">
          <el-tooltip
            effect="dark"
            v-if="!replaceDisable"
            content="请输入所有国际化文本的key"
            placement="top"
          >
            <div>
              <el-button :disabled="true" style="width: 100%" type="primary">
                替换
              </el-button>
            </div>
          </el-tooltip>
          <el-button v-else style="width: 100%" type="primary" @click="replace"
            >替换</el-button
          >
        </el-col>
      </el-row>
    </template>
    <template v-else>
      <div class="inputCode">
        <el-input
          type="textarea"
          placeholder="输入要提取国际化的页面代码"
          class="content"
          rows="40"
          v-model="code"
        ></el-input>
        <el-button type="primary" @click="analyse">寻找中文字符</el-button>
      </div>
    </template>
  </div>
</template>
<script>
import axios from "axios";
import vueCodeString from "../service/vueCodeString.ts";
import { markString, interpolationMark } from "../service/common";

const NAMESPACESTR = "."; // namespaceStr
const QUOTESMARK = '"'; // template中的quotes引号风格

function getKeyName(...str) {
  str = str
    .filter((str) => str)
    .reduce((arr, name) => {
      return [...arr, ...name.trim().split(/\s+/g)];
    }, []);
  return str
    .slice(0, 3)
    .map((name) => name.toLowerCase())
    .join(NAMESPACESTR);
}

export default {
  name: "HelloWorld",
  data() {
    return {
      pageName: "",
      code: ``,
      findWordArr: [],
      replaceCode: "",
      resultCode: "",
      keyCode: "",
    };
  },
  computed: {
    showReplaceCode() {
      if (!this.replaceCode) {
        return "";
      }
      let code = this.findWordArr.reduce((replaceCode, findWord, index) => {
        if (!findWord.used) {
          return this.replaceNouseCode(
            replaceCode,
            findWord,
            index,
            markString,
            interpolationMark
          );
        } else {
          replaceCode = replaceCode.replace(
            markString[0] + findWord.index + markString[1],
            findWord.replaceCode
          );
        }
        return replaceCode;
      }, this.replaceCode);

      return code;
    },
    showReplaceCodeHTML() {
      let div = document.createElement("div");
      div.textContent = this.showReplaceCode;
      let html = div.innerHTML;
      html = this.findWordArr.reduce((html, word, index) => {
        return html
          .replace(
            `${markString[0]}${index}${markString[1]}`,
            `<span class="heightlight word${index}">$t(${index + 1})</span>`
          )
          .replace(
            `${markString[0]}${index}${interpolationMark[0]}`,
            `<span class="heightlight word${index}">$t(${index + 1}, [</span>`
          )
          .replace(
            `${interpolationMark[1]}${index}${interpolationMark[0]}`,
            `<span class="heightlight word${index}">,</span>`
          )
          .replace(
            `${interpolationMark[1]}${index}${markString[1]}`,
            `<span class="heightlight word${index}">])</span>`
          );
      }, html);

      return html;
    },
    replaceDisable() {
      return this.findWordArr.reduce(
        (bool, findWord) => bool && (!findWord.used || findWord.key),
        true
      );
    },
  },
  watch: {
    code() {
      this.findWordArr = [];
      this.replaceCode = "";
      this.resultCode = "";
    },
  },
  methods: {
    analyse() {
      function hit(code) {
        return /[\u4e00-\u9fa5]/.test(code);
      }

      let result = vueCodeString.extractStringFromVue(this.code, {
        filter: hit,
      });
      window.result = result;
      this.replaceCode = result.result;
      this.findWordArr = result.extractString.map((item) => {
        return {
          ...item,
          used: true,
          key: "",
        };
      });
    },
    replace() {
      let keyArr = [];

      let div = document.createElement("div");
      div.textContent = this.showReplaceCode;
      let html = div.innerHTML;

      this.resultCode = this.findWordArr.reduce((replaceCode, word, index) => {
        if (!word.used) {
          return this.replaceNouseCode(
            replaceCode,
            findWord,
            index,
            markString,
            interpolationMark
          );
        } else {
          // remove pageName prefix
          // let key = getKeyName(this.pageName || "", word.key);
          let key = word.key;

          keyArr.push([key, word.word]);

          let quotationMarks = QUOTESMARK;
          // let quotationMarks = "'";
          let t = "$t";
          if (word.replaceType === "vue-attr" && "'" === word.quotationMarks) {
            quotationMarks = '"';
          }
          if (word.replaceType === "js") {
            t = "this.$t";
          }

          const replaceStr = this.pageName
            ? this.pageName + NAMESPACESTR + word.key
            : word.key;

          return replaceCode
            .replace(
              `${markString[0]}${index}${markString[1]}`,
              `<span class="heightlight">${t}(${quotationMarks}${replaceStr}${quotationMarks})</span>`
            )
            .replace(
              `${markString[0]}${index}${interpolationMark[0]}`,
              `<span class="heightlight">${t}(${quotationMarks}${replaceStr}${quotationMarks}, [</span>`
            )
            .replace(
              `${interpolationMark[1]}${index}${interpolationMark[0]}`,
              `<span class="heightlight">, </span>`
            )
            .replace(
              `${interpolationMark[1]}${index}${markString[1]}`,
              `<span class="heightlight">])</span>`
            );
        }
      }, html);

      const fromEntries = (arr) => {
        const mapData = new Map(arr);
        const obj = {};
        for (let [key, value] of mapData) {
          obj[key] = value;
        }
        return obj;
      };
      if (this.pageName) {
        this.keyCode = { [this.pageName]: fromEntries(keyArr) };
      } else {
        this.keyCode = fromEntries(keyArr);
      }
    },

    replaceNouseCode(
      replaceCode,
      findWord,
      index,
      markString,
      interpolationMark
    ) {
      if (
        replaceCode.indexOf(
          `${markString[0]}${findWord.index}${interpolationMark[0]}`
        ) > -1
      ) {
        let start = replaceCode.indexOf(
            `${markString[0]}${findWord.index}${interpolationMark[0]}`
          ),
          end =
            replaceCode.indexOf(
              `${interpolationMark[1]}${findWord.index}${markString[1]}`
            ) +
            `${markString[1]}${findWord.index}${interpolationMark[1]}`.length;

        return (
          replaceCode.substr(0, start) +
          findWord.originalCode +
          replaceCode.substr(end)
        );
      } else {
        return replaceCode.replace(
          markString[0] + findWord.index + markString[1],
          findWord.originalCode
        );
      }
    },
    goback2() {
      this.keyCode = "";
      this.resultCode = "";
    },
    goback() {
      this.replaceCode = "";
      this.findWordArr = [];
    },
    async getKey() {
      try {
        this.$Spin.show({
          render: (h) => {
            return h("div", [
              h("Icon", {
                class: "demo-spin-icon-load",
                props: {
                  type: "ios-loading",
                  size: 18,
                },
              }),
              h("div", "Loading"),
            ]);
          },
        });
        let data = await axios.get(
          `/transapi?from=cht&to=en&query=${this.findWordArr
            .map((word) => word.word)
            .join("%0A")}`
        );
        this.findWordArr.forEach((_data, index) => {
          if (!_data.key) {
            _data.key = getKeyName(data.data.data[index].dst);
          }
        });
      } catch (e) {
        console.error(e);
        this.$Message.error(e);
      }
      this.$Spin.hide();
    },
    focus(index) {
      let word = document.querySelectorAll(".word" + index);
      if (word.length) {
        word[0].scrollIntoView();
        for (let i = 0; i < word.length; i++) {
          word[i].classList.add("active");
        }
      }
    },
    blur(index) {
      let word = document.querySelectorAll(".word" + index);
      if (word.length) {
        word[0].scrollIntoView();
        for (let i = 0; i < word.length; i++) {
          word[i].classList.remove("active");
        }
      }
    },
  },
};
</script>

<style scoped>
.main {
  padding: 10px;
  width: calc(100vw - 20px);
  height: calc(100vh - 52px);
  box-sizing: border-box;
  margin: 0 auto;
}
.main button {
  padding: 5px;
  cursor: pointer;
}
.inputCode {
  display: flex;
  flex-direction: column;
}
.inputPageName {
  display: flex;
  justify-content: flex-start;
  align-items: center;
  padding: 5px 0;
}

.layout {
  display: flex;
}
.content {
  height: calc(100vh - 42px - 65px);
  padding: 5px;
  margin: 5px 0;
  border: 1px solid #dcdee2;
  border-radius: 5px;
  overflow: auto;
  text-align: left;
}
.result {
  height: calc(100vh - 42px - 100px);
  padding: 5px;
  margin: 5px 0;
  border: 1px solid #dcdee2;
  border-radius: 5px;
  overflow: auto;
  text-align: left;
}
.word-list {
  list-style: none;
  height: calc(100vh - 42px - 100px);
  padding: 5px;
  margin: 5px 0;
  border: 1px solid #dcdee2;
  border-radius: 5px;
  overflow: auto;
}
.word-list li {
  list-style: none;
  justify-content: flex-start;
  display: flex;
  flex-wrap: wrap;
  margin-bottom: 15px;
}
.word-list-item-key {
  width: 100%;
}
.main >>> .heightlight {
  color: red;
  font-weight: bold;
}
.main >>> .heightlight.active {
  text-shadow: 0 0 1px #ffa606;
  animation: hue 2s infinite linear;
  background: #dcdfe6;
}
@keyframes hue {
  10% {
    color: red;
  }
  50% {
    color: #ffa606;
  }
  90% {
    color: red;
  }
}
</style>
