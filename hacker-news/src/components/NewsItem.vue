<template>
  <div>
    <button @click="goBack">Back to news list</button>
    <h2>{{ news.title }}</h2>
    <p>Author: {{ news.by }}</p>
    <p>Date: {{ formatDate(news.time) }}</p>
    <p>Comments: {{ news.descendants }}</p>
    <a :href="news.url" target="_blank">Read more</a>
    <ul v-if="comments.length">
      <li v-for="comment in comments" :key="comment.id">
        <p>{{ comment.text }}</p>
        <ul v-if="comment.kids && comment.kids.length">
          <li v-for="childComment in comment.kids" :key="childComment.id">
            <p>{{ childComment.text }}</p>
          </li>
        </ul>
      </li>
    </ul>
  </div>
</template>

<script>
import axios from "axios";

export default {
  props: {
    newsId: {
      type: Number,
      required: true
    }
  },
  data() {
    return {
      news: {},
      comments: []
    };
  },
  methods: {
    async fetchNewsItem() {
      try {
        const response = await axios.get(
          `https://hacker-news.firebaseio.com/v0/item/${this.newsId}.json`
      );
        this.news = response.data;
        if (this.news.kids) {
          const commentPromises = this.news.kids.slice(0, 10).map(id =>
            axios.get(`https://hacker-news.firebaseio.com/v0/item/${id}.json`)
        );
          const commentData = await Promise.all(commentPromises);
          this.comments = commentData.map(comment => comment.data);
        }
      } catch (error) {
        console.error(error);
      }
    },
    formatDate(timestamp) {
      const date = new Date(timestamp * 1000);
      return date.toLocaleString();
    },
    goBack() {
      this.$router.push("/");
    }
  },
  mounted() {
    this.fetchNewsItem();
  }
};
</script>




