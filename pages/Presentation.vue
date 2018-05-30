<template lang="html">
  <div class="presentation" :class="getMode">
    <transition name="normal">
      <vue-modal v-show="isLoading && !imagesLoaded">
        <img slot="body" class="spinner" src="../assets/images/loading.gif" alt="loading...">
      </vue-modal>
    </transition>
    <vue-modal v-show="showModal && !getCookie()">
      <p slot="body">{{ modalTitle }}</p>
      <button slot="footer" class="btn" @click="doGotIt()">Got it</button>
    </vue-modal>


    <swiper :items="playlistData.playlist_items" :zoom="0" />

    <div class='welcome-message'>
      <div class='welcome-message-content'>
        <p class='welcome-title'>Welcome Message Here</p>
        <p class='welcome-content'>Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book.m</p>
      </div>
    </div>

    <b-row v-if="playlistData.playlist_items"
      v-for="(item, index) in playlistData.playlist_items"
      :class="playlistClass(index)"
      :key="item.id">
      <img :src="item.preview_image_url" class="hidden" @load="imageLoaded">
      <b-col cols="12" md="6" class='col-content' :class="orderClass(index).content">
        <div class='content'>
          <div class='title'>{{ item.title }}</div>
          <div class='sub-title'>
            <p>Lorem Ipsum is simply dummy text</p>
            <p>Lorem Ipsum is simply dummy text</p>
          </div>
          <router-link
            class="btn btn-flip"
            :to="{ name: 'Event', params: { id: $route.params.id, event_id: item.id}, query: { largeImageSide: getSidelargeImage(index)} }">
            Flip it
          </router-link>
        </div>
      </b-col>
      <b-col cols="12" md="6" class='col-image' :class="orderClass(index).image"
        :style='{ backgroundImage: "url(" + item.preview_image_url + ")"}'>
      </b-col>
    </b-row>
    <b-row class="footer">
      <b-col cols="12">
        <button class="btn btn-preview">Preview/Purchase</button>
        <button class="btn btn-personalize" disabled="true">Personalize</button>
        <button class="btn btn-publish" disabled="true">Publish</button>
      </b-col>
    </b-row>
  </div>
</template>

<script>
  import VueModal from '@/components/VueModal'
  import Swiper from '@/components/Swiper'
  import Spinner from '@/components/Spinner'
  import { HTTP } from '@/support/http-common'

  import mixins from '@/support/mixins'

  export default {
    name: 'Presentation',
    mixins: [mixins],
    data () {
      return {
        errors: [],
        playlistData: {},
        isLoading: true,
        showModal: true,
        imagesLoaded: false,
        modalTitle: 'Welcome to your preview edition message goes here: Lorem ipsum dolor sit amet, consectetur adip iscing elit, sed do eiusmod tempor incididunt uatq labore et dolore magna aliqua. exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.',
        loadedItemsCount: 0
      }
    },
    components: {
      VueModal,
      Spinner,
      Swiper
    },
    created () {
      this.fetchPlaylistData()
      window.document.body.className += 'noscroll'
    },
    computed: {
      getMode () {
        return (this.playlistData.mode || this.$route.query.mode || 'landscape')
      }
    },
    methods: {
      doGotIt () {
        this.showModal = false
        this.isLoading = true
        window.document.body.className -= 'noscroll'
        document.cookie = `presentation_${this.$route.params.id}=true`
      },
      getCookie () {
        var replace = '(?:(?:^|.*;\\s*)' + `presentation_${this.$route.params.id}` + '\\s*\\=\\s*([^;]*).*$)|^.*$'
        var re = new RegExp(replace)
        var str = document.cookie.replace(re, '$1')
        document.body.classList.remove('noscroll')
        return str
      },
      fetchPlaylistData () {
        var id = this.$route.params.id
        HTTP.get(`yearbooks/${id}/playlist_demo.json`)
        .then(response => {
          this.playlistData = response.data
        })
        .catch(e => { this.errors.push(e) })
      },
      playlistClass (index) {
        var suffix = index % 2 === 0 ? 'right' : 'left'
        return `present-content-image-${suffix}`
      },
      getSidelargeImage (index) {
        var side = index % 2 === 0 ? 'right' : 'left'
        return side
      },
      orderClass (index) {
        if (index % 2 === 0) {
          return { content: 'order-1', image: 'order-2' }
        } else {
          return { content: 'order-2', image: 'order-1' }
        }
      },
      imageLoaded (event) {
        this.loadedItemsCount += 1
        if (this.loadedItemsCount === this.playlistData.playlist_items.length) {
          this.isLoading = false
          this.imagesLoaded = true
        }
      }
    }
  }
</script>
<style lang='scss' scoped>
  @import  '../assets/stylesheets/presentation/medium_lower_screen.scss';
  @import  '../assets/stylesheets/presentation/medium_upper_screen.scss';
  @import  '../assets/stylesheets/presentation/presentation.scss';
</style>
