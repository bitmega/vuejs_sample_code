<template>
  <div class="caption" @click="$emit('hideCaption')">
      <div class="caption-container">
        <div class="box caption-content" v-show="caption">
          <div class="head">
            <span>Caption: </span>
          </div>
          <div class="body">
            {{ caption }}
          </div>
        </div>
        <div class="box comments"
          :class="{ 'full-comments': !caption }"
          v-show="comments && comments.length > 0">
          <div class="head">
            <span>Comments: </span>
          </div>
          <div class="body">
            <ul>
              <li :key="index" v-for="(comment, index) in comments">
                <b-row class="user" align-v="center">
                  <b-col class="cell info" cols="8">
                    <b-img :src="comment.avatar" rounded="circle" width="30" height="30" />
                    {{ comment.email }}
                  </b-col>
                  <b-col class="cell time" cols="3" offset="1">{{ comment.time | moment("ddd MM, YYYY") }}</b-col>
                </b-row>
                <div class="content">{{ comment.body }}</div>
              </li>
            </ul>
          </div>
        </div>
      </div>
    </div>
</template>
<script>
export default {
  props: ['comments', 'caption']
}
</script>
<style lang="scss" scoped>
  @import  '../assets/stylesheets/events/photo_info.scss';
</style>
