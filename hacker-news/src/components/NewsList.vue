<template>
  <div class="news">
    <h1 class="news__head">Список постов</h1>
    <button class="news__button" @click="refreshNews">ОБНОВИТЬ</button>
    <p class="news__error" v-if="isLoading">LOADING...</p>
    <ul class="news__list" v-if="!isLoading">
      <li  class="news__list-item" v-for="newsItem in newsList" :key="newsItem.id">
        <router-link class="news__item-title" :to="{ name: 'NewsItem', params: { id: newsItem.id } }"><h2 class="news__item-title">{{ newsItem.title }}</h2></router-link>
        <p class="news__item-score">Рейтинг: {{ newsItem.score }}</p>
        <p class="news__item-author">Автор: {{ newsItem.by }}</p>
        <p class="news__item-date">Дата публикации: {{ formatDate(newsItem.time) }}</p>
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref, onMounted, } from 'vue';
import axios from 'axios';

const newsList = ref([]);
const isLoading = ref(false);

const getNews = () => {
  isLoading.value = true;
  axios.get(`${process.env.VUE_APP_URL}v0/topstories.json?print=pretty`)
    .then(response => {
      const top100News = response.data.slice(0, 100);
      const promises = top100News.map(newsId => {
        return axios.get(`${process.env.VUE_APP_URL}v0/item/${newsId}.json?print=pretty`);
      });
      Promise.all(promises)
        .then(responses => {
          const newsListValue = responses.map(response => response.data);
          newsList.value = newsListValue.sort((a, b) => b.time - a.time);
          isLoading.value = false;
        });
    })
};

const formatDate = (timestamp) => {
  const date = new Date(timestamp * 1000);
  return date.toLocaleString();
};

const refreshNews = () => {
  getNews();
};

onMounted(() => {
  getNews();
  setInterval(() => {
    getNews();
  }, 60000);
});

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
  color: black;
}

</style>
