<template>
    <div class="singer" ref="singer">
      <list-view @select="selectSinger" :data="singers"></list-view>
      <router-view></router-view>
    </div>
</template>

<script type="text/ecmascript-6">
import ListView from '../../base/listview/listview'
import {getSinerList} from '../../api/singer'
import {ERR_OK} from '../../api/config'
import Singer from '../../common/js/singer'
import {mapMutations} from 'vuex'



const HOT_SINGER_LEN = 10
const HOT_NAME = '热门'
export default {
  name: 'singer',
  data() {
    return {
      singers: []
    }
  },
  created() {
    this._getSinerList()
  },
  methods: {
    selectSinger(singer) {
      console.log(singer)
      this.$router.push({
        path: `/singer/${singer.id}`
      })
      this.setSinger(singer)
    },
    _getSinerList() {
      getSinerList().then((res) => {
        if (ERR_OK === res.code) {
          this.singers = res.data.list
          this.singers = this._normalizeSiner(res.data.list)
        }
      })
    },
    // 格式化获得的数据
    _normalizeSiner(list) {
      let map = {
        hot: {
          title: HOT_NAME,
          items: []
        }
      }
      list.forEach((item, index) => {
        if (index < HOT_SINGER_LEN) {
          map.hot.items.push(new Singer({
            name: item.Fsinger_name,
            id: item.Fsinger_mid
          }))
        }
        const key = item.Findex
        if (!map[key]) {
          map[key] = {
            title: key,
            items: []
          }
        }
        map[key].items.push(new Singer({
          name: item.Fsinger_name,
          id: item.Fsinger_mid
        }))
      })

      // 为了得到有序列表,需要对map进行处理
      let ret = []
      let hot = []
      for (let key in map) {
        let val = map[key]
        if (val.title.match(/[a-zA-z]/)) {
          ret.push(val)
        } else if (val.title === HOT_NAME) {
          hot.push(val)
        }
      }
      ret.sort((a, b) => {
        return a.title.charCodeAt(0) - b.title.charCodeAt(0)
      })
      return hot.concat(ret)
    },
    ...mapMutations({
      setSinger: 'SET_SINGER'
    })
  },
  components: {
    ListView
  }
}
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
  .singer
    position: fixed
    top: 88px
    bottom: 0
    width: 100%
</style>
