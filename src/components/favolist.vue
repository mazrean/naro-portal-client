<template>
  <v-layout justify-center>
    <v-flex sm8 xs12>
      <v-layout wrap column justify-center>
        <v-layout
          row
          wrap
          class="light-blue lighten-1 white--text headline"
          justify-center
          >{{ this.userName }} FavoList
        </v-layout>

        <v-flex xs12>
          <v-card v-for="tweet in favos" :key="tweet.favoID">
            <v-card-title class="pt-0 pb-0">
              <v-layout wrap row justify-center aline-center class="title">
                {{ tweet.userName }}
              </v-layout>
              <v-layout wrap row justify-end align-center>
                {{
                  tweet.createdAt
                    .replace('T', ' ')
                    .replace('Z', '')
                    .replace('-', '/')
                    .replace('-', '/')
                }}
                <v-btn
                  flat
                  icon
                  color="pink"
                  v-bind:disabled="isPush"
                  @click="postFavo(tweet.tweetID)"
                >
                  <v-icon>favorite</v-icon>
                </v-btn>

                {{ tweet.favoNum || 0 }}
                <v-btn
                  flat
                  icon
                  color="amber darken-4"
                  v-bind:disabled="isPush"
                  @click="postPin(tweet.tweetID)"
                >
                  <v-icon>fa-thumbtack</v-icon>
                </v-btn>
              </v-layout>
            </v-card-title>
            <v-card-text class="pt-0">{{ tweet.tweet }}</v-card-text>
          </v-card>
        </v-flex>
      </v-layout>
    </v-flex>
  </v-layout>
</template>

<script>
import axios from 'axios'
import { setInterval, clearInterval } from 'timers'
export default {
  name: 'FavoList',
  data() {
    return {
      userName: null,
      favos: null,
      text: '',
      message: '',
      isPush: false,
      upDateTweetTimer: null,
    }
  },
  methods: {
    postFavo(tweetID) {
      this.isPush = true
      axios
        .post('/api/favo', {
          tweetID: tweetID,
        })
        .then(() => {
          axios.get('/api/timelineFavo/' + this.userName).then(res => {
            this.favos = res.data
            this.isPush = false
          })
        })
    },
    postPin(tweetID) {
      this.isPush = true
      axios
        .post('/api/pin', {
          tweetID: tweetID,
        })
        .then(() => {
          axios.get('/api/timelineFavo/' + this.userName).then(res => {
            this.favos = res.data
            this.isPush = false
          })
        })
    },
    changeToTimeline() {
      this.$router.push('/')
    },
    reloadTweet() {
      axios.get('/api/reloadTimeline/' + this.userName).then(res => {
        if (res === 'new message exist') {
          axios.get('/api/timelineFavo/' + this.userName).then(res => {
            this.favos = res.data
          })
        }
      })
    },
    logout() {
      axios.post('/api/logout').then(() => {
        this.$router.push('/login')
      })
    },
  },
  mounted() {
    this.upDateTweetTimer = setInterval(this.reloadTweet, 5000)
  },
  destroyed() {
    clearInterval(this.upDateTweetTimer)
  },
  beforeCreate() {
    axios.get('/api/whoAmI').then(res => {
      this.userName = res.data.userName
      axios.get('/api/timelineFavo/' + this.userName).then(res => {
        this.favos = res.data
      })
    })
  },
}
</script>
