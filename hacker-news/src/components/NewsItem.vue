<template>
  <div class="news__item">
    <button class="news__button" @click="goBack">Назад</button>
    <h1>{{ newsItem.title }}</h1>
    <p>Author: {{ newsItem.by }}</p>
    <p>Date: {{ formatDate(newsItem.time) }}</p>
    <p>Comments: {{ newsItem.comments }}</p>
    <a :href="newsItem.url" target="_blank">Read More</a>
    <button class="news__button" @click="refreshComments">Обновить коментарии</button>
    <ul v-if="commentsList.length">
      <li v-for="comment in commentsList" :key="comment.id">
        <p>{{ comment.content }}</p>
        <ul v-if="comment.children.length">
          <li v-for="childComment in comment.children" :key="childComment.id">
            <p>{{ childComment.content }}</p>
          </li>
        </ul>
      </li>
    </ul>
  </div>
</template>

<script>
import axios from 'axios';


export default {
  data() {
    return {
      newsItem: {},
      commentsList: []
    };
  },
  created() {
    this.fetchNewsItem();
    this.fetchComments();
    setInterval(this.fetchComments, 60000);
  },
  methods: {
    fetchNewsItem() {
      const newsId = this.$route.params.id;
      axios.get(`https://api.hnpwa.com/v0/item/${newsId}.json`)
        .then(response => {
          this.newsItem = response.data;
        })
    },
    fetchComments() {
      const newsId = this.$route.params.id;
      axios.get(`https://api.hnpwa.com/v0/item/${newsId}.json`)
        .then(response => {
          this.commentsList = response.data.comments;
        })
    },
    refreshComments() {
      this.fetchComments();
    },
    formatDate(timestamp) {
      const date = new Date(timestamp * 1000);
      return date.toLocaleString();
    },
    goBack() {
      this.$router.push('/');
    }
  }
};
</script>

<style>
  .news__item {
    max-width: 600px;
    margin: 0 auto;
  }
</style>
