<template>
  <div>
    <div class="header">
      我的评论
    </div>
    <alert v-show="ishowalere" class="alertbox"></alert>
    <div class="content" v-if="comments.length>0">
      <ul>
        <li class="bookbox" v-for="comment in comments" :key="comment.id">
          <div class="boxinfo">
            <div class="infobox">
              <div class="left">
                <router-link class="title" tag="div" :to="'/detail/' + comment.book.id">
                  {{comment.book.name}}
                </router-link>
                <div class="uploadtile">
                  {{comment.create_time}}
                </div>
                <div class="price">
                  ￥ {{comment.book.price}}
                </div>
              </div>
              <div class="right" @click="deleteBoxclick">
                <img :src="comment.book.images[0].url" alt="">
              </div>
            </div>
            <div class="commentbox" @click="deleteBoxclick" v-html="$options.filters.filemotion(comment.content,emotions)">
            </div>
            <div v-show="deleteBtnshow" class="buttonbox">
              <button @click="DeleteBtn" :comment-id="comment.id">删除</button>
            </div>
          </div>
        </li>
      </ul>
    </div>
    <div v-else class="none">
      空空如也......
    </div>
  </div>
</template>

<script>
import Alert from 'common/alert/Alert'
import axios from 'axios'
export default {
  name: 'MyComments',
  components: {Alert},
  comments: {
    Alert
  },
  data () {
    return {
      deleteBtnshow: false,
      comments: [],
      oldToken: '',
      ishowalere: false,
      emotions: []
    }
  },
  mounted () {
    var oldToken = JSON.parse(localStorage.getItem('token')).data
    if (oldToken) {
      this.oldToken = oldToken
      axios.post('/api/mycomments/', {token: oldToken}).then(this.handleaxios)
    }
    axios.get('/api/emotions/').then((res) => {
      this.emotions = res.data.data
    })
  },
  filters: {
    filemotion (value, emotions) {
      value = value.replace(/(\[.+?\])/g, (e, e1) => {
        for (var i in emotions) {
          if ((emotions[i].value.indexOf(e1)) > -1) {
            return '<img src="' + emotions[i].icon + '">'
          }
        }
      })
      return value
    }
  },
  methods: {
    handleemit (message, color) {
      this.$store.commit('msgchange', {message: message, color: color})
      this.ishowalere = true
    },
    handleaxios (res) {
      this.comments = res.data.comments
    },
    deleteBoxclick (e) {
      e.target.parentNode.children[2].style.display = 'block'
    },
    DeleteBtn (e) {
      var cur = e.target.parentNode.parentNode
      cur.parentNode.removeChild(cur)
      var clickcommentid = e.target.getAttribute('comment-id')
      axios.post('/api/delcomment/', {token: this.oldToken, commentid: clickcommentid}).then((res) => {
        this.handleemit('删除成功~', '#b1ff5b')
      })
    }
  }
}
</script>

<style scoped lang="stylus">
  .alertbox
    position fixed
    line-height 1rem
    bottom  0
    left 0
    right 0
  .none
    margin-top .5rem
    text-align center
    color #85846c
  .header
    width 100%
    height .86rem
    line-height .86rem
    background #ffc4be
    text-align center
    color #ffff
    .left
      float left
      margin-left .1rem
  .bookbox
    width 100%
    margin-top .1rem
    .boxinfo
      width 100%
      margin .2rem
      border-bottom .01rem solid #F2F3F4
      float left
      .infobox
        width 100%
        height 2rem
        .left
          width 60%
          margin-top .2rem
          float left
        .right
          width 30%
          float right
          img
            width 2rem
            height 2rem
            padding .1rem
            box-sizing border-box
      .commentbox
        padding .1rem
        letter-spacing 0.02rem
        color #9A913A
        font-weight lighter
      .title,.price,.buttonbox,.uploadtile
        margin-top .1rem
      .title
        font-size 0.3rem
        font-weight bold
      .uploadtile
        color #99c480
        margin-top .2rem
      .price
        margin-top .2rem
        color red
      .buttonbox
        float right
        button
          padding .1rem
          color #ffffff
          box-sizing border-box
          border-radius .1rem
          margin .3rem
</style>
