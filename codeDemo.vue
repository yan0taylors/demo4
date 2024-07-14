<template>
  <div>
    <label for="initialVersion">请输入初始版本号：</label>
    <input id="initialVersion" type="text" v-model="initialVersion">

    <label for="upgradedVersion">请输入升级后的版本号：</label>
    <input id="upgradedVersion" type="text" v-model="upgradedVersion">
    <button @click="getDataFromBackend" class="custom-button">点击生成lua脚本</button>
    <codemirror
      class="lua-editor"
      ref="editor"
      :value="value"
      :options="codemirrorOptions"
      v-model="code"
      @input="handleInputChange"
      @input-read="handleInputRead"
    >
        
    </codemirror>
  </div>
    
  </template>
  
  <script>
  import { codemirror } from 'vue-codemirror'
  import axios from 'axios';
  import 'codemirror/lib/codemirror.css'
  import 'codemirror/mode/lua/lua'
  
  import 'codemirror/theme/neat.css'
  import 'codemirror/theme/idea.css'
  
  // #region 搜索功能
  // find：Ctrl-F (PC), Cmd-F (Mac)
  // findNext：Ctrl-G (PC), Cmd-G (Mac)
  // findPrev：Shift-Ctrl-G (PC), Shift-Cmd-G (Mac)
  // replace：Shift-Ctrl-F (PC), Cmd-Alt-F (Mac)
  // replaceAll：Shift-Ctrl-R (PC), Shift-Cmd-Alt-F (Mac)
  import 'codemirror/addon/dialog/dialog.css'
  import 'codemirror/addon/dialog/dialog'
  import 'codemirror/addon/search/searchcursor'
  import 'codemirror/addon/search/search'
  import 'codemirror/addon/search/jump-to-line'
  import 'codemirror/addon/search/matchesonscrollbar'
  import 'codemirror/addon/search/match-highlighter'
  // #endregion
  
  // #region 代码提示功能
  // 具体语言可以从 codemirror/addon/hint/ 下引入多个
  import 'codemirror/addon/hint/show-hint.css'
  import 'codemirror/addon/hint/show-hint.js'
  import 'codemirror/addon/hint/anyword-hint'  // 简易的代码提示功能
  // #endregion
  
  // #region 高亮行功能
  import 'codemirror/addon/selection/active-line'
  import 'codemirror/addon/selection/selection-pointer'
  // #endregion
  
  // #region 覆盖scrollbar样式功能
  import 'codemirror/addon/scroll/simplescrollbars.css'
  import 'codemirror/addon/scroll/simplescrollbars'
  // #endregion
  
  //  #region 自动括号匹配功能
  import 'codemirror/addon/edit/matchbrackets.js'
  // #endregion
  
  // 全屏功能 由于项目复杂，自带的全屏功能一般不好使
  // import 'codemirror/addon/display/fullscreen.css'
  // import 'codemirror/addon/display/fullscreen.js'
  
  // 显示自动刷新
  import 'codemirror/addon/display/autorefresh.js'
  
  // 多语言支持？
  // import 'codemirror/addon/mode/overlay'
  // import 'codemirror/addon/mode/multiplex'
  
  import 'codemirror/addon/lint/lint.js'
  import 'codemirror/addon/lint/lint.css'
  
  // #region  代码段折叠功能
  import 'codemirror/addon/fold/foldcode.js'
  import 'codemirror/addon/fold/foldgutter.js'
  import 'codemirror/addon/fold/foldgutter.css'
  import 'codemirror/addon/fold/brace-fold.js' // 括号折叠
  import 'codemirror/addon/fold/comment-fold.js'
  import 'codemirror/addon/fold/indent-fold.js' // 缩进折叠
  import 'codemirror/addon/fold/comment-fold.js'
  // import 'codemirror/addon/fold/xml-fold.js'
  // import 'codemirror/addon/fold/markdown-fold.js'
  // #endregion
  
  // #region  merge功能
  // import 'codemirror/addon/merge/merge.css'
  // import 'codemirror/addon/merge/merge.js'
  // #endregion
  
  export default {
    name: 'LuaEditor',
  
    // #region 组件基础
    components: { codemirror: codemirror },
    props: {
      value: {
        type: String,
        required: true
      }
    },
    // #endregion
  
    // #region 数据相关
    data() {
      return {
        code: 'this is a lua code',  // 存储从服务端获取的代码
        /** @type {import('@types/codemirror/index').EditorConfiguration}*/
        codemirrorOptions: {
          // 语言及语法模式
          mode: 'text/x-lua',
          // 主题
          theme: 'neat',
          // 显示函数
          line: true,
          // 显示行号
          lineNumbers: true,
          // 软换行
          lineWrapping: true,
          // tab宽度
          tabSize: 4,
          // 允许拖入的文件类型
          allowDropFileTypes: ['text/x-lua'],
          cursorScrollMargin: 5,
          extraKeys: {},
          // 高亮行功能
          styleActiveLine: true,
          // 调整scrollbar样式功能
          // scrollbarStyle: 'overlay',
          // 自动括号匹配功能
          matchBrackets: true,
          autofocus: true,
          autoRefresh: true,
          // #region 代码折叠
          foldGutter: true,
          // foldOptions: { scanUp: true },
          gutters: [
            'CodeMirror-linenumbers',
            'CodeMirror-foldgutter',
            'CodeMirror-lint-markers'
          ],
          // #endregion
          showHint: true,
          lint: true,
          hintOptions: {
            // 避免由于提示列表只有一个提示信息时，自动填充
            completeSingle: false
          }
        }
      }
    },
    computed: {},
    watch: {},
    // #endregion
  
    // #region 生命周期
    created() {},
    mounted() {},
    // #endregion


    

    methods: {
      async getDataFromBackend() {
      this.code = "按钮已经点击";
      try {
        const response = await axios.get('/code/getCode');  // 替换为实际的获取代码的API接口
        this.code = response.data.code;  // 假设服务端返回的数据结构中有一个code字段
      } catch (error) {
        console.error('Error fetching code from server:', error);
        
      }
    },
      handleInputChange(value) {
        this.$emit('input', value)
      },
      handleInputRead(cm) {
        // 显示代码提示框
        cm.showHint()
      }
    }
  }
  </script>
  
  <style lang="scss">
  .lua-editor {
    textarea {
      height: 100%;
    }
  }
  </style>
  