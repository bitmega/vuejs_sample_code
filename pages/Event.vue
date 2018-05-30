<template>
  <div class="events-page">
    <div v-if="elements.length">
      <events-swiper v-if="element > -2" :elements="elements" :assets="assets" :element="element" v-on:setElement="setElement" :event="event" />
      <events-grid :class="{hidden: element > -2}" v-on:setElement="setElement" :elements='elements' :event='event' />
    </div>
  </div>
</template>
<script>
import { HTTP } from '@/support/http-common'
import EventsGrid from '@/components/EventsGrid'
import EventsSwiper from '@/components/EventsSwiper'

export default {
  name: 'Event',
  components: {
    EventsGrid,
    EventsSwiper
  },
  data () {
    return {
      event: {},
      elements: [],
      assets: [],
      errors: [],
      element: -2
    }
  },
  created () {
    this.fetchEvent()
  },
  methods: {
    fetchEvent () {
      HTTP.get(`da/playlists/${this.$route.params.id}/playlist_demo_items/${this.$route.params.event_id}.json`)
      .then(response => {
        var index = 1
        this.event = response.data
        this.elements = this.event.page.layout.elements
        this.assets = this.event.page.assets_containers
        if (this.elements.length > 0) {
          var tempList = []
          this.elements.map(object => {
            if (object.frame) {
              tempList.push({
                id: index++,
                type: 'frame',
                value: object.frame.picture.properties.url,
                high_value: object.frame.picture.properties.high_res_url,
                asset_id: object.frame.picture.properties.asset_id,
                view: object.frame.picture.view})
            } else if (object.paragraph) {
              tempList.push({
                id: index++,
                type: 'paragraph',
                value: object.paragraph.text.properties.text,
                view: object.paragraph.text.view})
            }
          })
          tempList.unshift({
            id: 0,
            type: 'frame',
            value: this.event.preview_image_url,
            view: ''})

          this.elements = tempList
        }
      })
      .catch(e => { this.errors.push(e) })
    },
    setElement (element) {
      this.element = element
    }
  }
}
</script>

<style>

</style>
