<template>
  <div class="swiper-container">
    <div class="header-title" v-show="!isEventSwiper">
      <div class="banner-title">
        <div class="title-content">
          <div class="title">Test</div>
          <hr>
          <p class="banner-sub-title">ANOTHER TEST</p>
        </div>
      </div>
    </div>
    <swiper ref="awesomeSwiper" v-if="filterImageItems" :options="swiperOption">
      <swiper-slide v-for="(item, index) in filterImageItems" :key="index">
        <photo-info v-if="showCaption" @hideCaption="toggleShowCaption" :caption="getImageInfo(item).caption" :comments="getImageInfo(item).comments"/>
        <span v-show="checkShowImageInfo(item)" class="show-caption" @click="toggleShowCaption"/>
        <div @dblclick="setIsZoom(true)"
          :style="bannerStyle(item)"
          :class="bannerClass"
          class="banner banner-img">
        </div>
        <div class="banner croppa-banner" :class="{ mobile: mobile, hidden: (isEventSwiper && !isZoom) }" v-show="isEventSwiper">
          <croppa
            placeholder=""
            :disable-drag-and-drop="true"
            :disable-click-to-choose="true"
            :disable-scroll-to-zoom="true"
            :width="clientWidth"
            :height="clientHeight"
            :disabled="false"
            :prevent-white-space="true"
            :show-remove-button="false">
            <img :src="zoomedUrl(item)" alt="url" slot="initial">
          </croppa >
        </div>
      </swiper-slide>
      <div :class="{hidden: isEventSwiper}" class="swiper-pagination swiper-pagination-bullets" slot="pagination"></div>
      <div :class="{hidden: showCaption}" class="swiper-button-prev swiper-button-white" slot="button-prev"></div>
      <div :class="{hidden: showCaption}" class="swiper-button-next swiper-button-white" slot="button-next"></div>
    </swiper>
  </div>
</template>

<script>
import PhotoInfo from '@/components/PhotoInfo'

export default {
  components: {
    PhotoInfo
  },
  props: ['items', 'assets', 'isEventSwiper'],
  data () {
    return {
      showCaption: false,
      isZoom: false,
      clickTimer: 0,
      clientHeight: window.innerHeight,
      clientWidth: window.innerWidth,
      drapMode: false,
      mobile: false,
      targetSlidePrev: null,
      targetSlideNext: null,
      swiperOption: {
        speed: 600,
        loop: true,
        pagination: {
          el: '.swiper-pagination',
          clickable: true
        },
        navigation: {
          nextEl: '.swiper-button-next',
          prevEl: '.swiper-button-prev'
        }
      }
    }
  },
  mounted () {
    if (this.isEventSwiper && this.swiper) {
      var images = document.getElementsByClassName('banner-img')
      var canvases = document.getElementsByTagName('canvas')
      this.addTouchListener(images)
      this.addTouchListener(canvases)
      Array.from(canvases).forEach(element => {
        if (/Android|iPhone/i.test(navigator.userAgent)) return 0
        element.click = null
        element.addEventListener('dblclick', () => {
          this.isZoom = false
        })
      })
    }
  },
  created () {
    if (this.isEventSwiper) {
      window.onresize = () => {
        this.clientHeight = window.innerHeight
        this.clientWidth = window.innerWidth
        if (window.innerHeight > window.innerWidth) {
          this.mobile = true
        }
      }
    }
  },
  computed: {
    bannerClass () {
      return {
        mobile: this.mobile,
        hidden: this.isEventSwiper && this.isZoom,
        'full-height': this.isEventSwiper
      }
    },
    swiper () {
      return this.$refs.awesomeSwiper.swiper
    },

    /*
      Filter image items when display slide show
      When click a image in grid view, slide show only display images
    */
    filterImageItems () {
      if (this.isEventSwiper) {
        return this.items.filter(item => item.type === 'frame')
      } else {
        return this.items
      }
    }
  },
  methods: {

    /*
      Defind image use to show when double click a image in the gird
    */
    zoomedUrl (item) {
      return item.preview_image_url ? item.preview_image_url : item.value
    },

    /*
      Defind to show close button and slide show translate button
    */
    toggleShowCaption () {
      this.showCaption = !this.showCaption
      this.swiper.allowSlideNext = !this.swiper.allowSlideNext
      this.swiper.allowSlidePrev = !this.swiper.allowSlidePrev
      this.swiper.allowTouchMove = !this.swiper.allowTouchMove
      this.$emit('onShowCaption', this.showCaption)
    },
    addTouchListener (elements) {
      Array.from(elements).forEach(element => {
        element.addEventListener('touchstart', () => {
          var self = this
          if (self.clickTimer === 0) {
            self.clickTimer = setTimeout(() => { self.clickTimer = 0 }, 300)
          } else {
            clearTimeout(self.clickTimer)
            self.clickTimer = 0
            self.isZoom = !self.isZoom
          }
        }, false)
      })
    },

    /*
      Checking zoom image or not
    */
    setIsZoom (isZoom) {
      if (this.isEventSwiper) {
        this.isZoom = isZoom
      }
    },

    /*
      Defind place get image url for image slide show
    */
    bannerStyle (item) {
      if (item.type === 'frame') {
        return {
          backgroundImage: 'url(' + (item.value) + ')',
          backgroundSize: this.isEventSwiper ? 'contain' : 'cover',
          cursor: this.isEventSwiper ? 'pointer' : 'normal'
        }
      } else {
        return {
          backgroundImage: 'url(' + (item.preview_image_url) + ')',
          backgroundSize: this.isEventSwiper ? 'contain' : 'cover',
          cursor: this.isEventSwiper ? 'pointer' : 'normal'
        }
      }
    },

    /*
      Get information to image when display slide show
    */
    getImageInfo (item) {
      return (item.asset_id) ? this.assets.filter(asset => asset.asset_id === item.asset_id)[0] : {'caption': null, 'comments': []}
    },

    /*
      Checking to show image information when display slide show
    */
    checkShowImageInfo (item) {
      item = this.getImageInfo(item)
      return (this.isEventSwiper && !this.showCaption && (item.caption || (item.comments && item.comments.length)))
    }

  }
}
</script>

<style lang='scss' scoped>
  @import "../assets/stylesheets/presentation/medium_lower_screen.scss";
  @import "../assets/stylesheets/presentation/medium_upper_screen.scss";
  @import "../assets/stylesheets/presentation/presentation.scss";
</style>
