<template>
  <div>

    <!-- JUST FOR TESTING --> <button @click="getTracks()">REFRESH</button>
    
    <div class="wrapper container-fluid">
      <div class="content d-flex">
        <div v-if="!tracks[0]" class="no-songs container">No favorite songs</div>
        <div v-if="tracks[0]" class="song-list container">
          <div class="song d-flex row" v-for="(track, index) in tracks" :key="index">
            <div
              class="song-index d-none d-md-flex col-md-1 align-items-center justify-content-end"
            >{{index + 1}}.&#8194;</div>
            <div class="song-cover d-flex align-items-center justify-content-center">
              <img
                class="b-radius"
                :src="_imageEncode(track.picture.data)"
                alt="IMG"
                height="60px"
                width="60px"
              />
            </div>
            <div class="song-info col col-md row">
              <div class="info-title col-12 col-md-6 col-xl-4 d-flex align-items-center">
                <span class="text-truncate">
                  <b>{{track.title}}</b>
                </span>
              </div>
              <!-- <div class="info-split d-none d-md-flex align-items-center">&#8212;</div> -->
              <div class="info-artist col-12 col-md-6 col-xl-4 d-flex align-items-center">
                <span class="text-truncate">{{track.artist}}</span>
              </div>
              <div class="info-albom d-none col-xl-4 d-xl-flex align-items-center">{{track.album}}</div>
            </div>
            <div
              class="song-duration col-1 d-flex align-items-center justify-content-center"
            >{{parseInt(track.duration / 60) + ':' + PrettyTimeFormat(parseInt(track.duration % 60))}}</div>
            <div
              class="song-controls col-1 d-flex align-items-center justify-content-center"
              @click="hideTrack($event, track.fileName)"
            >X</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "FavoritesList",
  data() {
    return {
      url: "http://localhost:10204/",
      accessToken: "asdas",
      refreshToken:
        "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJsb2dpbiI6ImFzZCIsImlhdCI6MTU3MjM1ODU2MH0.Y7w-hVZyDxul-t26JHT6qgTDuFl2M-8PsP8kv1kM_zI",
      tracks: []
    };
  },
  computed: {},
  methods: {
    hideTrack(e, filename) {
      e.target.parentElement.style.visibility = "hidden";
      this.delTrack(filename, e.target.parentElement);
    },
    getTracks() {
      this.$http
        .post(
          this.url + "favorites/",
          {},
          {
            headers: {
              Authorization: "Bearer " + this.accessToken
            }
          }
        )
        .then(res => {
          this.tracks = res.body;
        })
        .catch(async res => {
          console.log("REFRESHING..");
          if (res.status == 403) {
            await this.refreshTokens({ func: this.getTracks });
          }
        });
    },
    PrettyTimeFormat(num) {
      return num < 10 ? `0${num}` : num;
    },
    _imageEncode(arrayBuffer) {
      if (!arrayBuffer) return "https://i.imgur.com/1Y5Uhk6.png";
      let u8 = new Uint8Array(arrayBuffer);
      let b64encoded = btoa(
        [].reduce.call(
          new Uint8Array(arrayBuffer),
          (p, c) => {
            return p + String.fromCharCode(c);
          },
          ""
        )
      );
      let mimetype = "image/jpeg";
      return "data:" + mimetype + ";base64," + b64encoded;
    },
    delTrack(fileName, target) {
      this.$http
        .post(
          this.url + "favorites/delete",
          { song: fileName },
          {
            headers: {
              "Content-Type": "application/json",
              Authorization: "Bearer " + this.accessToken
            }
          }
        )
        .then(res => {
          this.tracks = res.body;
          target.style.visibility = "visible";
        })
        .catch(async res => {
          console.log("REFRESHING..");
          if (res.status == 403) {
            await this.refreshTokens({
              func: this.delTrack,
              args: [fileName, target]
            });
          }
        });
    },
    refreshTokens(callbeck) {
      this.$http
        .post(
          this.url + "auth/refresh",
          { refreshToken: this.refreshToken },
          {
            headers: {
              "Content-Type": "application/json"
            }
          }
        )
        .then(res => {
          if (res.body.refreshToken) {
            this.refreshToken = res.body.refreshToken;
            this.accessToken = res.body.accessToken;
            if (callbeck.args) callbeck.func(...callbeck.args);
            else callbeck.func();
          }
        });
    }
  },
  created() {
    this.getTracks();
  }
};
</script>

<style lang="scss" >
* {
  // outline: 1px solid black;
}

html {
  font-size: 0.9rem;

  @media screen and (min-width: 768px) {
    font-size: 1rem;
  }
}

.content {
  margin: 0 auto;
}

.song {
  margin-bottom: 8px;
  // border-bottom: 1px solid gray;

  & > div {
    padding: 0;
    margin: 0;
  }

  .song-info {
    min-width: 100px;
  }

  .song-cover {
    min-width: 60px;
    min-height: 60px;
    max-width: 60px;
    max-height: 60px;

    .b-radius {
      border-radius: 8px;
    }
  }

  .song-duration {
    min-width: 36px;
  }
}
</style>