<template>
  <div class="editor-page">
    <div class="container page">
      <div class="row">

        <div class="col-md-10 offset-md-1 col-xs-12">
          <form>
            <fieldset>
              <fieldset class="form-group">
                <input 
                  type="text" 
                  class="form-control form-control-lg" placeholder="Article Title" v-model="article.title"
                />
              </fieldset>
              <fieldset class="form-group">
                <input 
                  type="text" 
                  class="form-control" 
                  placeholder="What's this article about?" v-model="article.description"
                />
              </fieldset>
              <fieldset class="form-group">
                <textarea 
                  class="form-control" 
                  rows="8" 
                  placeholder="Write your article (in markdown)" v-model="article.body"
                ></textarea>
              </fieldset>
              <fieldset class="form-group">
                <input 
                  type="text" 
                  class="form-control" 
                  placeholder="Enter tags" 
                  v-model="newTag"
                  @keyup.enter="addTag"
                />
                <div class="tag-list">
                  <span v-for="(tag,i) in article.tagList" class="tag-default tag-pill" :key="i">
                    <i class="ion-close-round" @click="removeTag(i)"></i>
                    {{tag}}
                </span>
                </div>
              </fieldset>
              <button 
                class="btn btn-lg pull-xs-right btn-primary" type="button" 
                @click.prevent="publishArticle" :disabled="publishDisabled"
              >Publish Article</button>
            </fieldset>
          </form>
        </div>

      </div>
    </div>
  </div>
</template>

<script>
import { getArticle, createArticle, updateArticle } from "@/api/article.js";

export default {
  // 在路由匹配组件渲染之前会先执行中间件处理
  middleware: 'authenticated',
  name: 'EditorIndex',
  data() {
    return {
      publishDisabled: false,
      newTag: '',
      article: {
        title: '',
        description: '',
        body: '',
        tagList: []
      }
    }
  },
  created() {
    if (this.slug) {
      this.getArticleDetail();
    }
  },
  computed: {
    slug: {
      get() {
        return this.$route.params.slug
      }
    }
  },
  methods: {
    // 获取文章详情
    async getArticleDetail() {
      const { data } = await getArticle(this.slug);
      this.article = data.article;
    },
    // 发布文章
    async publishArticle() {
      this.publishDisabled = true;
      const { data } = this.slug 
        ? await updateArticle(this.slug, this.article)
        : await createArticle(this.article);
      if (data.article){ 
        this.$router.push({
         name:'article',
         params:{
           slug: data.article.slug
         }
       });
      }
      this.publishDisabled = false;
    },
    // 增加 tag
    addTag() {
      this.article.tagList.push(this.newTag);
      this.newTag = '';
    },
    // 删除 tag
    removeTag(i){
      this.article.tagList.splice(i, 1);
    }
  }

}
</script>

<style>

</style>
