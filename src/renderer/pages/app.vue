<template>
  <div
    class="editor-container"
  >
    <side-bar></side-bar>
    <div class="editor-middle">
      <title-bar
        :project="projectTree"
        :pathname="pathname"
        :filename="filename"
        :active="windowActive"
        :word-count="wordCount"
        :platform="platform"
        :is-saved="isSaved"
      ></title-bar>
      <recent
        v-if="!hasCurrentFile && init"
      ></recent>
      <editor-with-tabs
        v-if="hasCurrentFile && init"
        :markdown="markdown"
        :filename="filename"
        :cursor="cursor"
        :source-code="sourceCode"
        :show-tab-bar="showTabBar"
        :text-direction="textDirection"
        :platform="platform"
      ></editor-with-tabs>
      <aidou></aidou>
      <about-dialog></about-dialog>
      <rename></rename>
      <tweet></tweet>
      <import-modal></import-modal>
    </div>
  </div>
</template>

<script>
  import { addStyles, addThemeStyle } from '@/util/theme'
  import Recent from '@/components/recent'
  import EditorWithTabs from '@/components/editorWithTabs'
  import TitleBar from '@/components/titleBar'
  import SideBar from '@/components/sideBar'
  import AboutDialog from '@/components/about'
  import Rename from '@/components/rename'
  import Tweet from '@/components/tweet'
  import ImportModal from '@/components/import'
  import { mapState } from 'vuex'
  import bus from '@/bus'
  import { DEFAULT_STYLE } from '@/config'

  export default {
    name: 'marktext',
    components: {
      Recent,
      EditorWithTabs,
      TitleBar,
      SideBar,
      AboutDialog,
      Rename,
      Tweet,
      ImportModal
    },
    data () {
      return {
      }
    },
    computed: {
      ...mapState({
        'showTabBar': state => state.layout.showTabBar,
        'sourceCode': state => state.preferences.sourceCode,
        'theme': state => state.preferences.theme,
        'textDirection': state => state.preferences.textDirection
      }),
      ...mapState({
        'projectTree': state => state.project.projectTree,
        'pathname': state => state.editor.currentFile.pathname,
        'filename': state => state.editor.currentFile.filename,
        'isSaved': state => state.editor.currentFile.isSaved,
        'markdown': state => state.editor.currentFile.markdown,
        'cursor': state => state.editor.currentFile.cursor,
        'wordCount': state => state.editor.currentFile.wordCount
      }),
      ...mapState([
        'windowActive', 'platform', 'init'
      ]),
      hasCurrentFile () {
        return this.markdown !== undefined
      }
    },
    watch: {
      theme: function (value, oldValue) {
        if (value !== oldValue) {
          addThemeStyle(value)
        }
      }
    },
    created () {
      const { commit, dispatch } = this.$store

      // Apply initial state (theme and titleBarStyle) and delay load other values.
      if (global.marktext.initialState) {
        commit('SET_USER_PREFERENCE', global.marktext.initialState)
      }

      // store/index.js
      dispatch('LINTEN_WIN_STATUS')
      // module: tweet
      dispatch('LISTEN_FOR_TWEET')
      // module: layout
      dispatch('LISTEN_FOR_LAYOUT')
      dispatch('LISTEN_FOR_REQUEST_LAYOUT')
      // module: listenForMain
      dispatch('LISTEN_FOR_EDIT')
      dispatch('LISTEN_FOR_VIEW')
      dispatch('LISTEN_FOR_ABOUT_DIALOG')
      dispatch('LISTEN_FOR_PARAGRAPH_INLINE_STYLE')
      // module: project
      dispatch('LISTEN_FOR_UPDATE_PROJECT')
      dispatch('LISTEN_FOR_LOAD_PROJECT')
      dispatch('LISTEN_FOR_SIDEBAR_CONTEXT_MENU')
      // module: autoUpdates
      dispatch('LISTEN_FOR_UPDATE')
      // module: editor
      dispatch('LISTEN_SCREEN_SHOT')
      dispatch('ASK_FOR_USER_PREFERENCE')
      dispatch('ASK_FOR_MODE')
      dispatch('LISTEN_FOR_CLOSE')
      dispatch('LISTEN_FOR_SAVE_AS')
      dispatch('LISTEN_FOR_MOVE_TO')
      dispatch('LISTEN_FOR_SAVE')
      dispatch('LISTEN_FOR_SET_PATHNAME')
      dispatch('LISTEN_FOR_BOOTSTRAP_WINDOW')
      dispatch('LISTEN_FOR_SAVE_CLOSE')
      dispatch('LISTEN_FOR_EXPORT_PRINT')
      dispatch('LISTEN_FOR_RENAME')
      dispatch('LINTEN_FOR_SET_LINE_ENDING')
      dispatch('LISTEN_FOR_NEW_TAB')
      dispatch('LISTEN_FOR_CLOSE_TAB')
      dispatch('LINTEN_FOR_PRINT_SERVICE_CLEARUP')
      dispatch('LINTEN_FOR_EXPORT_SUCCESS')
      dispatch('LISTEN_FOR_FILE_CHANGE')
      // module: notification
      dispatch('LISTEN_FOR_NOTIFICATION')

      // prevent Chromium's default behavior and try to open the first file
      window.addEventListener('dragover', e => {
        // Cancel to allow tab drag&drop.
        if (!e.dataTransfer.types.length) return

        if (e.dataTransfer.types.indexOf('Files') >= 0) {
          if (e.dataTransfer.items.length === 1 && e.dataTransfer.items[0].type.indexOf('image') > -1) {
            // Do nothing, because we already drag/drop image in muya.
          } else {
            e.preventDefault()
            if (this.timer) {
              clearTimeout(this.timer)
            }
            this.timer = setTimeout(() => {
              bus.$emit('importDialog', false)
            }, 300)
            bus.$emit('importDialog', true)
          }

          e.dataTransfer.dropEffect = 'copy'
        } else {
          e.stopPropagation()
          e.dataTransfer.dropEffect = 'none'
        }
      }, false)

      this.$nextTick(() => {
        const style = global.marktext.initialState || DEFAULT_STYLE
        addStyles(style)
      })
    }
  }
</script>

<style scoped>
  .editor-container {
    display: flex;
    flex-direction: row;
    position: absolute;
    width: 100vw;
    height: 100vh;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
  }
  .editor-container .hide {
    z-index: -1;
    opacity: 0;
    position: absolute;
    left: -10000px;
  }
  .editor-middle {
    display: flex;
    flex-direction: column;
    flex: 1;
    min-height: 100vh;
    position: relative;
    & > .editor {
      flex: 1;
    }
  }
</style>
