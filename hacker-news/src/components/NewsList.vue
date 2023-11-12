<template>
  <div class="news">
    <h1 class="news__head">Список постов</h1>
    <button class="news__button" @click="refreshNews">ОБНОВИТЬ</button>
    <p class="news__error" v-if="isLoading">LOADING...</p>
    <ul class="news__list" v-if="!isLoading">
      <li  class="news__list-item" v-for="newsItem in newsList" :key="newsItem.id" @click="goToNewsItem(newsItem.id)">
        <h2 class="news__item-title">{{ newsItem.title }}</h2>
        <p class="news__item-score">Рейтинг: {{ newsItem.score }}</p>
        <p class="news__item-author">Автор: {{ newsItem.by }}</p>
        <p class="news__item-date">Дата публикации: {{ formatDate(newsItem.time) }}</p>
      </li>
    </ul>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      newsList: [],
      isLoading: false
    };
  },
  mounted() {
    this.getNews();
    setInterval(() => {
      this.getNews();
    }, 60000);
  },
  methods: {
    getNews() {
      this.isLoading = true;
      axios.get('https://hacker-news.firebaseio.com/v0/topstories.json?print=pretty')
        .then(response => {
          const top100News = response.data.slice(0, 100);
          const promises = top100News.map(newsId => {
            return axios.get(`https://hacker-news.firebaseio.com/v0/item/${newsId}.json?print=pretty`);
          });
          Promise.all(promises)
            .then(responses => {
              const newsList = responses.map(response => response.data);
              this.newsList = newsList.sort((a, b) => b.time - a.time);
              this.isLoading = false;
            });
        })
    },
    formatDate(timestamp) {
      const date = new Date(timestamp * 1000);
      return date.toLocaleString();
    },

    refreshNews() {
      this.getNews();
    },
    goToNewsItem(newsId) {
      this.$router.push({ name: 'NewsItem', params: { id: newsId } });
    }
  }
};
</script>

<style>
.news {
  max-width: 600px;
  margin: 0 auto;
}

.news__head {
  text-align: center;
}

.news__button {
  margin-bottom: 20px;
  padding: 10px 10px;
  background: #ffe148;
}

.news__error {
  font-size: 20px;
}

.news__list {
  list-style-type: none;
  padding: 0;
}

.news__list-item {
  margin-bottom: 20px;
  border: 2px solid #000000;
  border-radius: 5px;
  padding: 0 10px;
  background-color: #ffe148;
}

.news__item-title {
  font-size: 18px;
}

</style>
