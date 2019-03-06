<template>
  <!-- Code from designer -->
  <div class="dashboard" v-bind:style="{ 'background-image': 'url(' + backgroundImage + ')' }">
    <div class="overlay">
      <div>
          <div class="tile is-ancestor">
            <div class="tile is-vertical is-8" style="margin-top: 40px;">
              <div class="tile animated fadeInLeft" v-on:click="setPlace()">
                <div class="tile is-parent is-vertical">
                  <article class="tile is-child">
                    <div style="background-color: rgba(0,0,0,0.7); padding: 10px 10px 10px 40px; width: 50%;">
                    <div class="title" ><span style="font-size: 0.65em; margin-right: 10px; vertical-align: middle;">
                      <i class="fa fa-map-marker-alt" aria-hidden="true"></i></span>{{city}} <span style="font-family: 'OstrichSans'">|</span> {{country}}
                    </div>
                    </div>
                  </article>
                </div>
              </div>
              <div class="tile is-parent animated fadeInLeft" style="margin-top: 30%;">
                <article class="tile is-child">
                    <div class="weather level level-left" style="background-color: rgba(0,0,0,0.7); padding: 10px;">
                      <img :src="weatherIcon" style="vertical-align: middle;">
                      <span class="has-text-white is-size-1">{{currentTemp}}°{{currentTempUnit}}</span>
                      <span class="has-text-white is-size-3" style="font-family:'OstrichSans'; margin-left: 10px;">{{currentTempText}}</span>
                    </div>
                </article>
              </div>
            </div>
            <div class="tile is-parent animated fadeInDown" style="background-color: rgba(0,0,0,0.6); padding-top: 40px; padding-left: 20px; margin-right: 5%;">
              <article class="tile is-child">
                <div class="subtitle"><span style="font-size: 0.65em; margin-right: 10px; vertical-align: middle;">
                      <i class="fa fa-newspaper" aria-hidden="true"></i></span>NEWS</div>
                <div v-for="newsItem in news" v-bind:key="newsItem.name">
                    <!-- Content -->
                    <article class="media animated fadeInDown" style="margin-bottom: 24px; padding: 8px;">
                      <figure class="media-left" v-if="newsItem.image">
                        <p class="image is-64x64">
                          <img v-bind:src="newsItem.image.thumbnail.contentUrl" style="border: 1px white solid; margin-top: 8px;">
                        </p>
                      </figure>
                      <div class="media-content has-text-white is-size-5" style="letter-spacing: 0.7px;">
                        {{newsItem.name}}
                      </div>
                    </article>
                  </div>
              </article>
            </div>
          </div>
        </div>
      </div>
    </div>
</template>

<script>
// We are going to use momentJS for the timezone conversion of local time
const unirest = require('unirest');
const RapidAPI = require('rapidapi-connect');
const rapid = new RapidAPI("default-application_5b149ddfe4b083052c9cda3a", "770c6c7b-d26b-49c7-8154-2836ff6722c0");

export default {
  name: 'Dashboard',
  data() {
    return {
      // Define the data values which we will populate from the APIs

      // Location variables
      city: '',
      country: '',
      locale: 'ja-JP',
      lang: 'ja',

      // News API variables
      news: [],

      // Image API variables
      backgroundImage: '',

      // Weather API variables
      currentTemp: '0',
      currentTempUnit: 'C',
      currentTempText: '',
      currentTempCode: 0 // used for the weather icon
    }
  },
  computed: {
    // We need to require the weather icon here as a computed variable so that webpack can find it
    // Since webpack replaces the url for the icon with a random string one
    weatherIcon () {
      // return require('../assets/icons/' + this.currentTempCode + '.svg')
    }
  },
  methods: {
    getMyLocation(parent){
      unirest.get("https://ipinfo.p.rapidapi.com/json")
        .header("X-RapidAPI-Key", "jrNHGDmcGhmshBBOCny0I5RRti8hp1EK4GcjsnnwyItFhUUMZJ")
        .end(function (result) {
          // parent.city = result.body.city
          // parent.country = result.body.country
          parent.city = 'Tokyo'
          parent.country = 'JP'
          parent.getNews(parent)
          parent.getWeather(parent)
          parent.getPhotos()
        })
    },
    getWeather(parent){
      // Get the location key first
      unirest.get(`https://community-open-weather-map.p.rapidapi.com/weather?units=metric&q=${parent.city}%2C${parent.country}`)
      .header("X-RapidAPI-Key", "jrNHGDmcGhmshBBOCny0I5RRti8hp1EK4GcjsnnwyItFhUUMZJ")
      .end(function (result) {
        parent.currentTemp = result.body.main.temp
        parent.currentTempUnit = 'C'
        parent.currentTempText = result.body.weather[0].main
        parent.currentTempCode = result.body.weather[0].icon
      });
    },
    getPhotos(){
      rapid.call('Unsplash', 'searchPhotos', { 
        'accessToken': '3e60947b57ad1a77cfa45576549e5e6367a5af9d299c83542e9ff4d9df7f884c',
        'page': '1',
        'query': 'Tokyo',
        'perPage': '1'
      }).on('success', (payload)=>{
        const results = payload[0].results.map(p => p.urls.full)
        const randomIndex = Math.floor(Math.random() * results.length)
        this.backgroundImage = results[randomIndex] 
      }).on('error', (payload)=>{
        console.log(payload)
      });
  },
    getNews(parent){
      unirest.get(`https://microsoft-azure-bing-news-search-v1.p.rapidapi.com/search?q=${this.city}&mkt=${this.locale}`)
      .header("X-RapidAPI-Key", "jrNHGDmcGhmshBBOCny0I5RRti8hp1EK4GcjsnnwyItFhUUMZJ")
      .end(function (result) {
        parent.news = result.body.value.slice(0,5)
      });
    }
  },
  mounted() {
    // On mount or load, set the location, call the methods
    this.getMyLocation(this)
  }
}
</script>

<style lang="scss">
.weather {
  width: 50%;
}
.title {
  font-weight: 200!important;
  font-size: 4em;
  color: white;
}
.subtitle {
  font-size: 3em;
  color: white;
  font-family:'OstrichSans';
}
.dashboard {
  height:100vh;
  background: no-repeat center center fixed;
  -webkit-background-size: cover;
  -moz-background-size: cover;
  -o-background-size: cover;
  background-size: cover;
  background-color: rgb(0,0,0);
  -webkit-transition: background 1s ease;
  -moz-transition: background 1s ease;
  -o-transition: background 1s ease;
  -ms-transition: background 1s ease;
  transition: background 1s ease;
}
.overlay {
  height:100vh;
  background: url('../assets/overlay.png') repeat 0 0;
  z-index: 9999;
}
.newsitem {
  font-size: 1.5em;
  color: white;
}
</style>
