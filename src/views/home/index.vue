<template>
  <div class="home-container">
    <!-- 导航栏 -->
    <van-nav-bar class="page-nav-bar" fixed>
      <van-button class="search-btn" slot="title" type="info" size="small" round icon="search" to="/search">搜索</van-button>
    </van-nav-bar>

    <van-tabs class="channel-tabs" v-model="active" animated swipeable>
      <van-tab v-for="channel in channels" :key="channel.id" :title="channel.name">
        <article-list :channel="channel"></article-list>
      </van-tab>
      <div slot="nav-right" class="placeholder"></div>
      <div slot="nav-right" class="hamburger-btn" @click="isChennelEditShow = true">
        <i class="toutiao toutiao-gengduo"></i>
      </div>
    </van-tabs>

    <!-- 频道列表 -->

    <!-- 频道编辑弹出层 -->
    <van-popup v-model="isChennelEditShow" closeable position="bottom" close-icon-position="top-left" :style="{ height: '100%' }"><channel-edit :my-channels="channels" :active="active" @update-active="onUpdateActive"></channel-edit></van-popup>
  </div>
</template>

<script>
import { getUserChannels } from '../../api/user'
import ArticleList from '../../views/home/components/article-list.vue'
import ChannelEdit from './components/channel-edit.vue'
import { mapState } from 'vuex'
import { getItem } from '../../utils/storage'

export default {
  name: 'HeimaVueHome',

  data() {
    return {
      active: 0,
      channels: [], //频道列表
      isChennelEditShow: false //控制编辑频道弹出层的显示状态
    }
  },

  mounted() {},
  computed: {
    ...mapState(['user'])
  },

  methods: {
    async loadChannels() {
      try {
        // const { data } = await getUserChannels()
        // this.channels = data.data.channels

        let channels = []
        if (this.user) {
          // 已登录，请求获取用户频道列表

          const { data } = await getUserChannels()
          this.channels = data.data.channels
        } else {
          // 未登录，判断是否有本地的频道列表数据
          const loadChannels = getItem('TOUTIAO_CHANNELS')
          // 有，拿来使用
          if (loadChannels) {
            channels = loadChannels
          } else {
            // 没有， 请求获取默认频道列表
            const { data } = await getUserChannels()
            this.channels = data.data.channels
          }
        }
        this.channels = channels
      } catch (err) {
        this.$toast('获取频道列表数据失败')
      }
    },
    onUpdateActive(index, isChennelEditShow = true) {
      this.active = index
      this.isChennelEditShow = isChennelEditShow
    }
  },

  created() {
    this.loadChannels()
  },
  components: {
    ArticleList,
    ChannelEdit
  }
}
</script>

<style lang="less" scoped>
.home-container {
  padding-top: 174px;
  padding-bottom: 100px;
  width: 100%;
  /deep/ .van-nav-bar__title {
    max-width: 100% !important;
  }
  .search-btn {
    width: 555px;
    height: 64px;
    background-color: #5babfb;
    border: none;
    font-size: 28px;
    .van-icon {
      font-size: 32px;
    }
  }
  /deep/ .channel-tabs {
    .van-tabs__wrap {
      height: 82px;
      position: fixed;
      top: 92px;
      z-index: 1;
      left: 0;
      right: 0;
    }
    .van-tab {
      border-right: 1px solid #edeff3;
      min-width: 184px;
      font-size: 30px;
      color: #777;
    }
    .van-tab--active {
      color: #333;
    }
    .van-tabs__nav {
      padding-bottom: 0;
    }
    .van-tabs__line {
      width: 31px !important;
      height: 6px;
      background-color: #3296fa;
      bottom: 8px;
    }
    .placeholder {
      // flex-shrink: 0;
      width: 66px;
      height: 82px;
    }
    .hamburger-btn {
      position: fixed;
      right: 0;
      width: 66px;
      height: 82px;
      display: flex;
      justify-content: center;
      align-items: center;
      background-color: #fff;
      opacity: 0.9;
      i.toutiao {
        font-size: 33px;
      }
      &::before {
        content: '';
        position: absolute;
        left: 0;
        width: 2px;
        height: 100%;
        background-image: url('../../assets/gradient-gray-line.png');
        background-size: contain;
      }
    }
  }
}
</style>
