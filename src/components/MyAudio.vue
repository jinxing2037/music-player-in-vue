<template>
  <section>
    <audio
      :src="songInfo.musicUrl"
      ref="audio"
      @ended="handleEnded"
      @durationchange="refreshDuration"
      autoplay
    ></audio>
    <section v-show="showFlag" class="footer-control-panel clear" @click="toSongComponent">
      <div class="img-wrap">
        <!-- FIXME: 使用v-lazy指令时，图片不会更新 -->
        <img class="small-cover-img" v-lazy="songInfo.coverImgUrl" alt>
      </div>
      <div class="song-info">
        <p class="song-name ellipsis">{{songInfo.name}}</p>
        <p class="song-singers ellipsis">{{songInfo.singers}}</p>
      </div>
      <div class="btn-wrap">
        <button class="btn">
          <i class="iconfont icon-gengduoxiao"></i>
        </button>
      </div>
      <div class="btn-wrap" @click.stop="togglePlayerStatus">
        <button class="btn">
          <i :class="iconClassname"></i>
        </button>
      </div>
    </section>
  </section>
</template>

<script>
import { mapGetters, mapActions } from 'vuex';

export default {
  name: 'MyAudio',
  data() {
    return {
      iconClassname: 'iconfont icon-bofang1',
    };
  },
  methods: {
    ...mapActions([
      'getThenSetDuration',
      'getThenSetCurrentTime',
      'getThenSetIsPlaying',
    ]),
    // TODO: 把定时器改为computed或者watch
    setTimer() {
      this.timer = setInterval(() => {
        this.getThenSetCurrentTime(this.$refs.audio.currentTime);
      }, 500);
    },
    clearTimer() {
      clearInterval(this.timer);
      this.timer = null;
    },
    toggleToStopStatus() {
      this.$refs.audio.pause();
      this.iconClassname = 'iconfont icon-bofang1';
      this.clearTimer();
      this.getThenSetIsPlaying(false);
    },
    toggleToPlayStatus() {
      this.$refs.audio.play();
      this.iconClassname = 'iconfont icon-zanting';
      this.setTimer();
    },
    refreshDuration() {
      this.getThenSetDuration(this.$refs.audio.duration);
    },
    handleEnded() {
      this.clearTimer();
      this.getThenSetIsPlaying(false);
      this.getThenSetCurrentTime(0);
    },
    togglePlayerStatus() {
      this.getThenSetIsPlaying(!this.isPlaying);
    },
    toSongComponent() {
      // 下面这行代码有很大的改进空间，这些参数在另一种方案下应该就不需要这样传来传去了
      this.$router.push({
        name: 'Song',
        query: {
          name: this.songInfo.name,
          singers: this.songInfo.singers,
          id: this.songInfo.id,
        },
        params: { coverImgUrl: this.songInfo.coverImgUrl },
      });
    },
  },
  computed: {
    // TODO: 为什么在computed里面？
    ...mapGetters(['songInfo', 'isPlaying']),
    showFlag() {
      // emmmmmmmm 这个判定条件写的真好
      // 不在‘song'路由，且audio里有歌时(标志位为duration不为NaN）才显示（后者是为了避嫌没歌的时候没有封面的尴尬情况）
      if (
        this.$route.name !== 'Song'
        && this.$refs.audio
        && this.$refs.audio.duration
      ) {
        // debugger;
        return true;
      }
      // debugger;
      return false;
    },
  },
  watch: {
    isPlaying(newValue) {
      newValue ? this.toggleToPlayStatus() : this.toggleToStopStatus();
    },
    /*     [this.$ref.audio.current]() {

    }, */
  },
};
</script>

<style scoped lang='less'>
img[lazy=loading]{
  width: .5rem;
  height: .5rem;
}

.footer-control-panel {
  border-top: 1px solid #e4e4e4;
  position: fixed;
  z-index: 999;
  bottom: 0;
  width: 100%;
  height: 0.5rem;
  padding: 0.03rem;
  background: #fff;
  > * {
    margin-right: 0.05rem;
  }
  .img-wrap,
  .song-info {
    float: left;
  }
  .btn-wrap {
    float: right;
    height: 0.5rem;
    .btn {
      .iconfont {
        line-height: 0.5rem;
        color: #2e3030;
        font-size: 0.36rem;
      }
    }
  }
  .img-wrap {
    .small-cover-img {
      width: 0.5rem;
      height: 0.5rem;
      border-radius: 0.02rem;
    }
  }
  .song-info {
    color: #2e3030;
    width: 60%;
    .song-name {
      font-size: 0.17rem;
      line-height: 0.3rem;
    }
    .song-singers {
      line-height: 0.17rem;
      color: lightgray;
    }
  }
}
</style>

// 设置URL (==》 更新duration)
//         ==> 设置isPlaying为true ==> 开始播放，更新进度
