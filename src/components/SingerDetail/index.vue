<template>
  <transition name="slide">
    <music-list v-if="singer.id" class="singer-detail" :title="title" :bgImage="bgImage" :songs="songs" />
  </transition>
</template>
<script>
  import { mapGetters } from 'vuex'
  import { getSingerDetail } from 'api/singer'
  import * as config from 'api/config'
  import MusicList from 'components/MusicList'
  import { getvkey } from "../../api/song"
  import { createSong } from 'common/js/song'
  export default {
    name: 'scrollDetail',
    data () {
      return {
        songs: []
      }
    },
    computed: {
      ...mapGetters(['singer']),
      title () {
        return this.singer.name
      },
      bgImage () {
        return this.singer.avatar
      }
    },
    created () {
      this._getDetail()
    },
    methods: {
      _getDetail () {
        if (!this.singer.id) {
          this.$router.push('/singer')
          return
        }
        let data = Object.assign({}, config.singerDetailParam, {
          singermid: this.singer.id
        })
        getSingerDetail(
          config.singerDetailUrl,
          data,
          config.singetDetailOpts
        ).then(res => {
          if (res.code === config.ERR_OK) {
            this._normalizeSongs(res.data.list)
          } else {
            console.log(`歌手详情失败, 返回歌手列表哦`)
          }
        })
      },
      _normalizeSongs (list) {
        // console.log(list)
        let songmidlist = []
        let songtypelist = []
        let ret = []
        list.forEach(item => {
          let { musicData } = item
          songmidlist.push(musicData.songmid)
          songtypelist.push(0)
        })
        getvkey(songmidlist,songtypelist).then((res)=>{
          // console.log(res.url_mid.data.midurlinfo[0].vkey)
          const vkeylist = res.url_mid.data.midurlinfo
          let songlist = [...list]
          songlist.map((item,index)=>{
            item.purl = vkeylist[index].purl
            return item
          })
          songlist.forEach(item =>{
            const { musicData,purl } = item
            if (musicData.songid && musicData.albummid) {
              ret.push(createSong(musicData,purl))
            }
          })
          // console.log(ret)
          this.songs = ret
        })
      }
    },
    components: {
      MusicList
    }
  }
</script>
<style lang="less" scoped>
  .singer-detail {
    // position: absolute;
    // left: 0;
    // right: 0;
    // bottom: 0;
    // top: 0;
    // z-index: 3;
    background: #f4f4f4;
    overflow: hidden;
    &.slide-enter-active,
    &.slide-leave-active {
      transition: all 0.5s;
    }
    &.slide-enter,
    &.slide-leave-to {
      opacity: 0;
      transform: translate3d(100%, 0, 0);
    }
  }
</style>
