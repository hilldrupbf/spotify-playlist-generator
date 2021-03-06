<template>
  <base-card>
    <form @submit.prevent="submitForm">
      <h1>Generate a Playlist</h1>
      <div class="form-row">
        <div class="form-control">
          <div>
            <label for="subreddit">Subreddit</label>
            <input
              name="redditType"
              type="radio"
              id="subreddit"
              value="subreddit"
              v-model="redditType"
            />
          </div>
          <div>
            <label for="multireddit">Multireddit</label>
            <input
              name="redditType"
              type="radio"
              id="multireddit"
              value="multireddit"
              v-model="redditType"
            />
          </div>
        </div>
      </div>
      <transition name="fade" mode="out-in">
        <div class="form-row" v-if="redditType === 'subreddit'" key="sub">
          <div class="form-control">
            <label for="subreddit">Subreddit(s)</label>
            <input
              :title="subredditHelp"
              type="text"
              id="subreddit"
              required
              v-model="subreddit"
            />
          </div>
        </div>
        <div class="form-row" v-else key="multi">
          <div class="form-control">
            <label for="user">Username</label>
            <input type="text" id="user" required v-model="username" />
          </div>
          <div class="form-control">
            <label for="multi">Multireddit</label>
            <input type="text" id="multi" required v-model="multireddit" />
          </div>
        </div>
      </transition>
      <div class="form-row">
        <div class="form-control">
          <label for="num-songs">Max # of songs</label>
          <input
            type="number"
            id="num-songs"
            min="1"
            max="300"
            v-model="numSongs"
          />
        </div>
        <div class="form-control">
          <label for="upvotes">Minimum Upvotes</label>
          <input type="number" id="upvotes" min="1" v-model="minUpvotes" />
        </div>
      </div>
      <div class="form-control">
        <h3>Sort By</h3>
        <div>
          <label for="hot">Hot</label>
          <input
            name="sort"
            type="radio"
            id="hot"
            value="hot"
            @change="toggleTop"
            v-model="sort"
          />
        </div>
        <div>
          <label for="top">Top</label>
          <input
            name="sort"
            type="radio"
            id="top"
            value="top"
            @change="toggleTop"
            v-model="sort"
          />
        </div>
        <div>
          <label for="new">New</label>
          <input
            name="sort"
            type="radio"
            id="new"
            value="new"
            @change="toggleTop"
            v-model="sort"
          />
        </div>
        <div>
          <label for="rising">Rising</label>
          <input
            name="sort"
            type="radio"
            id="rising"
            value="rising"
            @change="toggleTop"
            v-model="sort"
          />
        </div>
      </div>
      <div class="form-control" v-if="showTop">
        <h3>Top Posts From...</h3>

        <div>
          <label for="hour">Hour</label>
          <input
            name="time"
            type="radio"
            id="hour"
            value="hour"
            v-model="top"
          />
        </div>
        <div>
          <label for="day">Day</label>
          <input name="time" type="radio" id="day" value="day" v-model="top" />
        </div>
        <div>
          <label for="week">Week</label>
          <input
            name="time"
            type="radio"
            id="week"
            value="week"
            v-model="top"
          />
        </div>
        <div>
          <label for="month">Month</label>
          <input
            name="time"
            type="radio"
            id="month"
            value="month"
            v-model="top"
          />
        </div>
        <div>
          <label for="year">Year</label>
          <input
            name="time"
            type="radio"
            id="year"
            value="year"
            v-model="top"
          />
        </div>
        <div>
          <label for="all">All</label>
          <input name="time" type="radio" id="all" value="all" v-model="top" />
        </div>
      </div>
      <div class="form-control checkboxes">
        <div>
          <label for="albums">Include Albums</label>
          <input type="checkbox" name="albums" id="albums" v-model="albums" />
        </div>
        <div>
          <label for="playlists">Include Playlists</label>
          <input
            type="checkbox"
            name="playlists"
            id="playlists"
            v-model="playlists"
          />
        </div>
      </div>
      <div v-if="loading">
        <base-spinner></base-spinner>
      </div>
      <base-button v-else mode="flat-green">Create Playlist</base-button>
    </form>
  </base-card>
</template>

<script>
export default {
  emits: ['submission'],
  props: ['loading'],
  data() {
    // Consider moving to vuex
    return {
      username: '',
      multireddit: '',
      subreddit: '',
      redditType: 'subreddit',
      numSongs: 50,
      minUpvotes: 1,
      sort: 'hot',
      top: 'all',
      albums: false,
      playlists: false,
      showTop: false,
      subredditHelp: 'Enter a comma separated list of subreddits',
    };
  },
  methods: {
    submitForm() {
      // Consider moving to vuex
      let values = {
        username: this.username,
        multireddit: this.multireddit,
        subreddit: this.subreddit,
        redditType: this.redditType,
        numSongs: this.numSongs,
        minUpvotes: this.minUpvotes,
        sort: this.sort,
        top: this.top,
        albums: this.albums,
        playlists: this.playlists,
      };

      this.$store.dispatch('updateNumTracks', this.numSongs);

      this.$emit('submission', values);
    },
    // Shows/hides sort options if 'Top' isn't selected
    toggleTop() {
      this.sort === 'top' ? (this.showTop = true) : (this.showTop = false);
    },
  },
};
</script>

<style scoped>
form {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-width: 25rem;
}
.form-control {
  margin: 0.5rem 0;
}
.form-control div {
  display: inline-block;
  margin: 0 1rem;
}

.form-row {
  display: flex;
}
.form-row input {
  width: 10rem;
  margin: 0 0.5rem;
}

label {
  font-weight: bold;
  display: block;
  margin-bottom: 0.5rem;
}

input {
  font: inherit;
}
input:focus:required:invalid {
  box-shadow: 0 0px 1px 1px red;
}
input:focus:required:valid,
input:focus:required:valid,
input:required:invalid {
  box-shadow: none;
}

input[type='radio'] {
  display: inline;
  width: auto;
  border: none;
}

input[type='radio']:focus {
  outline: #1ed760 solid 1px;
}

h3 {
  margin: 0.5rem 0;
  font-size: 1rem;
}

.invalid label {
  color: red;
}

.invalid input,
.invalid textarea {
  border: 1px solid red;
}

.checkboxes {
  padding-bottom: 0.5rem;
  border-bottom: 1px solid #555;
  margin-bottom: 1rem;
  width: 100%;
}

/* Animation */
.fade-enter-active,
.fade-leave-active {
  transition: all 0.2s ease;
}
.fade-enter-from {
  opacity: 0;
}
.fade-leave-to {
  opacity: 0;
}
</style>
