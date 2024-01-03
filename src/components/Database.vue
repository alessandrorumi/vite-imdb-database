<script>
export default {
  name: 'Database',
  data() {
    return {
      movieName: null,
      name: '',
      streamingData: null
    }
  },
  methods: {
    async getDataFromImdb() {
      // Prima chiamata Axios
      const imdbUrl = `https://search.imdbot.workers.dev/?q=${this.name}`;
      try {
        const imdbResponse = await axios.get(imdbUrl);
        this.movieName = imdbResponse.data;
      } catch (error) {
        console.error('Errore nella chiamata IMDb:', error);
      }

      // Seconda chiamata Axios
      const streamingOptions = {
        method: 'GET',
        url: 'https://streaming-availability.p.rapidapi.com/search/title',
        params: {
          title: this.name,
          country: 'it',
          show_type: 'all',
          output_language: 'en'
        },
        headers: {
          'X-RapidAPI-Key': '8950c0239cmsh617b82405d0177ep1f4730jsnd6dfc6ffac3e',
          'X-RapidAPI-Host': 'streaming-availability.p.rapidapi.com'
        }
      };

      try {
        const streamingResponse = await axios.request(streamingOptions);
        this.streamingData = streamingResponse.data;
      } catch (error) {
        console.error('Errore nella chiamata Streaming Availability:', error);
      }
    }
  }
}
</script>


<template>
  <div class="container">
    <h1>MOVIE</h1>
    <input type="text" v-model="name" @keyup.enter="getDataFromImdb">
    <div class="info" v-if="movieName && streamingData">

      <!-- Titolo AXIOS 2 (Link AXIOS 1) -->
      <a :href="movieName.description[0]['#IMDB_URL']" target="_blank" rel="noopener noreferrer">
        <h3>
          <i class="fa-solid fa-film"></i> Titolo Originale:
        </h3>
        <h2>
          {{ streamingData.result[0].title }}
        </h2>
      </a>

      <!-- Regista AXIOS 2-->
      <h3><i class="fa-solid fa-user-large"></i> {{ streamingData.result[0].directors[0] }}</h3>

      <!-- Anno Pubblicazione AXIOS 2-->
      <h3><i class="fa-regular fa-calendar"></i> {{ streamingData.result[0].year }}</h3>

      <!-- Streaming AXIOS 2 -->
      <h3><i class="fa-solid fa-tv"></i> Streaming</h3>
      <!-- Piattaforme -->
      <div class="platforms">
        <ul v-for="(item, index) in streamingData.result[0].streamingInfo.it" :key="index">
          <li>{{ item.service }}</li>
          <li>{{ item.streamingType }}</li>
          <li>
            <a :href="item.link" target="_blank" rel="noopener noreferrer">Guarda / Compra / Noleggia</a>
          </li>
        </ul>
      </div>

      <!-- Poster -->
      <a :href="movieName.description[0]['#IMDB_URL']" target="_blank" rel="noopener noreferrer">
        <img :src="movieName.description[0]['#IMG_POSTER']" alt="">
      </a>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.container {
  width: 50%;
  margin: 0 auto;
  padding: 3rem;
  text-align: center;
  background-color: rgb(12, 12, 12);

  .info {
    margin: 1rem;

    .platforms {
      margin: 1rem 0;
    }
  }
}
</style>
