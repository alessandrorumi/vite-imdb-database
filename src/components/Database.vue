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

      // Prima chiamata Axios
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
  <!-- Searchbar -->
  <div class="search">
    <input type="text" v-model="name" @keyup.enter="getDataFromImdb">
  </div>

  <div class="wrapper">
    <div class="container-data">
      <div v-if="movieName && streamingData">

        <!-- Film -->
        <div class="data" v-if="movieName.short['@type'] === 'Movie'">
          <!-- Titolo Ita (Link) -->
          <div class=" title-it">
            <h3>
              <i class="fa-solid fa-film"></i> Titolo:
            </h3>
            <a :href="`https://www.imdb.com/title/${streamingData.result[0].imdbId}`" target="_blank"
              rel="noopener noreferrer">
              <h2>
                {{ movieName.top.titleText.text }}
              </h2>
            </a>
          </div>

          <!-- Titolo (Link) -->
          <div class="title">
            <h3>
              <i class="fa-solid fa-film"></i> Titolo Originale:
            </h3>
            <a :href="`https://www.imdb.com/title/${streamingData.result[0].imdbId}`" target="_blank"
              rel="noopener noreferrer">
              <h2>
                {{ movieName.top.originalTitleText.text }}
              </h2>
            </a>
          </div>

          <!-- Ratings -->
          <div class="ratings">
            <img src="../assets/imdb.png" alt="">
            <h3>{{ movieName.short.aggregateRating.ratingValue }}
              ({{ formatNumber(movieName.short.aggregateRating.ratingCount) }} Voti)
            </h3>
          </div>

          <!-- Ranking -->
          <div class="ranking">
            <h3>
              <i class="fa-solid fa-ranking-star"></i> Ranking: {{ movieName.main.ratingsSummary.topRanking.rank }}°
            </h3>
          </div>

          <!-- Durata -->
          <div class="time">
            <h3><i class="fa-solid fa-clock"></i> {{ movieName.top.runtime.displayableProperty.value.plainText }}</h3>
          </div>

          <!-- Regista-->
          <div class="director">
            <h3>
              <a :href="movieName.short.director[0].url" target="_blank" rel="noopener noreferrer">
                <i class="fa-solid fa-user-large"></i> {{ movieName.short.director[0].name }}
              </a>
            </h3>
          </div>

          <!-- Anno Pubblicazione-->
          <div class="year">
            <h3><i class="fa-regular fa-calendar"></i> {{ movieName.top.releaseDate.year }}</h3>
          </div>

          <!-- Streaming -->
          <div class="streaming">
            <h3><i class="fa-solid fa-tv"></i> Streaming:</h3>
            <!-- Piattaforme -->
            <div class="platforms">
              <div v-for="(item, index) in streamingData.result[0].streamingInfo.it" :key="index">
                <div id="service">{{ transformService(item.service) }}:</div>
                <div v-if="item.streamingType === 'subscription'">
                  <a :href="item.link" target="_blank" rel="noopener noreferrer">
                    {{ transformStreamingType(item.streamingType) }}
                  </a>
                </div>
                <div v-if="item.streamingType !== 'subscription'">
                  <a :href="item.link" target="_blank" rel="noopener noreferrer">
                    {{ transformStreamingType(item.streamingType) }} ({{ transformQuality(item.quality) }})
                  </a>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- Serie TV -->
        <div class="data" v-if="movieName.short['@type'] === 'TVSeries'">
          <!-- Titolo Ita (Link) -->
          <div class=" title-it">
            <h3>
              <i class="fa-solid fa-film"></i> Titolo:
            </h3>
            <a :href="`https://www.imdb.com/title/${streamingData.result[0].imdbId}`" target="_blank"
              rel="noopener noreferrer">
              <h2>
                {{ movieName.top.titleText.text }}
              </h2>
            </a>
          </div>

          <!-- Titolo (Link) -->
          <div class="title">
            <h3>
              <i class="fa-solid fa-film"></i> Titolo Originale:
            </h3>
            <a :href="`https://www.imdb.com/title/${streamingData.result[0].imdbId}`" target="_blank"
              rel="noopener noreferrer">
              <h2>
                {{ movieName.top.originalTitleText.text }}
              </h2>
            </a>
          </div>

          <!-- Trama -->
          <div class="plot">
            Trama: {{ movieName.top.plot.plotText.plainText }}
          </div>

          <!-- Ratings -->
          <div class="ratings">
            <img src="../assets/imdb.png" alt="">
            <h3>{{ movieName.short.aggregateRating.ratingValue }}
              ({{ formatNumber(movieName.short.aggregateRating.ratingCount) }} Voti)
            </h3>
          </div>

          <!-- Ranking -->
          <div class="ranking" v-if="movieName.main.ratingsSummary.topRanking !== null">
            <h3>
              <i class="fa-solid fa-ranking-star"></i> Ranking: {{ movieName.main.ratingsSummary.topRanking.rank }}° Serie
              TV
            </h3>
          </div>

          <!-- Stagioni -->
          <div class="seasons">
            <h3>Stagioni: {{ movieName.main.episodes.seasons[movieName.main.episodes.seasons.length - 1].number }}</h3>
          </div>

          <!-- Episodi -->
          <div class="episodes">
            <h3>Episodi: {{ movieName.main.episodes.episodes.total }}</h3>
          </div>

          <!-- Durata complessiva -->
          <!-- <div class="time">
            <h3><i class="fa-solid fa-clock"></i> Durata Tot: {{ movieName.top.runtime.displayableProperty.value.plainText
            }}</h3>
          </div> -->

          <!-- Streaming -->
          <div class="streaming">
            <h3><i class="fa-solid fa-tv"></i> Streaming:</h3>
            <!-- Piattaforme -->
            <div class="platforms">
              <div v-for="(item, index) in streamingData.result[0].streamingInfo.it" :key="index">
                <div id="service">{{ transformService(item.service) }}:</div>
                <div v-if="item.streamingType === 'subscription'">
                  <a :href="item.link" target="_blank" rel="noopener noreferrer">
                    {{ transformStreamingType(item.streamingType) }}
                  </a>
                </div>
                <div v-if="item.streamingType !== 'subscription'">
                  <a :href="item.link" target="_blank" rel="noopener noreferrer">
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
    <div class="container-poster" v-if="movieName && streamingData">
      <a :href="`https://www.imdb.com/title/${streamingData.result[0].imdbId}`" target=" _blank"
        rel="noopener noreferrer">
        <img :src="movieName.short.image" alt="">
      </a>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.search {
  height: 40px;
  padding: 0 6rem;
  background-color: rgb(12, 12, 12);
}

.wrapper {
  display: flex;
  justify-content: space-around;
  height: calc(100vh - 40px);
  background-color: rgb(12, 12, 12);

  .container-data {
    padding: 1.5rem;
    width: 50%;

    .data {
      .title-it {
        margin-bottom: .5rem;
      }

      &>div {
        margin-bottom: 1.5rem;

        img {
          width: 7.5%;
          margin-right: .5rem;
        }
      }

      .ratings {
        display: flex;
        align-items: center;
      }

      .platforms {
        &>div {
          margin-top: .5rem;
          display: flex;

          #service {
            margin-right: .5rem;
          }
        }
      }
    }
  }

  .container-poster {
    height: 100%;

    img {
      height: 100%;
      display: block;
    }
  }
}
</style>
