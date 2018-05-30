<template lang="html">
  <div class="event-spread" :class="getMode">
    <vue-modal v-if="showModal"/>
    <transition name="normal">
      <vue-modal v-show="isLoading && !imagesLoaded">
        <img slot="body" class="spinner" src="../assets/images/loading.gif" alt="loading...">
      </vue-modal>
    </transition>
    <div class= "event-header">
      <div class="btn-goback" @click="goBack"></div>
      <div v-if="event" class="event-title">{{ event.title }}</div>
    </div>
    <div class="events-grid-5">
      <b-row v-if="listItem.length > 0" :class="eventClass(largeImageSide)">
        <b-col cols="12" md="6" class="col-large-image" :class="orderClass(largeImageSide).largeImage">
          <div class="image-container">
            <div class="image-wrapper">
              <div v-if="largeImage.type === 'paragraph'"
                    class="large-image col-child-paragraph" id="item-large">
                <div class="child-paragraph-content"> {{largeImage.value}}</div>
              </div>
              <div v-else @click="handleClickPhoto(-1)" :style='{ backgroundImage: "url(" + largeImage.value + ")" }' class="large-image" id="item-large"></div>
            </div>
          </div>
        </b-col>
        <b-col cols="12" md="6" class="col-list-image" :class="orderClass(largeImageSide).listImage">
          <b-row v-if="listItem.length > 4" class="row-images">
            <b-col cols="6" v-for="(item, index) in listItemUnique"
              :key="index" class='col-tiny-item-grid-5'>
              <div class="image-container">
                <div class="image-wrapper">
                  <img v-if="item.type === 'frame'" :src="item.value" class="hidden" @load="imageLoaded">
                  <div v-if="item.type === 'paragraph'"
                        class="col-child-image col-child-paragraph" :id="`item-${index}`">
                    <div class="child-paragraph-content" :style= "getStyle(item.view)"> {{item.value}}</div>
                  </div>
                  <div v-else @click="handleClickPhoto(index)"
                        :style='{ backgroundImage: "url(" + item.value + ")"}'
                        class="col-child-image" :id="`item-${index}`">
                  </div>
                </div>
             </div>
            </b-col>
          </b-row>
          <b-row v-else class="row-images">
            <b-col cols="12" md="12" class="col-tiny-item-grid-2">
             <div class="image-container">
                <div class="image-wrapper">
                  <img v-if="listItemUnique[0].type === 'frame'" :src="listItemUnique[0].value" class="hidden" @load="imageLoaded">
                  <div v-if="listItemUnique[0].type === 'paragraph'"
                        class="col-child-image col-child-paragraph" :id="`item-0`">
                    <div class="child-paragraph-content" :style= "getStyle(listItemUnique[0].view)"> {{listItemUnique[0].value}}</div>
                  </div>
                  <div v-else @click="handleClickPhoto(0)" :style='{ backgroundImage: "url(" + listItemUnique[0].value + ")" }'
                      class="col-child-image"
                      :id="`item-0`"></div>
                </div>
             </div>
            </b-col>
          </b-row>
        </b-col>
      </b-row>
    </div>

  </div>
  </div>
</template>

<script>
  import Event from '@/pages/Event'
  import mixins from '@/support/mixins'
  import VueModal from '@/components/VueModal'
  import Spinner from '@/components/Spinner'

  var timeToLive = 0 // variable carry a random time to recall reloadCell
  var lastItem = 0 // variable carry a last item in shuffle number array
  var flagReloadCell
  export default {
    name: 'EventsGrid',
    mixins: [mixins],
    props: [
      'elements',
      'event',
      'dimension'
    ],
    data () {
      return {
        listItem: [],
        largeImage: {},
        listItemUnique: [],
        listItemPrepared: [],
        largeImageSide: this.$route.query.largeImageSide,
        interval: false,
        listFirstListImageLoad: [],
        loadedItemsCount: 0,
        isLoading: true,
        showModal: true,
        imagesLoaded: false,
        shuffIndex: [],
        oldLargeItem: []
      }
    },
    components: {
      Event,
      VueModal,
      Spinner
    },
    created () {
      flagReloadCell = true
      this.getListItem()
      if (this.listItem.length > 4) {
        this.getListFirstLoad(4)
      } else {
        this.getListFirstLoad(1)
      }
      if (this.listItemUnique.filter(o => o.type === 'frame').length === 0) {
        this.noneImageLoad()
      }
    },
    computed: {
      getMode () {
        return 'landscape'
      }
    },
    destroyed () {
      flagReloadCell = false
    },
    mounted () {
      this.interval = true
      if (this.elements.length > 1) this.reloadCell()
    },
    methods: {

      /*
        Used to get style of paragraph element
      */
      getStyle (view) {
        if (view) {
          return `font-family: '${view.font_family}'; color: rgb(${view.font_color.red},${view.font_color.green},${view.font_color.blue}); font-weight: ${view.text_presentation.weight}; text-decoration: ${view.text_presentation.decoration}; font-style: ${view.text_presentation.style}; text-align: ${view.text_alignment};`
        }
      },

      /*
        Used to select a element ID from url content
        @params {integer} index - Required
      */
      handleClickPhoto (index) {
        var element
        if (index >= 0) { // valid element in elements array, selects and assigns it for element variable
          element = this.listItem.filter(object => object === this.listItemUnique[index])[0]
        } else { // it is large item, use url to find element in elements array
          var img = document.getElementById('item-large')
          if (img) {
            var style = img.currentStyle || window.getComputedStyle(img, false)
            var url = style.backgroundImage.slice(5, -2).replace(/"/g, '')
          }
          element = this.listItem.filter(object => object.value === url)[0]
        }
        var id = element ? element.id : -1
        this.emitShowSwiper(id)
      },

      /*
        Get all items for show on grid event
      */
      getListItem () {
        if (this.elements.length > 0) {
          this.listItem = this.elements
        }
      },
      /*
        Get images show at the first time on grid
      */
      getListFirstLoad (count) {
        if (this.elements.length > 0) {
          /*
            get large image to grid from json
          */
          var tempListFirstLoad = this.listItem.slice(this.listItem)
          this.largeImage = tempListFirstLoad.shift()
          // var tempList = tempListFirstLoad.slice(tempListFirstLoad)
          var retList = []
          for (var i = 0; i < count; i++) {
            var index = Math.floor(Math.random() * tempListFirstLoad.length)
            var removed = tempListFirstLoad.splice(index, 1)
            retList.push(removed[0])
          }
          /*
            get list image to grid from json
          */
          this.listItemUnique = retList
          this.listItemPrepared = tempListFirstLoad.filter(item => this.listItemUnique.indexOf(item) < 0)
          this.preloadEventImage(this.listItemPrepared)
        }
      },

      /*
        Used to pick a random cell in grid and change a random image in random time
      */
      reloadCell () {
        // Unless intevel, break here
        if (!this.interval) return
        if (!this.shuffIndex.length) {
          // generate a number array from 0 to listItemUnique length
          this.shuffIndex = Array.apply(null, {length: this.listItemUnique.length}).map(Number.call, Number)
          // unshift -1 to array, it will be used to know whether event is
          this.shuffIndex.unshift(-1)
          // shuffle array until the first item of shuffle not equal to lastItem global variable
          // prevent one cell change image two times
          do {
            this.shuffIndex.sort(() => { return 0.5 - Math.random() })
          } while (this.shuffIndex[0] === lastItem)
        }
        // assigns a new lastItem
        lastItem = this.shuffIndex[this.shuffIndex.length - 1]
        // first time visit this page, timeToLive is 5s
        if (timeToLive <= 0) timeToLive = 5000
        else timeToLive = Math.floor(Math.random() * 1500) + 2000 // random timeToLive
        // Wait in tiemToLive seconds and recall reloadCell
        setTimeout(() => {
          // random an index from listItemUnique
          var rIndex = this.shuffIndex.splice(0, 1)[0]
          // pick a first item from listItemPrepared assigns to listItemUnique[rIndex]
          var nextImage = this.listItemPrepared.splice(0, 1)[0]
          if (rIndex >= 0) {
            // append picked item by rIndex from listItemUnique to listItemPrepared
            this.listItemPrepared.push(this.listItemUnique[rIndex])
            this.listItemUnique[rIndex] = nextImage
            if (document.getElementById(`item-${rIndex}`)) {
              this.translateImages(rIndex)
            }
          } else {
            var img = document.getElementById('item-large')
            if (img) {
              var style = img.currentStyle || window.getComputedStyle(img, false)
              var urlLargeImage = this.listItem.filter(object => object.value === style.backgroundImage.slice(4, -1).replace(/"/g, ''))[0]
            }
            // append picked item by rIndex from listItemUnique to listItemPrepared
            if (urlLargeImage) {
              this.listItemPrepared.push(urlLargeImage)
            } else {
              this.listItemPrepared.push(this.oldLargeItem)
            }
            if (img) {
              this.translateImages(-1, nextImage)
              this.oldLargeItem = nextImage
            }
          }
          if (flagReloadCell) {
            this.reloadCell()
          }
        }, timeToLive)
      },
      /*
        Used to trigger show EventSwiper component
      */
      emitShowSwiper (element) {
        this.$emit('setElement', element)
      },
      /*
        Defind class name for large image show at right side or left side
      */
      eventClass (side) {
        var suffix = side === 'right' ? 'right' : 'left'
        return `col-large-image-${suffix}`
      },
      orderClass (side) {
        if (side === 'left') {
          return { largeImage: 'order-1', listImage: 'order-2' }
        } else {
          return { largeImage: 'order-2', listImage: 'order-1' }
        }
      },

      /*
        Used go back the privious page
      */
      goBack () {
        this.$router.go(-1)
      },

      /*
        Used to count images were loaded in the grid
      */
      imageLoaded (event) {
        this.loadedItemsCount += 1
        if (this.loadedItemsCount === this.listItemUnique.filter(o => o.type === 'frame').length) {
          this.noneImageLoad()
        }
      },

      /*
        If there are no image in the grid, the page not show modal preload
      */
      noneImageLoad () {
        this.isLoading = false
        this.imagesLoaded = true
        this.showModal = false
      },

      /*
        Used to translate image in the cell of grid
      */
      translateImages (index, nextImage = '') {
        if (index < 0) {
          index = 'large'
        }
        var el = document.getElementById(`item-${index}`)
        var div = el.cloneNode(true)
        div.innerHTML = ''
        div.setAttribute('style', '')
        if (index === 'large') {
          this.createCellElement(div, nextImage, 'large-image', index)
        } else {
          this.createCellElement(div, this.listItemUnique[index], 'col-child-image', index)
        }
        el.parentElement.prepend(div)
        el.parentElement.style.transitionDuration = '0s'
        el.parentElement.style.transform = 'translate(-50%, 0px)'
        setTimeout(() => {
          el.parentElement.style.transitionDuration = '1s'
          el.parentElement.style.transform = 'translate(0px, 0px)'
        }, 20)
        setTimeout(() => {
          el.remove()
        }, 1500)
      },

      /*
        Create element into cell of grid view
      */
      createCellElement (divElement, object, typeCell, index) {
        if (object.type === 'frame') {
          divElement.setAttribute('class', typeCell)
          divElement.addEventListener('click', () => { this.handleClickPhoto(index) })
          divElement.style.backgroundImage = `url(${object.value})`
        } else if (object.type === 'paragraph') {
          divElement.style.backgroundImage = `none`
          if (typeCell === 'col-child-image') {
            divElement.classList.add(typeCell)
          }
          divElement.classList.add('col-child-paragraph')
          divElement.setAttribute('style', this.getStyle(object.view))
          var paragraph = this.breakParagraph(object.value)
          divElement.innerHTML = `<div ${this.$options._scopeId} class='child-paragraph-content'> ${paragraph} </div>`
        }
      },

      /*
        Break paragraph for paragraph element in grid view
      */
      breakParagraph (text) {
        return (text.indexOf('\n') >= 0) ? text.replace('\n', '<br />') : text
      }
    }
  }
</script>

<style lang='scss' scoped>
  @import '../assets/stylesheets/events/medium_lower_screen_events_grid.scss';
  @import '../assets/stylesheets/events/medium_upper_screen_events_grid.scss';
  @import '../assets/stylesheets/events/event.scss';
</style>
