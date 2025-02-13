<template>
    <div class="center-flex center-flex--overlow-hide">
        <div class="player">
          <transition name="fade-in" appear>
            <div v-if="video[0]" class="player__top">
              <span @click="copyMirko" class="hide-mobile"><i class="fas fa-play"></i>{{ video[0].duration }}</span>
              <i class="fas fa-eye"></i>{{ video[0].view_count }}
              <i class="fas fa-calendar"></i>{{ date }}
              <input ref="mirkoInput" :value="mirkoLink" type="hidden" />
              <span class="topic">{{ streamTopic }}</span>
              <app-toggle-watched :videoId="video[0].id" :watched="video[0].watched"></app-toggle-watched>
              <app-toggle-book-marked :video="video[0]" :bookMarked="video[0].bookmarked"></app-toggle-book-marked>
            </div>
          </transition>
            <app-twitch-player ref="player" :video-id="videoId">
            </app-twitch-player>
          </div>
        <div class="poorchat" :class="{ 'poorchat--close' : !showChat }">
          <div @click="showChat = !showChat" class="poorchat__hide"><i :class="{'fas fa-eye-slash': showChat, 'fas fa-eye': !showChat}"></i></div>
          <div class="player__top player__top--left-border">
            <a target="_blank" href="https://www.poorchat.net/subscriptions/jadisco"><i class="fas fa-heart"></i>Subskrybuj czatek</a>
            <template v-if="this.streamingService === 'twitch'">
              <a @click="showChatReplay=false">Czat teraz</a>
              <a @click="showChatReplay=true">Czat powtórka</a>
            </template>
          </div>
          
          <keep-alive>
            <app-chat-replay ref="chatReplay" v-if="showChatReplay"
              :streaming-service="streamingService"
              :stream-id="videoId"
              :disable-chat-callback="() => showChatReplay = false"
              :set-topic-callback="newTopic => streamTopic = newTopic">
            </app-chat-replay>
          </keep-alive>
          <iframe v-if="renderRealChatIframe" v-show="!showChatReplay" class="poorchat__container" frameborder="0" width="100%" id="jd-chat" src="https://client.poorchat.net/jadisco"></iframe>
        </div>
    </div>
</template>
<script>
import AppTwitchPlayer from '../components/players/TwitchPlayer'
import AppChatReplay from '../components/chatReplay/ChatReplay'
import AppToggleWatched from '../UI/ToggleWatched'
import AppToggleBookMarked from '../UI/ToggleBookMarked'
import { mapActions, mapState } from 'vuex'

export default {
  data () {
    return {
      showChat: true,
      showChatReplay: false,
      renderRealChatIframe: false,
      streamTopic: null
    }
  },
  metaInfo () {
    return {
      title: !(this.streamerName in this.streamers.data) ? 'Jarchiwum' : `Jarchiwum - ${this.streamers.data[this.streamerName].info.display_name}`
    }
  },
  components: {
    AppTwitchPlayer,
    AppChatReplay,
    AppToggleWatched,
    AppToggleBookMarked,
  },
  computed: {
    ...mapState([
      'singleVideo',
      'streamers'
    ]),
    video () {
      const id = this.$route.params.id
      const userName = this.singleVideo[0].user_name
      if (userName && (userName.toLowerCase() !== id)) {
        this.$router.replace({ params: { id: userName.toLowerCase(), video: this.videoId } })
      }
      return this.singleVideo
    },
    videoURL () {
      switch(this.streamingService){
        case 'twitch':
          return `https://player.twitch.tv/?video=v${this.videoId}&autoplay=false`
        case 'youtube':
          return `https://www.youtube.com/embed/${this.videoId}?autoplay=0`
      }
    },
    mirkoLink () {
      let link
      switch(this.streamingService){
        case 'twitch':
          link = `https://player.twitch.tv/?video=v${this.videoId}&autoplay=false`
        case 'youtube':
          link = `https://www.youtube.com/embed/${this.videoId}?autoplay=0`
      }

      return `${link} (${this.streamingService})(**${this.video[0].duration}**)(_${this.video[0].title}_)`
    },
    date () {
      const date = new Date(this.video[0].published_at)
      return date.toLocaleString('nl-NL')
    },
    streamerName () {
      return this.$route.params.id
    },
    videoId() {
      return this.$route.params.video
    },
    streamingService() {
      return this.$route.query.yt === 'true' ? 'youtube' : 'twitch'
    }
  },
  methods: {
    ...mapActions([
      'getSingleVideo'
    ]),
    copyMirko () {
      this.$refs.mirkoInput.setAttribute('type', 'text')
      this.$refs.mirkoInput.select()
      document.execCommand('copy')
      this.$refs.mirkoInput.setAttribute('type', 'hidden')
    },
    getVideo () {
      const videoData = {
        streamer: this.$route.params.id,
        video: this.$route.params.video,
        isYoutube: this.$route.query.yt !== 'false'
      }
      this.getSingleVideo(videoData)
    }
  },
  watch: {
    '$route' () {
      this.getVideo()
    },
    showChatReplay(oldValue, newValue) {
      if(!oldValue)
        this.renderRealChatIframe = true
    }
  },
  created () {
    this.getVideo()
    this.showChatReplay = this.streamingService === 'twitch'
  },
  mounted() {
    if(this.$refs.chatReplay)
      this.$refs.chatReplay.setPlayer(this.$refs.player)
  }
}
</script>
