<template>
  <transition name="list-fade">
    <div
      class="playlist"
      @click="hide"
      v-show="showFlag"
    >
      <div class="list-wrapper" @click.stop>
        <div class="list-header">
          <h1 class="title">
            <i class="icon" :class="iconMode" @click="changeMode"></i>
            <span class="text">{{modeText}}</span>
            <span class="clear" @click="showConfirm"><i class="icon-clear"></i></span>
          </h1>
        </div>
        <scroll
          class="list-content"
          ref="listContent"
          :data="sequenceList"
          :refreshDelay="refreshDelay"
        >
          <transition-group ref="list" name="list" tag="ul">
            <li
              class="item"
              v-for="(item,index) in sequenceList"
              :key="item.id"
              @click="selectItem(item,index)"
            >
              <i class="current" :class="getCurrentIcon(item)"></i>
              <span class="text">{{item.name}}</span>
              <span @click.stop="toggleFavorite(item)" class="like">
                <i :class="getFavoriteIcon(item)"></i>
              </span>
              <span @click.stop="deleteOne(item)" class="delete">
                <i class="icon-delete"></i>
              </span>
            </li>
          </transition-group>
        </scroll>
        <div class="list-operate">
          <div @click="addSong" class="add">
            <i class="icon-add"></i>
            <span class="text">添加歌曲到队列</span>
          </div>
        </div>
        <div @click="hide" class="list-close">
          <span>关闭</span>
        </div>
      </div>
      <confirm
        ref="confirm"
        @confirm="confirmClear"
        text="是否清空播放列表"
        confirmBtnText="清空"
      >
      </confirm>
      <!--<add-song ref="addSong"></add-song>-->
    </div>
  </transition>
</template>

<script type="text/ecmascript-6">
  import {mapActions} from "vuex"
  import {playMode} from "common/js/config"
  import {playerMixin} from "common/js/mixin"
  import Scroll from "base/scroll/scroll"
  import Confirm from "base/confirm/confirm"
  //  import AddSong from "components/add-song/add-song"

  export default {
    mixins: [playerMixin],
    data () {
      return {
        showFlag: false,
        refreshDelay: 120
      }
    },
    computed: {
      modeText () {
        return this.mode === playMode.sequence
          ? "顺序播放"
          : this.mode === playMode.random
            ? "随机播放" : "单曲循环"
      }
    },
    methods: {
      show () {
        this.showFlag = true
        // 展示后刷新，防止dom没有被正确计算,导致scroll失效
        setTimeout(() => {
          this.$refs.listContent.refresh()
          this.scrollToCurrent(this.currentSong)
        }, 20)
      },
      hide () {
        this.showFlag = false
      },
      showConfirm () {
        this.$refs.confirm.show()
      },
      confirmClear () {
        this.deleteSongList()
        this.hide()
      },
      getCurrentIcon (item) {
        if (this.currentSong.id === item.id) {
          return "icon-play"
        }
        return ""
      },
      selectItem (item, index) {
        if (this.mode === playMode.random) {
          index = this.playlist.findIndex(song => {
            return song.id === item.id
          })
        }
        this.setCurrentIndex(index)
        this.setPlayingState(true)
      },
      scrollToCurrent (current) {
        const index = this.sequenceList.findIndex(song => {
          return current.id === song.id
        })
        this.$refs.listContent.scrollToElement(this.$refs.list.$el.children[index], 300)
      },
      deleteOne (item) {
        this.deleteSong(item)
        if (!this.playlist.length) {
          this.hide()
        }
      },
      addSong () {
        this.$refs.addSong.show()
      },
      ...mapActions([
        "deleteSong",
        "deleteSongList"
      ])
    },
    watch: {
      currentSong (newSong, oldSong) {
        if (!this.showFlag || newSong.id === oldSong.id) {
          return
        }
        setTimeout(() => {
          this.scrollToCurrent(newSong)
        }, 20)
      }
    },
    components: {
      Scroll,
//      AddSong
      Confirm
    }
  }
</script>

<style scoped lang="scss" rel="stylesheet/scss">
  @import "../../common/scss/variable.scss";
  @import "../../common/scss/_mixin.scss";
  @import "./playlist.scss";
</style>