<template>
  <div class="wrap">
    <div class="codeList">
      <el-input v-model="inputValue" placeholder="请输入内容" class="input"></el-input>
      <ul class="snippets-groups">
        <li>
          <ul class="snippets-categories">
            <li v-for="(value,key,index) in filteredCodeSnippetCategories" :key="index">
              <div class="category nowrap" @click="openCodeList(value)">
                <span class="iconfont icon-xia pointer" v-show="value.isExpanded"></span>
                <span class="iconfont icon-you pointer" v-show="!value.isExpanded"></span>
                  <el-tooltip class="item" effect="dark" :content="key" placement="top" :open-delay=500>
                    <span class="list-text">
                      {{key}}
                      </span>
                  </el-tooltip>
                  <span class="badge">{{value.list.length}}</span>
              </div>
              <ul class="snippets-types" v-show="value.isExpanded">
                <li class="snippet nowrap" v-for="(codeSnippet, index) in value.list" :key="index">
                    <el-popover
                      placement="top"
                      width="400"
                      :open-delay=300
                      trigger="hover">
                      <div :class="codeSnippet.name" v-html="`${codeSnippet.name}`" class="color">
                      </div>
                      <span class="list-text" slot="reference" >{{codeSnippet.name}}</span>
                    </el-popover>
                  <span class="iconfont icon-jiantou pointer" @click="addCode(codeSnippet)"></span>
                </li>
              </ul>
            </li>
          </ul>
        </li>
      </ul>
    </div>
    <div class="code">
      <div class="header">
        <el-tooltip :content="'撤销上次修改'" placement="bottom" effect="light">
          <span class="iconfont icon-chexiao1" @click="undo()"></span>
        </el-tooltip>
        <el-tooltip :content="'重做上次修改'" placement="bottom" effect="light">
          <span class="iconfont icon-huifu1" @click="redo()"></span>
        </el-tooltip>
        <span class="seperator"></span>
        <el-tooltip :content="'注释选定文本'" placement="bottom" effect="light">
          <span class="iconfont icon-jia1" @click="comment()"></span>
        </el-tooltip>
        <el-tooltip :content="'取消对选定文本的注释'" placement="bottom" effect="light">
          <span class="iconfont icon-jian1" @click="uncomment()"></span>
        </el-tooltip>
        <span class="seperator"></span>
        <el-tooltip :content="'向左缩进'" placement="bottom" effect="light">
          <span class="iconfont icon-xiangzuosuojin" @click="indentLess()"></span>
        </el-tooltip>
        <el-tooltip :content="'向右缩进'" placement="bottom" effect="light">
          <span class="iconfont icon-xiangyousuojin" @click="indentMore()"></span>
        </el-tooltip>
        <el-tooltip :content="'自动格式化选定文本'" placement="bottom" effect="light">
          <span class="iconfont icon-wusuojin" @click="indentAuto()"></span>
        </el-tooltip>
        <span class="seperator"></span>
        <el-tooltip :content="'全部折叠'" placement="bottom" effect="light">
          <span class="iconfont icon-jian" @click="collapseAll()"></span>
        </el-tooltip>
        <el-tooltip :content="'全部展开'" placement="bottom" effect="light">
          <span class="iconfont icon-jia" @click="expandAll()"></span>
        </el-tooltip>
        <span class="seperator"></span>
      </div>
      <div class="center">
        <textarea class="form-control" id="code" name="code"></textarea>
      </div>
      <!-- 引入后在html界面中建立textarea标签，用于生成代码框 -->
    </div>
  </div>
  
</template>

<script>
import "codemirror/theme/base16-light.css"
import "codemirror/lib/codemirror.css"
import "codemirror/addon/hint/show-hint.css"

import "codemirror/addon/dialog/dialog.css"
let CodeMirror = require("codemirror/lib/codemirror")
require("codemirror/addon/edit/matchbrackets")
require("codemirror/addon/selection/active-line")

import "codemirror/addon/fold/foldgutter.css"
require("codemirror/addon/fold/foldcode")
require("codemirror/addon/fold/brace-fold")
require("codemirror/addon/fold/comment-fold")
require("codemirror/addon/fold/indent-fold")
require("codemirror/addon/fold/foldgutter")

import "codemirror/addon/lint/lint.css"
require("codemirror/addon/lint/lint")
require("codemirror/addon/lint/javascript-lint")

require("codemirror/addon/hint/show-hint")
require("codemirror/mode/javascript/javascript")
require("codemirror/addon/comment/comment")

import 'codemirror/mode/sql/sql'  // 语言语法定义文件



import FunctionCode from '../assets/code/functionCode'



export default {
  name: 'HelloWorld',
  data () {
    return {
      codeList: {},
      filteredCodeSnippetCategories: {},
      inputValue: '',
      toolBar: { isLint: false },
      CodeMirrorEditor: null,
      codeValue: '',
      selection: { from: {}, to: {} },
      hintFunctionCode: [],
      hintColumnCode: [],
    }
  },
  computed: {
    hintCode () {
      return [].concat(this.hintFunctionCode, this.hintColumnCode)
    }
  },
  methods: {
    addCode (val) {
      console.log(val)
      this.CodeMirrorEditor.replaceSelection(val.code) // 追加内容
    },
    openCodeList (item) {
      item.isExpanded = !item.isExpanded
    },
    initCode (data) {
      let obj = {}
      let arr 
      for (let i = 0; i < data.length; i++) {
        if (!obj[data[i].category]) {
          obj[data[i].category] = {}
          obj[data[i].category].list = []
          obj[data[i].category].isExpanded = false
        }
        obj[data[i].category].list.push(data[i])
      }
      return obj
    },
    filterCategories(srcCategories, codeKeyword) {
      console.log(srcCategories)
      let filteredCategories = {}
      let arr = []
      for (let category in srcCategories) {
        console.log(category)
        let codeSnippets = srcCategories[category].list,
          filteredCodeSnippets = codeSnippets.filter((codeSnippet) => {
            return codeSnippet.name.toLowerCase().indexOf(codeKeyword.toLowerCase()) > -1
          })
          filteredCategories[category] = {}
          filteredCategories[category].list = filteredCodeSnippets
          filteredCategories[category].isExpanded = true
      }
      console.log(filteredCategories)
      return filteredCategories
    },
    undo () {
      this.CodeMirrorEditor.undo()
    },
    redo () {
      this.CodeMirrorEditor.redo()
    },
    comment () {
      if (typeof this.selection.from.line === 'undefined' || typeof this.selection.to.line === 'undefined') {
        return
      }
      if (this.selection.from.line <= this.selection.to.line) {
        this.CodeMirrorEditor.lineComment(this.selection.from, this.selection.to)
      } else {
        this.CodeMirrorEditor.lineComment(this.selection.to, this.selection.from)
      }
    },
    uncomment () {
      if (typeof this.selection.from.line === 'undefined' || typeof this.selection.to.line === 'undefined') {
        return
      }
      if (this.selection.from.line <= this.selection.to.line) {
        this.CodeMirrorEditor.uncomment(this.selection.from, this.selection.to)
      } else {
        this.CodeMirrorEditor.uncomment(this.selection.to, this.selection.from)
      }
    },
    indentLess () {
      this.CodeMirrorEditor.indentSelection('subtract')
    },
    indentMore () {
      this.CodeMirrorEditor.indentSelection('add')
    },
    indentAuto () {
      this.CodeMirrorEditor.indentSelection('smart')
    },
    collapseAll () {
      let cm = this.CodeMirrorEditor
      cm.operation(() => {
        for (let i = cm.firstLine(), e = cm.lastLine(); i <= e; i++)
          cm.foldCode(CodeMirror.Pos(i, 0), null, 'fold');
      })
    },
    expandAll () {
      let cm = this.CodeMirrorEditor
      cm.operation(() => {
        for (let i = cm.firstLine(), e = cm.lastLine(); i <= e; i++)
          cm.foldCode(CodeMirror.Pos(i, 0), null, 'unfold');
      })
    },

    addIcon (data, type, name) {
      // 给 hint 添加类型 icon
      let arr = []
      for (let i = 0; i < data.length; i++) {
        let nativeItem = {
          text: data[i].code,
          displayText: data[i].name,
          className: '',
          render: (elt, self, data) => {
            let compEle = document.createElement('div')
            compEle.innerHTML = `<span class="iconfont icon-${type}" style='color: red;'></span>  ${data.displayText}`
            elt.appendChild(compEle)
          }
        }
        this[name].push(nativeItem)
      }
    },

    getHints (cm, option) {
      return new Promise((accept) => {
        setTimeout(() => {
          let WORD = /[\w\[\]\"\.$]+/
          let cursor = cm.getCursor(), curLine = cm.getLine(cursor.line)
          let start = cursor.ch, end = cursor.ch
          while (start && WORD.test(curLine.charAt(start - 1))) {
            --start
          }
          let curWord = start != end && curLine.slice(start, end)
          let list = [], isDefinedObj = false
          if (curWord) {
            if (list.length === 0) {
              var dotIndex = curWord.lastIndexOf('.'),
                  memberStr = curWord.slice(dotIndex + 1),
                  comList = []
              comList = this.hintCode
              for (let j = 0; j < comList.length; j++) {
                  if (comList[j].displayText.toLowerCase().lastIndexOf(memberStr.toLowerCase(), 0) === 0) {
                    list.push(comList[j])
                  }
                }
              isDefinedObj = true
            }
          }
          if (isDefinedObj) {
              return accept({list: list,
                from: CodeMirror.Pos(cursor.line, start),
                to: CodeMirror.Pos(cursor.line, end)})
            } else {
              return accept({list: list,
                from: CodeMirror.Pos(cursor.line, start  + 1),
                to: CodeMirror.Pos(cursor.line, end)})
              }
          }, 100) 
        
      })
    },

    initCursorToEnd() {
        // 初始化光标位置到编辑器结尾
        
      let lastLineNumber = this.CodeMirrorEditor.lastLine(),
        lastCharNumber = this.CodeMirrorEditor.getLine(lastLineNumber).length
      this.CodeMirrorEditor.setCursor(lastLineNumber, lastCharNumber)
      this.selection.from = this.selection.to = this.CodeMirrorEditor.getCursor()
    },
    initCodeMirror () {
      let myTextarea = document.getElementById('code');
      this.CodeMirrorEditor = CodeMirror.fromTextArea(myTextarea, {
        value: '',
        mode:'javascript',//编辑器语言
        // theme:'base16-light', //不设置主题, 使用默认主题
        lineNumbers: true,//显示行号
        smartIndent: true,
        autofocus: true,
        autoRefresh: true,
        historyEventDelay: 500,
        indentWithTabs: true,
        tabSize: 4,
        indentUnit: 4,
        line: true,
        matchBrackets: true,
        autoCloseBrackets: true,
        styleActiveLine: true,
        foldGutter: true,
        lint: this.toolBar.isLint, 
        gutters: ["CodeMirror-linenumbers", "CodeMirror-foldgutter", "CodeMirror-lint-markers"],
        extraKeys: {"Ctrl-Space": "autocomplete"},//自定义快捷键
        // extraKeys: {"Ctrl": "autocomplete"},//ctrl可以弹出选择项 
        hintOptions: {
          completeSingle: false,
          alignWithWord: false,
          hint: this.getHints
        }
      })
      // 当输入框内容发生变化 更新 codeValue 值
      this.CodeMirrorEditor.on('change', (cm) => {
        this.codeValue = cm.getValue()
        console.log(JSON.stringify(this.codeValue))
      })



      this.CodeMirrorEditor.on('cursorActivity', (cm) => {
        this.CodeMirrorEditor.showHint()
        this.selection.to = cm.getCursor()
        if (this.CodeMirrorEditor.getSelection().length === 0) {
          this.selection.from = cm.getCursor()
        }
      })
      this.CodeMirrorEditor.on('mousedown', (cm) => {
        this.selection.from = cm.getCursor()
      })
      
    }
  },
  mounted() {
    this.initCodeMirror()
    this.addIcon(FunctionCode, 'function', 'hintFunctionCode')
    this.codeList = this.initCode(FunctionCode)
    this.filteredCodeSnippetCategories = this.codeList
    console.log(CodeMirror.resolveMode('javascript'))
    
  },
  watch: {
    inputValue (val) {
      this.filteredCodeSnippetCategories = this.filterCategories(this.codeList, val)
    },
  }
}
</script>

<style scoped>
.wrap{display: flex; border: 1px solid #ccc;}
.wrap .codeList{width: 35%;}
.wrap .codeList .input{padding: 10px}
.wrap .codeList .snippets-groups{padding: 5px 10px;}
.wrap .codeList .snippet{display: flex; justify-content: space-between}
.wrap .codeList .snippets-types{ padding:0 20px;}
.wrap .codeList .badge{color: #fff; background-color: #4090F7; padding: 0 8px; border-radius: 4px; margin-left: 8px;}
.wrap .codeList li{margin: 5px 0;}
.wrap .code{width: 65%;}
.code >>> .cm-comment{color: #a50}
.code >>> .cm-keyword{color: #708}
.code{border-left: 1px solid #ccc;}
.header{height: 28px; border-bottom: 1px solid #ccc;}
.code .iconfont {float: left; width: 28px; height: 28px; cursor: pointer; background-position: center; background-repeat: no-repeat; padding-top: 6px; text-align: center;}
.seperator { float: left; width: 0; height: 28px; border-right: 1px solid #ccc; }
</style>
