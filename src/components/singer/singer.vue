<template>
  <!-- static/02-歌手界面.png -->
  <div class="singer" ref="singer">
    <list-view 
      ref="list"
      :data="singers"
      @select="selectSinger"
    >
    </list-view>
    <!-- singer-detail.vue render here -->
    <router-view></router-view>
  </div>
</template>

<script type="text/ecmascript-6">
/* vuex grammer suger */
import { mapMutations } from 'vuex'
import { getSingerList } from 'api/singer'
import { ERR_OK } from 'api/config'
import { playlistMixin } from 'common/js/mixin'
import ListView from 'base/listview/listview'
import Singer from 'common/js/singer'

const HOT_SINGER_LEN = 10
const HOT_NAME = '热门'

export default {
	components: {
		ListView
	},
	/* insert minxin.js */
	mixins: [playlistMixin],
	/* store back-end data */
	data() {
		return {
			singers: []
		}
	},
	/* get back-end data, created not monitor, data, props will monitor */
	created() {
		/* get data(singer data) */
		this._getSingerList()
	},
	methods: {
		/* if mini player, the singerlist bottom add height */
		handlePlaylist(playlist) {
			const bottom = playlist.length > 0 ? '60px' : ''
			this.$refs.singer.style.bottom = bottom
			this.$refs.list.refresh()
		},
		selectSinger(singer) {
      /* 路由跳转 */
			this.$router.push({
				path: `/singer/${singer.id}`
			})
      /* this.setSinger: ...mapMutations */
      /* 保存数据到 vuex/state */
			this.setSinger(singer)
		},
		_getSingerList() {
			getSingerList().then(res => {
				if (res.code === ERR_OK) {
          /* https://c.y.qq.com/v8/fcg-bin/v8.fcg?g_tk=5381&inCharset=utf-8&outCharset=utf-8&notice=0&format=jsonp&channel=singer&page=list&key=all_all_all&pagesize=100&pagenum=1&hostUin=0&needNewCode=0&platform=yqq&jsonpCallback=__jp0 */ 
					this.singers = this._normalizeSinger(res.data.list)
          // console.log(this._normalizeSinger(res.data.list))
        }
			})
    },
    /* handle data */ 
		_normalizeSinger(list) {
			/* data structure */
			let map = {
				hot: {
					title: HOT_NAME,
					items: []
				}
			}
			list.forEach((item, index) => {
				/* previous 10 data, include '热门' */
				if (index < HOT_SINGER_LEN) {
					map.hot.items.push(
						new Singer({
							name: item.Fsinger_name,
							id: item.Fsinger_mid
						})
					)
				}
        /* after 10 data */
        /* https://c.y.qq.com/v8/fcg-bin/v8.fcg?g_tk=5381&inCharset=utf-8&outCharset=utf-8&notice=0&format=jsonp&channel=singer&page=list&key=all_all_all&pagesize=100&pagenum=1&hostUin=0&needNewCode=0&platform=yqq&jsonpCallback=__jp0 */ 
        /* item.Findex: 'A', 'B'... */ 
				const key = item.Findex
				/* data structure */
				if (!map[key]) {
					map[key] = {
						title: key,
						items: []
					}
				}
				map[key].items.push(
					new Singer({
						name: item.Fsinger_name,
						id: item.Fsinger_mid
					})
				)
			})
			/* handle map get order list */
			let ret = []
			let hot = []
			for (let key in map) {
				let val = map[key]
				if (val.title.match(/[a-zA-Z]/)) {
					ret.push(val)
				} else if (val.title === HOT_NAME) {
					hot.push(val)
				}
      }
      /* ascend sort */
			ret.sort((a, b) => {
				return a.title.charCodeAt(0) - b.title.charCodeAt(0)
			})
			return hot.concat(ret)
		},
		/* call vuex/mutations */
		...mapMutations({
      /* SET_SINGER: vuex/mutation-type */ 
			setSinger: 'SET_SINGER'
		})
	}
}
</script>

<style scoped lang="scss" rel="stylesheet/scss">
@import './singer.scss';
</style>
