<template>
  <div class="event-swiper">
    <span v-show="!showCaption" @click="hideEventSwiper" class="close"/>
    <swiper v-bind="swiperProps" @onShowCaption="onShowCaption" />
  </div>
</template>

<script>
import Swiper from '@/components/Swiper'
// import axios from 'axios'
import VueModal from '@/components/VueModal'
export default {
  name: 'EventsSwiper',
  props: ['elements', 'element', 'event', 'assets'],
  components: {
    Swiper,
    VueModal
  },
  data () {
    return {
      showCaption: false,
      comments: [],
      caption: ''
    }
  },
  computed: {
    /*
      Prepare params to Swiper component
      @return {Object} - Prop for Swiper component
    */
    swiperProps () {
      return {
        items: this.shuffleElements,
        assets: this.assets,
        isEventSwiper: true
      }
    },
    /*
      Used to generate a shuffle elements array
      @return element[]
    */
    shuffleElements () {
      // declare some temp variable
      var j, x, i, currentItem
      // init a shuffle  array
      var shuff = this.elements
      var largeItem = this.elements.filter(item => item.id === 0)[0]
      // push an event object to shuffle
      // element > 0 => valid element is, splice it from shuff array
      if (this.element >= 0) {
        currentItem = shuff.filter(item => item.id === this.element)[0]
        shuff.splice(shuff.indexOf(currentItem), 1)
      } else if (this.element === -1) { // element === -1, it is event, splice it from shuff array
        currentItem = largeItem

        shuff.splice(shuff.indexOf(currentItem), 1)
      }
      // shuffle elements array
      for (i = shuff.length - 1; i > 0; i--) {
        j = Math.floor(Math.random() * (i + 1))
        x = shuff[i]
        shuff[i] = shuff[j]
        shuff[j] = x
      }
      // unshift selected element (or event) to shuff
      if (currentItem) shuff.unshift(currentItem)
      return shuff
    }
  },
  methods: {
    /*
      Used to hide event swiper component
    */
    hideEventSwiper () {
      this.$emit('setElement', -2)
    },
    onShowCaption (isShow) {
      this.showCaption = isShow
    }
  }
}
</script>

<style lang="scss" scoped>
  .swiper-container .banner {
    height: 100vh;
  }

  .close {
    background-image: url('../assets/images/cross_icon.svg');
    position: fixed;
    width: 14px;
    height: 14px;
    color: white;
    z-index: 9999;
    cursor: pointer;
    opacity: 1;
    background-size: 14px 14px;
    background-repeat: no-repeat;
    padding: 30px;
    background-position: center;
    &:hover {
      color: white;
    }
  }
</style>
