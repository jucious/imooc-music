<template>
  <!-- static/07-排行榜详情界面.png -->
  <transition name="slide">
    <music-list
      :rank="rank"
      :title="title"
      :bg-image="bgImage"
      :songs="songs"
    >
    </music-list>
  </transition>
</template>

<script type="text/ecmascript-6">
import MusicList from 'components/music-list/music-list'
import { getMusicList } from 'api/rank'
import { ERR_OK } from 'api/config'
import { mapGetters } from 'vuex'
import { createSong } from 'common/js/song'

export default {
	// 数据, 不需要被监控, data, props里面的数据会被监控
	// get back-end data
	created() {
		this._getMusicList()
	},
	data() {
		return {
			songs: [],
			rank: true
		}
	},
	computed: {
		// vuex (store/state.js)
		...mapGetters(['topList']),
		title() {
			// ...mapGetters
			return this.topList.topTitl
		},
		bgImage() {
			if (this.songs.length) {
				return this.songs[0].image
			}
			return ''
		}
	},
	methods: {
		_getMusicList() {
			if (!this.topList.id) {
				this.$router.push('/rank')
				return
			}
			getMusicList(this.topList.id).then(res => {
				if (res.code === ERR_OK) {
					this.songs = this._normalizeSongs(res.songlist)
				}
			})
		},
		_normalizeSongs(list) {
			let ret = []
			list.forEach(item => {
				const musicData = item.data
				if (musicData.songid && musicData.albummid) {
					ret.push(createSong(musicData))
				}
			})
			return ret
		}
	},
	components: {
		MusicList
	}
}
</script>

<style scoped lang="scss" rel="stylesheet/scss">
@import './top-list.scss';
</style>
