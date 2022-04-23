<template>
  <div class="channel-edit">
    <van-cell :border="false">
      <div slot="title" class="title-text">我的频道</div>
      <van-button class="edit-btn" type="danger" plain round size="mini" @click="isEdit = !isEdit">{{ isEdit ? '完成' : '编辑' }}</van-button>
    </van-cell>
    <van-grid class="my-grid" :gutter="10">
      <van-grid-item class="grid-item" v-for="(channel, index) in myChannels" :key="index" @click="onMyChannelClick(channel, index)">
        <van-icon slot="icon" name="clear" v-show="isEdit && !flexChannels.includes(channel.id)"></van-icon>
        <span class="text" :class="{ active: index === active }" slot="text">{{ channel.name }}</span>
      </van-grid-item>
    </van-grid>
    <!-- 频道推荐 -->
    <van-cell :border="false">
      <div slot="title" class="title-text">频道推荐</div>
    </van-cell>
    <van-grid class="recommend-grid" :gutter="10">
      <van-grid-item @click="onAddChannel(channel)" icon="plus" class="grid-item" v-for="(channel, index) in recommendChannels" :key="index" :text="channel.name" />
    </van-grid>
  </div>
</template>

<script>
import { getAllChannels, addUserChannel, deleteUserChannel } from '../../../api/channel'

import { mapState } from 'vuex'

import { setItem } from '../../../utils/storage'

export default {
  name: 'HeimaVueChannelEdit',
  props: {
    myChannels: {
      type: Array,
      required: true
    },
    active: {
      type: Number,
      required: true
    }
  },
  computed: {
    // recommendChannels() {
    //   const channels = []
    //   this.allChannels.forEach(channel => {
    //     const res = this.myChannelS.find(myChannel => {
    //       return myChannel.id === channel.id
    //     })
    //     // 如果我的频道中不包括我的频道项，则收集到推荐频道中
    //     if (!res) {
    //       channels.push(channel)
    //     }
    //   })
    //   return channels
    // }

    ...mapState(['user']),

    recommendChannels() {
      // 这是数组的 filter 方法，把符合条件的元素存储到新数组中并返回
      return this.allChannels.filter(channel => {
        // const channels = []

        // 数组的 find 方法，遍历数组， 把符合条件的第一个元素返回
        return !this.myChannelS.find(myChannel => {
          return myChannel.id === channel.id
        })
        // return channels
      })
    }
  },

  data() {
    return {
      allChannels: [], //所有频道
      myChannelS: this.myChannels,
      isEdit: false, // 控制编辑状态的显示
      flexChannels: [0] //固定频道，不允许删除
    }
  },

  mounted() {},
  created() {
    this.loadSAllChannels()
  },

  methods: {
    async loadSAllChannels() {
      try {
        const { data } = await getAllChannels()
        // console.log(data)
        this.allChannels = data.data.channels
      } catch (err) {
        this.$toast('数据获取失败')
      }
    },
    async onAddChannel(channel) {
      this.myChannelS.push(channel)

      // 数据持久化处理
      if (this.user) {
        // 已登录，把数据请求接口放到线上
        try {
          await addUserChannel({
            id: channel.id, //频道id
            seq: this.myChannels.length // 序号
          })
        } catch (err) {
          console.log('出错了', err)
        }
      } else {
        // 未登录，把数据存储到本地
        setItem('TOUTIAO_CHANNELS', this.myChannels)
      }
    },
    onMyChannelClick(channel, index) {
      if (this.isEdit) {
        // 1.如果是固定频道， 则不删除
        if (this.flexChannels.includes(channel.id)) {
          return
        }
        // 2. 删除频道项
        this.myChannelS.splice(index, 1)

        // 3.如果删除的是激活频道之前的频道，则更新激活的频道项
        // 参数一： 要删除的元素的索引（包括该索引）
        // 参数二： 删除的个数，如果不指定，则从参数一开始一直删到最后
        if (index <= this.active) {
          // 让激活频道的索引 -1
          this.$emit('update-active', this.active - 1)
        }

        // 4. 处理持久化

        this.deleteChannel()
      } else {
        // 非编辑状态，执行切换频道
        this.$emit('update-active', index, false)
      }
    },
    async deleteChannel(channel) {
      try {
        if (this.user) {
          // 已登录，则将数据更新到线上
          await deleteUserChannel(channel.id)
        } else {
          // 未登录，将数据更新到本地
          setItem('TOUTIAO_CHANNELS', this.myChannels)
        }
      } catch (err) {
        this.$toast('操作失败，请稍后重试')
      }
    }
  }
}
</script>

<style lang="less" scoped>
.channel-edit {
  padding: 85px 0;
  .title-text {
    font-size: 32px;
    color: #333;
  }
  .edit-btn {
    width: 104px;
    height: 48px;
    font-size: 26px;
    color: #f85959;
    border: 1px solid #f85959;
  }
  .grid-item {
    width: 160px;
    height: 86px;
    /deep/ .van-grid-item__content {
      word-spacing: nowrap;
      background-color: #f4f5f6;
      .van-grid-item__text,
      .text {
        font-size: 28px;
        color: #222;
        margin-top: 0;
      }
    }
  }

  /deep/ .my-grid {
    .grid-item {
      .van-icon-clear {
        position: absolute;
        right: -10px;
        top: -10px;
        font-size: 30px;
        color: #cacaca;
        z-index: 2;
      }
      .active {
        color: red;
      }
      .van-grid-item__icon-wrapper {
        position: unset;
      }
    }
  }

  /deep/ .recommend-grid {
    .grid-item {
      .van-grid-item__content {
        flex-direction: row;
        .van-icon-plus {
          font-size: 28px;
          margin-right: 6px;
        }
      }
    }
  }
}
</style>
