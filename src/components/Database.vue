<script>
export default {
  name: 'Database',
  data() {
    return {
      movieName: null,
      name: ''
    }
  },
  methods: {
    getDataFromImdb() {
      const url = `https://search.imdbot.workers.dev/?q=${this.name}`
      axios.get(url)
        .then((response) => {
          this.movieName = response.data
        })
    }
  }
}
</script>

<template>
  <div class="container">
    <h1>MOVIE</h1>
    <input type="text" v-model="name" @keyup.enter="getDataFromImdb">
    <div class="info" v-if="movieName">
      <!-- Titolo (Link) -->
      <a :href="movieName.description[0]['#IMDB_URL']" target="_blank" rel="noopener noreferrer">
        <h3>
          <i class="fa-solid fa-film"></i> Titolo Originale:
        </h3>
        <h2>
          {{ movieName.description[0]['#TITLE'] }}
        </h2>
      </a>
      <!-- Anno Pubblicazione -->
      <h3><i class="fa-regular fa-calendar"></i> {{ movieName.description[0]['#YEAR'] }}</h3>
      <!-- Poster -->
      <a :href="movieName.description[0]['#IMDB_URL']" target="_blank" rel="noopener noreferrer">
        <img :src="movieName.description[0]['#IMG_POSTER']" alt="">
      </a>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.container {
  width: 25%;
  margin: 0 auto;
  padding: 3rem;
  text-align: center;
  background-color: rgb(12, 12, 12);

  .info {
    margin: 1rem;
  }
}
</style>
