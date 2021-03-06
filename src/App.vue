<style>
@import "./assets/styles/custom.css";
</style>

<template>
  <section>
    <h2>Dictionary</h2>
    <span class="input-custom">
      <input
        class="p-col-2"
        v-model="word"
        placeholder="Input word"
        v-on:keyup.enter="getEntry"
      />
    </span>
    <Button v-on:click="getEntry" class="pi pi-search p-button-icon">
      Search</Button
    >

    <ul v-if="entries && entries.length">
      <div v-for="entry in entries" :key="entry.word">
        <div class="general">
          <span class="text-left">{{ entry.word }} </span>
          <span v-for="phonetic in entry.phonetics" :key="phonetic.text">
            <span class="phonetic">
              <span class="text">{{ phonetic.text }}</span>
              <Button @click="doCopy(phonetic.text)" icon="pi pi-copy" class="p-button-sm margin-right"></Button>
              <label class="padding-right">
                <Button v-if="phonetic.audio.length"
                  v-on:click="playSound(phonetic.audio)"
                  icon="pi pi-volume-up"
                  class="p-button-sm padding-right"
                >
                </Button>
                <Button v-if="!phonetic.audio.length"
                        v-on:click="playSound('https://ssl.gstatic.com/dictionary/static/sounds/oxford/' + entry.word + '--_us_1.mp3')"
                        icon="pi pi-volume-up"
                        class="p-button-sm padding-right"
                >
                </Button>
              </label>
              <label>
                <Button v-if="phonetic.audio.length"
                        v-on:click="downloadMp3(phonetic.audio)"
                        icon="pi pi-download"
                        class="p-button-sm padding-left"
                />
                <Button v-if="!phonetic.audio.length"
                        v-on:click="downloadMp3('https://ssl.gstatic.com/dictionary/static/sounds/oxford/' + entry.word + '--_us_1.mp3')"
                        icon="pi pi-download"
                        class="p-button-sm padding-left"
                />
              </label>
            </span>
          </span>
        </div>

        <div v-for="meaning in entry.meanings" :key="meaning.partOfSpeech">
          <div class="">
            <ul>
              <div>
                <li>
                  <p class="text-left">
                    Part of Speech: {{ meaning.partOfSpeech }}
                  </p>
                  <ul
                    v-for="definition in meaning.definitions"
                    :key="definition.definition"
                  >
                    <span class="shadow">
                      <p class="text-left">
                        Definition: {{ definition.definition }}
                      </p>
                      <p class="text-left">Example: {{ definition.example }}</p>
                      <p class="text-left">
                        Synonyms: {{ definition.synonyms }}
                      </p>
                    </span>
                  </ul>
                </li>
              </div>
            </ul>
          </div>
        </div>
      </div>
    </ul>
  </section>
</template>

<script>
import axios from "axios";
import { copyText } from 'vue3-clipboard'

const HOST = "https://dictionary-webapp.herokuapp.com/api/dictionary";

export default {
  data() {
    return {
      entries: [],
      word: [],
    };
  },

  methods: {
    getEntry: function() {
      let url = HOST + "?word=" + this.word;
      axios
        .get(url)
        .then((response) => {
          this.entries = response.data;
        })
        .catch((e) => {
          alert("Can't found the word: " + this.word);
          console.log("Error: " + e);
        });
    },
    downloadMp3(url) {
      var newUrl = HOST + "/audio";
      axios
        .get(newUrl, {
          params: {
            audioUrl: url,
          },
        })
        .then((response) => {
          var fileURL = response.request.responseURL;
          var fileLink = document.createElement("a");

          fileLink.href = fileURL;

          fileLink.click();
        })
        .catch((e) => {
          alert("Can't download the audio file: " + this.word);
          console.log("Error: " + e);
        });
    },

    playSound(url) {
      var audio = new Audio(url);
      audio.play();
    },
    doCopy(phoneticText) {
        copyText(phoneticText, undefined, (error) => {
          if (error) {
            console.log(error)
          }
        })
      }
  },
};
</script>
