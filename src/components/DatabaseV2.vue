<script>
export default {
  name: 'DatabaseV2',
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

      // Seconda chiamata Axios
      const imdbUrl = `https://search.imdbot.workers.dev/?tt=${this.streamingData.result[0].imdbId}`;
      try {
        const imdbResponse = await axios.get(imdbUrl);
        this.movieName = imdbResponse.data;
      } catch (error) {
        console.error('Errore nella chiamata IMDb:', error);
      }
    },

    transformStreamingType(streamingType) {
      if (streamingType === 'rent') {
        return 'Noleggia';
      } else if (streamingType === 'buy') {
        return 'Compra';
      } else if (streamingType === 'subscription') {
        return 'Guarda';
      } else if (streamingType === 'addon') {
        return 'Apple TV (iTunes)';
      }
      return streamingType;
    },

    transformQuality(quality) {
      if (quality === 'sd') {
        return 'SD';
      } else if (quality === 'hd') {
        return '1080p';
      } else if (quality === 'uhd') {
        return '4K';
      }
      return quality;
    },

    transformService(service) {
      if (service === 'netflix') {
        return 'Netflix';
      } else if (service === 'apple') {
        return 'Apple TV';
      } else if (service === 'prime') {
        return 'Prime Video';
      } else if (service === 'now') {
        return 'Now TV';
      } else if (service === 'disney') {
        return 'Disney Plus';
      }
      return service;
    },

    formatNumber(number) {
      return number.toLocaleString('it-IT');
    },
  }
}
</script>

<template>
  <div class="container">

    <!-- Searchbar -->
    <div class="search">
      <input type="text" v-model="name" @keyup.enter="getDataFromImdb">
      <button @click="getDataFromImdb">Cerca</button>
    </div>

    <div class="data-poster">

      <!-- Data -->
      <div class="data">
        <div v-if="movieName && streamingData">

          <!-- Film -->
          <div class="movie" v-if="movieName.short['@type'] === 'Movie'">

            <!-- Titolo Ita (Link) -->
            <div class=" title-it">
              <h3>
                <i class="fa-solid fa-film"></i> Titolo:
                <a :href="`https://www.imdb.com/title/${streamingData.result[0].imdbId}`" target="_blank">
                  <h3>
                    {{ movieName.top.titleText.text }}
                  </h3>
                </a>
              </h3>
            </div>

            <!-- Titolo (Link) -->
            <div class="title">
              <h3>
                <i class="fa-solid fa-film"></i> Titolo Originale:
                <a :href="`https://www.imdb.com/title/${streamingData.result[0].imdbId}`" target="_blank">
                  <h3>
                    {{ movieName.top.originalTitleText.text }}
                  </h3>
                </a>
              </h3>
            </div>

            <!-- Regista-->
            <div class="director">
              <h3>
                <a :href="movieName.short.director[0].url" target="_blank">
                  <i class="fa-solid fa-user-large"></i> {{ movieName.short.director[0].name }}
                </a>
              </h3>
            </div>

            <!-- Anno Pubblicazione-->
            <div class="year">
              <h3><i class="fa-regular fa-calendar"></i> {{ movieName.top.releaseDate.year }}</h3>
            </div>

            <!-- Ratings -->
            <div class="ratings">
              <img src="../assets/imdb.png" alt="">
              <h3>{{ movieName.short.aggregateRating.ratingValue }}
                ({{ formatNumber(movieName.short.aggregateRating.ratingCount) }} Voti)
              </h3>
            </div>

            <!-- Durata -->
            <div class="time">
              <h3><i class="fa-solid fa-clock"></i> {{ movieName.top.runtime.displayableProperty.value.plainText }}</h3>
            </div>

            <!-- Streaming -->
            <div class="streaming">
              <h3><i class="fa-solid fa-tv"></i> Streaming:</h3>
              <!-- Piattaforme -->
              <div class="platforms">
                <div v-for="(item, index) in streamingData.result[0].streamingInfo.it" :key="index">
                  <div id="service">{{ transformService(item.service) }}:</div>

                  <div id="service-name" v-if="item.streamingType === 'subscription'">
                    <a :href="item.link" target="_blank">
                      {{ transformStreamingType(item.streamingType) }}
                    </a>
                  </div>

                  <div id="service-name" v-if="item.streamingType !== 'subscription'">
                    <a :href="item.link" target="_blank">
                      {{ transformStreamingType(item.streamingType) }} ({{ transformQuality(item.quality) }})
                    </a>
                  </div>

                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Poster -->
      <div class="poster" v-if="movieName && streamingData">
        <a :href="`https://www.imdb.com/title/${streamingData.result[0].imdbId}`" target=" _blank">
          <img :src="movieName.short.image" alt="">
        </a>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
@use '../components/styles/partials/variables' as *;
@use '../components/styles/partials/mixins' as *;

.container {
  background-color: $color-950;
  padding: 0 4rem;
  height: 100vh;

  .search {
    display: flex;
    align-items: center;
    height: 80px;

    button {
      background-color: $color-500;
      color: whitesmoke;
      margin-left: 1rem;

      &:hover {
        background-color: $color-600;
        cursor: pointer;
      }

      &:active {
        background-color: $color-700;
      }
    }
  }

  .data-poster {
    display: flex;
    justify-content: space-between;
    height: calc(100vh - 80px);

    .data {

      .movie {
        &>div {
          margin-bottom: 1rem;
        }

        h3 {
          display: inline-block;
        }
      }

      .ratings {
        display: flex;
        align-items: center;

        img {
          width: 2.5rem;
          margin-right: .75rem;
        }
      }

      .streaming {

        #service,
        #service-name {
          display: inline-block;
        }
      }
    }

    .poster {
      height: 100%;

      img {
        height: 100%;
        display: block;
      }
    }
  }
}
</style>