<template>
  <div class="article-meta">
    <nuxt-link :to="{
      name: 'profile',
      params: {
        username: article.author.username
      }
    }">
      <img :src="article.author.image" />
    </nuxt-link>
    <div class="info">
      <nuxt-link class="author" :to="{
        name: 'profile',
        params: {
          username: article.author.username
        }
      }">
        {{ article.author.username }}
      </nuxt-link>
      <span class="date">{{ article.createdAt | date('MMMM DD, YYYY') }}</span>
    </div>
    <!-- 是自己的文章 -->
    <template v-if="article.author.username === user.username">
      <nuxt-link :to="{
          name:'editor',
          params:{
            slug: article.slug
          }
        }"
        exact
        class="btn btn-sm btn-outline-secondary"
      >
        <i class="ion-edit"></i>&nbsp;Edit Article
      </nuxt-link>
      <button :disabled="article.deleteDisabled" class="btn btn-outline-danger btn-sm" @click.prevent="delArticle(article)">
        <i class="ion-trash-a"></i>&nbsp;Delete Article
      </button>
    </template>
    <!-- 别人的文章 -->
    <template v-else>
      <button
        class="btn btn-sm btn-outline-secondary"
        :class="{
          active: article.author.following
        }"
         @click="onFollow(article)"
      >
        <i class="ion-plus-round"></i>
        &nbsp;{{ article.author.following ? 'Unfollow':'Follow'}}&nbsp;{{ article.author.username }}
      </button>
      &nbsp;&nbsp;
      <button
        class="btn btn-sm btn-outline-primary"
        :class="{
          active: article.favorited
        }"
        @click="onFavorite(article)"
      >
        <i class="ion-heart"></i>
        &nbsp;{{article.favorited?'Unfavorite Article':'Favorite Article'}}&nbsp;<span class="counter">({{ article.favoritesCount }})</span>
      </button>
    </template>
  </div>
</template>

<script>
import { mapState } from "vuex";
import {
  addFollow,
  deleteFollow,
  deleteFavorite,
  addFavorite,
  deleteArticle
} from "@/api/article.js";

export default {
  name: 'ArticleMeta',
  props: {
    article: {
      type: Object,
      required: true
    }
  },
  computed: {
    ...mapState(['user'])
  },
  methods: {
    async onFollow(article) {
      article.followDisabled = true; // 禁用点击
      if (article.author.following) {
        // 取消关注
        await deleteFollow(article.author.username);
        article.author.following = false;
      } else {
        // 添加关注
        await addFollow(article.author.username);
        article.author.following = true;
      }
      article.followDisabled = false; // 允许点击
    },
    async onFavorite(article) {
      article.favoriteDisabled = true; // 禁用点击
      if (article.favorited) {
        // 取消点赞
        await deleteFavorite(article.slug);
        article.favorited = false;
        article.favoritesCount -= 1;
      } else {
        // 添加点赞
        await addFavorite(article.slug);
        article.favorited = true;
        article.favoritesCount += 1;
      }
      article.favoriteDisabled = false; // 允许点击
    },
    async delArticle(article){
      article.deleteDisabled = true;
      const { data } = await deleteArticle(article.slug);
      this.$router.push('/');
      article.deleteDisabled = false;
    }
  }
}
</script>

<style>

</style>
