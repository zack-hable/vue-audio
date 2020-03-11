# vue-bootstrap-audio
<a href="https://www.npmjs.com/package/vuetify-audio"><img src="https://img.shields.io/npm/dt/vuetify-audio.svg" alt="Downloads"></a>
<a href="https://www.npmjs.com/package/vuetify-audio"><img src="https://img.shields.io/npm/v/vuetify-audio.svg" alt="Version"></a>
<a href="https://www.npmjs.com/package/vuetify-audio"><img src="https://img.shields.io/npm/l/vuetify-audio.svg" alt="License"></a>

Vue.js sound audio player based on Bootstrap 4 and Font awesome CSS. Covers audio-tag API and adds more.

### Demo

https://wilsonwu.github.io/dist/index.html#/vuetifyaudio

### Installation

Use npm: ```npm install vue-boostrap-audio --save```

### Usage
1.
Add below code into your ```<script>```:
```js
import VueBootstrapAudio from 'vue-bootstrap-audio';

export default {
    data() {
        return {
            file: 'http://www.noiseaddicts.com/samples_1w72b820/290.mp3',
        };
    },
    components: {
        'vue-bootstrap-audio': VueBootstrapAudio
    },
    // If you want to use ended callback add below code
    methods: {
        audioFinish () {
            console.log('You see this means audio finish.');
        }
    },
}

```

And below code in the ```<template>```:
```js
<vue-bootstrap-audio :file="file" :ended="audioFinish"></vue-bootstrap-audio>
```


### Attributes

 - **file** (String): Set audio file for the audio player
 - **ended** (Function): Set callback function name after audio finish
 - **canPlay** (Function): Set callback function name when audio ready for playing

### Known Issues
1. Audio play pregress bar can't support drag, only support click.

### ToDo

 - ~~Create online demo~~
 - ~~Create npm install~~
 - Add customize styles for component
 - ~~Add event for end audio~~

### License

MIT
