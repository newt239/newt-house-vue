<script setup lang="ts">
import { ref, onMounted } from 'vue';
import axios from "axios";

type trackListProp = {
  "name": string,
  "artists": string[],
  "thumbnail": string,
  "preview": string | null,
  "duration": number,
  "popularity": number,
  "link": string
}[];

const trackList = ref<trackListProp>([]);

onMounted(async () => {
  const res = await axios.get("https://script.google.com/macros/s/AKfycbwnD3fNu5hmfRqpMBc1RCtWqWklYThS4QhQFREKF-EBAeJXMKVmM6BItBWvMTRwrSIm/exec");
  trackList.value = res.data.slice(0, 12);

})

type audioStateProp = {
  "state": boolean,
  "source": string,
  "music": HTMLAudioElement | null
};

const audioState = ref<audioStateProp>({ "state": false, "source": "", "music": null });
const audioButton = (src: string | null) => {
  if (src) {
    const normalPlayAction = () => {
      audioState.value.music = new Audio(src);
      audioState.value.music.play();
      audioState.value.state = true;
      audioState.value.source = src;
    }
    if (!audioState.value.music) {
      normalPlayAction();
    } else {
      audioState.value.music.pause();
      if (!audioState.value.state) {
        if (audioState.value.source != src) {
          normalPlayAction();
        } else {
          audioState.value.music.play();
          audioState.value.state = true;
        }
      } else if (audioState.value.source != src) {
        normalPlayAction();
      } else {
        audioState.value.state = false;
      }
    }
  }
};
</script>

<template>
  <div class="myTopTracks">
    <div v-if="trackList.length == 0">Loading...</div>
    <div class="track" v-for="track in trackList" :alt="track.name">
      <div class="thumbnailWrapper" @click="audioButton(track.preview ? track.preview : null)">
        <img :src="track.thumbnail" class="trackThumbnail" :class="track.preview ? 'trackPreview' : ''" />
        <div class="thumbnailCaption">{{ track.preview ? audioState.state ? audioState.source == track.preview ? "PAUSE"
            : "PLAY" :
            "PLAY" : ""
        }}</div>
      </div>
      <div class="trackInfo">
        <a :href="track.link" target="_blank" class="trackName">{{ track.name }}</a>
        <div class="trackArtists">
          {{ track.artists.join(", ") }}
        </div>
        <div class="subInfo">
          <div>Duration / {{ Math.floor(track.duration / 1000 / 60) + ":" + (String(Math.floor(track.duration / 1000) %
              60).length == 1 ? "0" + String(Math.floor(track.duration / 1000) % 60) : String(Math.floor(track.duration /
                1000) % 60))
          }}
          </div>
          <div>Popularity / {{ track.popularity }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.myTopTracks {
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;

  .track {
    display: flex;
    flex-grow: 1;
    align-items: flex-start;
    width: max(30%, 300px);
    gap: 1rem;
    border-radius: 1rem;
    transition: all 0.5s;

    .thumbnailWrapper {
      position: relative;
      width: min(150px, 50%);
      flex-grow: 1;

      .trackThumbnail {
        width: 100%;
        border-radius: 1rem;
        filter: drop-shadow(2px 4px 6px black);
      }

      .thumbnailCaption {
        font-size: 2rem;
        font-weight: 800;
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translateY(-50%) translateX(-50%);
        mix-blend-mode: difference;
        opacity: 0;
        transition: all 0.5s;
        cursor: pointer;
      }

      &:hover .thumbnailCaption {
        opacity: 1;
      }
    }

    .trackPreview {
      cursor: pointer;
    }

    .trackInfo {
      width: min(150px, 50%);
      flex-grow: 1;
    }

    .trackName {
      font-size: 1.2rem;
      line-height: 1rem;
      font-weight: 800;
      cursor: pointer;
    }

    .subInfo {
      padding-top: .3rem;
      font-size: .7rem;
      line-height: .7rem;
    }
  }

  @include mq(mobile) {
    flex-direction: column;
  }
}
</style>