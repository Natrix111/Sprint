<template>
  <div class="item">
    <h1 class="item__title">{{ newsItem.title }}</h1>
    <p class="item__author">Автор: {{ newsItem.by }}</p>
    <p class="item__time">Дата: {{ formatDate(newsItem.time) }}</p>
    <p class="item__comments">Счетчик комментариев: {{ newsItem.descendants }}</p>
    <button class="item__button" @click="refreshComments">Обновить комментарии</button>
    <ul>
      <li v-for="comment in comments" :key="comment.id">
        <div v-html="comment.text"></div>
        <ul v-if="comment.kids && comment.kids.length > 0">
          <li v-for="kidId in comment.kids" :key="kidId">
            <div v-html="getCommentText(kidId)"></div>
          </li>
        </ul>
      </li>
    </ul>
    <router-link to="/"><button class="item__button">Вернуться к списку новостей</button></router-link>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      newsItem: {},
      comments: []
    };
  },
  created() {
    const newsId = this.$route.params.id;
    this.fetchNewsItem(newsId);
    this.fetchComments(newsId);
    setInterval(this.fetchComments, 60000);
  },
  methods: {
    fetchNewsItem(newsId) {
      axios.get(`https://hacker-news.firebaseio.com/v0/item/${newsId}.json?print=pretty`)
    .then(response => {
        this.newsItem = response.data;
      })
        .catch(error => {
          console.error(error);
        });
    },
    fetchComments(newsId) {
      axios.get(`https://hacker-news.firebaseio.com/v0/item/${newsId}.json?print=pretty`)
    .then(response => {
        const kids = response.data.kids.slice(0, 10);
        const requests = kids.map(id => axios.get(`https://hacker-news.firebaseio.com/v0/item/${id}.json?print=pretty`));
        Promise.all(requests)
          .then(responses => {
            this.comments = responses.map(response => response.data);
          });
      })
        .catch(error => {
          console.error(error);
        });
    },
    refreshComments() {
      this.fetchComments(this.$route.params.id);
    },
    formatDate(timestamp) {
      const date = new Date(timestamp * 1000);
      return date.toLocaleString();
    },
    getCommentText(commentId) {
      const comment = this.comments.find(c => c.id === commentId);
      return comment.text;
    }
  }
};
</script>


<style>
.item {
  background: antiquewhite;
  padding-left: 10px;
}

.item__button {
  margin-bottom: 20px;
  padding: 10px 10px;
  background: #ffe148;
}

</style>





