<template>
  <div class="content">
    <base-card>
      <div class="container">
        <base-button
          v-if="access_token === '' || access_token === 'access_denied'"
          class="flat-green"
          @click="authorize"
          >Authorize To Export</base-button
        >
        <div v-else>
          <form @submit.prevent="exportPlaylist">
            <label for="name">Playlist Name: </label>
            <input type="text" required v-model="playlistName" />
            <base-button class="flat-green">Export Playlist</base-button>
          </form>
        </div>
      </div>
      <div class="playlist">
        <div class="table-header">
          <!-- Number -->
          <div class="column-header-item number">#</div>
          <!-- Title -->
          <div class="column-header-item title">Title</div>
          <!-- Artist -->
          <div class="column-header-item">Artist</div>
          <!-- Album -->
          <div class="column-header-item">Album</div>
          <!-- Duration -->
          <div class="column-header-item duration">Duration</div>
          <!-- Remove -->
          <div class="column-header-item remove">Remove</div>
        </div>
        <div class="table-row" v-for="(track, index) in tracks" :key="index">
          <!-- Number -->
          <div class="column-row-item number">{{ index + 1 }}.</div>
          <!-- Title -->
          <div :title="track.name" class="column-row-item title">
            {{ track.name }}
          </div>
          <!-- Artist -->
          <div :title="getArtists(track.artists)" class="column-row-item">
            {{ getArtists(track.artists) }}
          </div>
          <!-- Album -->
          <div :title="track.album" class="column-row-item">
            {{ track.album }}
          </div>
          <!-- Duration -->
          <div :title="track.duration" class="column-row-item duration">
            {{ track.duration }}
          </div>
          <!-- Remove -->
          <button class="column-row-item remove" @click="removeTrack(index)">
            X
          </button>
        </div>
      </div>
    </base-card>
  </div>
</template>

<script>
import BaseCard from '../components/ui/BaseCard.vue';
import BaseButton from '../components/ui/BaseButton.vue';
import * as spotify from '../api/spotify.js';

export default {
  components: {
    BaseCard,
    BaseButton,
  },
  data() {
    return {
      tracks: null,
      access_token: '',
      username: '',
      playlistName: '',
    };
  },
  methods: {
    getArtists(artists) {
      let artistString = '';
      artists.forEach((artist, index) => {
        artistString += artist.name;
        if (index < artists.length - 1) {
          artistString += ', ';
        }
      });
      return artistString;
    },
    removeTrack(trackIndex) {
      this.tracks.splice(trackIndex, 1);
    },
    authorize() {
      spotify.authorizeUser();
    },
    async exportPlaylist() {
      const playlistResponse = await spotify.createPlaylist(
        this.access_token,
        this.username,
        this.playlistName
      );

      const uris = this.$store.getters['getTracks'].map(track => track.uri);

      try {
        await spotify.addTracksToPlaylist(
          this.access_token,
          playlistResponse.data.id,
          uris
        );
        this.$router.push('/form');
      } catch (e) {
        console.log(e);
      }
    },
  },
  mounted() {
    this.tracks = this.$store.getters['getTracks'];

    if (this.$route.hash.includes('access_token')) {
      this.access_token = this.$route.hash.split('&')[0].split('=')[1];
      spotify.getProfileInfo(this.access_token).then(res => {
        this.username = res.data.id;
      });
    } else if (this.$route.hash.includes('access_denied')) {
      this.access_token = 'denied';
    }
  },
};
</script>

<style scoped>
label {
  color: #1ed760;
  font-weight: bold;
}
input {
  margin-right: 2rem;
}
input:focus:required:invalid {
  box-shadow: 0 0px 1px 1px red;
}
input:focus:required:valid,
input:focus:required:valid,
input:required:invalid {
  box-shadow: none;
}
.content {
  min-height: 93vh;
  margin-top: 2rem;
  position: relative;
  margin-bottom: 3rem;
}
.container {
  display: flex;
  justify-content: center;
}
.playlist {
  display: flex;
  flex-direction: column;
  background: #1d1f20;
  color: white;
  margin-top: 2rem;
}

.table-header,
.table-row {
  display: flex;
  min-height: 40px;
  padding: 5px;
  color: #fff;
  font-size: 15px;
  border-bottom: 1px solid #333;
}
.table-header {
  text-transform: uppercase;
  color: #959595;
  font-size: 13px;
}
.column-header-item,
.column-row-item {
  flex: 0 1 250px;
  align-self: center;
  margin: 0 10px;
}
.column-row-item {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.number {
  flex: 0 0 30px;
}
.title {
  flex: 0 0 250px;
}
.duration {
  flex: 0 0 80px;
  text-align: center;
}
.remove {
  flex: 0 0 60px;
}
.column-row-item.remove {
  border: none;
  background: transparent;
  color: red;
  font-size: 1rem;
  cursor: pointer;
  border-radius: 5px;
}
.column-row-item.remove:hover {
  background: red;
  color: black;
}
</style>
