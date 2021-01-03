<template>
  <div>
    <form class="card comment-form">
      <div class="card-block">
        <textarea class="form-control" placeholder="Write a comment..." rows="3" v-model="body"></textarea>
      </div>
      <div class="card-footer">
        <img :src="user.image" class="comment-author-img" />
        <button class="btn btn-sm btn-primary" @click.prevent="addComment">
        Post Comment
        </button>
      </div>
    </form>
    <!-- 评论 list -->
    <div
      class="card"
      v-for="comment in comments"
      :key="comment.id"
    >
      <div class="card-block">
        <p class="card-text">{{ comment.body }}</p>
      </div>
      <div class="card-footer">
        <nuxt-link class="comment-author" :to="{
          name: 'profile',
          params: {
            username: comment.author.username
          }
        }">
          <img :src="comment.author.image" class="comment-author-img" />
        </nuxt-link>
        &nbsp;
        <nuxt-link class="comment-author" :to="{
          name: 'profile',
          params: {
            username: comment.author.username
          }
        }">
          {{ comment.author.username }}
        </nuxt-link>
        <span class="date-posted">{{ comment.createdAt | date('MMM DD, YYYY') }}</span>
         <span class="mod-options" v-if="user.username === comment.author.username" @click="deleteComment(comment.id)">
          <i class="ion-trash-a"></i>
        </span>
      </div>
    </div>
  </div>
</template>

<script>
import { getComments,addComment ,deleteComment} from "@/api/article.js";
import { mapState } from 'vuex'

export default {
  name: 'ArticleComments',
  props: {
    article: {
      type: Object,
      required: true
    }
  },
  data () {
    return {
      comments: [], // 文章列表
      body: ''
    }
  },
  computed: {
    ...mapState(['user'])
  },
  async mounted () {
    const { data } = await getComments(this.article.slug);
    this.comments = data.comments;
  },
   methods: {
    async getComments() {
      const { data } = await getComments(this.article.slug);
      this.comments = data.comments;
    },
    async addComment(){
      if(!this.body) return
      const { data } = await addComment(this.article.slug,this.body);
      this.comments.unshift(data.comment);
      this.body = '';
    },
    // 删除评论
    async deleteComment(commentId){
      const { data } = await deleteComment(this.article.slug,commentId);
      this.comments = this.comments.filter(comment => comment.id !== commentId);
    },
  }
}
</script>

<style>

</style>
