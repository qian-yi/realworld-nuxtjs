<template>
  <div class="profile-page">

    <div class="user-info">
      <div class="container">
        <div class="row">

          <div class="col-xs-12 col-md-10 offset-md-1">
            <img :src="profile.image" class="user-img" />
            <h4>{{ profile.username }}</h4>
            <p>{{ profile.bio }}</p>
            <!-- 是个人资料 -->
            <template v-if="user.username === profile.username">
              <nuxt-link class="btn btn-sm btn-outline-secondary action-btn" to="/settings">
                <i class="ion-gear-a"></i> Edit Profile Settings
              </nuxt-link>
            </template>
            <!-- 是别人资料 -->
            <template v-else>
              <button
                class="btn btn-sm btn-outline-secondary action-btn"
                :disabled="profile.followDisabled"
                @click="onFollow(profile)"
              >
                <i class="ion-plus-round"></i>
                &nbsp;
                {{ profile.following?`Unfollow&nbsp;`:`Follow `}}{{profile.username }}
              </button>
            </template>
          </div>

        </div>
      </div>
    </div>

    <div class="container">
      <div class="row">

        <div class="col-xs-12 col-md-10 offset-md-1">
          <!-- tab页签 -->
          <div class="articles-toggle">
            <ul class="nav nav-pills outline-active">
              <li class="nav-item">
                <nuxt-link
                  class="nav-link"
                  :class="{ 
                    active: tab==='My'
                  }"
                  :to="{
                    name: 'profile',
                    params: {
                      username
                    }
                  }"
                  exact
                >My Articles</nuxt-link>
              </li>
              <li class="nav-item">
                <nuxt-link
                  class="nav-link"
                  :class="{
                    active:tab === 'Favorited'
                  }"
                  exact
                  :to="{
                    name: 'profile',
                    query: {
                      tab:'Favorited'
                    },
                    params: {
                      username
                    }
                  }"
                >Favorited Articles</nuxt-link>
              </li>
            </ul>
          </div>
          <!-- 对应的文章list -->
          <div class="article-preview" v-for="article in articles" :key="article.slug">
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
                }">{{ article.author.username }}</nuxt-link>
                <span class="date">{{ article.createdAt | date }}</span>
              </div>
              <button
               :disabled="article.favoriteDisabled"
                class="btn btn-sm pull-xs-right"
                :class="article.favorited?'btn-primary':'btn-outline-primary'"
                @click="onFavorite(article)"
              >
                <i class="ion-heart"></i>
                {{article.favoritesCount}}
              </button>
            </div>
            <nuxt-link :to="{
              name: 'article',
              params: {
                slug: article.slug
              }
            }" class="preview-link">
              <h1>{{article.title}}</h1>
              <p>{{article.description}}</p>
              <span>Read more...</span>
              <ul class="tag-list">
                <li
                  class="tag-default tag-pill tag-outline"
                  v-for="(tag,i) in article.tagList"
                  :key="i"
                >{{tag}}</li>
              </ul>
            </nuxt-link>
          </div>
          
          <!-- 分页 -->
          <nav>
            <ul class="pagination">
              <li
                v-for="item in totalPage"
                :key="item"
                :class="['page-item', {active:item===page}]"
              >
                <nuxt-link class="page-link" :to="{
                  name: 'profile',
                  query: {
                    page: item,tab
                  },
                  params: {
                    username
                  }
                }">{{item}}</nuxt-link>
              </li>
            </ul>
          </nav>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { mapState } from "vuex";
import { getProfiles } from "@/api/profile";
import {
  getArticles,
  addFollow,
  deleteFollow,
  deleteFavorite,
  addFavorite
} from "@/api/article";

export default {
  middleware: 'authenticated',
  name: 'UserProfile',
  watchQuery: ["tab", "page"],
  computed: {
    ...mapState(["user"]),
    totalPage (){
      return Math.ceil(this.articlesCount / this.limit)
    }
  },
  async asyncData({ query, params }) {
    const { tab = "My", page = 1 } = query;
    const { username } = params;
    const limit = 20;
    const offset = (page - 1) * limit;
    const articleType = tab === "My" ? "author" : "favorited";
    const [profileRes, articlesRes] = await Promise.all([
      getProfiles(username),
      getArticles({ 
        limit, 
        offset, 
        [articleType]: username 
      }),
    ]);
    const { profile } = profileRes.data;
    const { articles, articlesCount } = articlesRes.data;
    profile.followDisabled = false;
    articles.forEach(article => article.favoriteDisabled = false);
    return {
      tab,
      page,
      limit,
      profile,
      articles,
      articlesCount,
      username
    };
  },
   methods: {
    // 点击关注
    async onFollow(profile) {
      profile.followDisabled = true; // 禁用点击
      if (profile.following) {
        // 取消关注
        await deleteFollow(profile.username);
        profile.following = false;
      } else {
        // 添加关注
        await addFollow(profile.username);
        profile.following = true;
      }
      profile.followDisabled = false; // 允许点击
    },
    // 点赞
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
    }
  }
}
</script>

<style>

</style>
