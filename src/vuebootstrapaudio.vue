<template>
	<div class="card text-center border-0">
		<div class="card-text">
			<button class="btn btn-outline-primary rounded-circle mb-2" @click="playing ? pause() : play()" :disabled="!loaded">
				<i class="fas fa-play fa-fw" v-if="!playing || paused"></i>
				<i class="fas fa-pause fa-fw" v-else></i>
			</button>
			<button class="btn btn-outline-danger rounded-circle mb-2" @click="stop()" :disabled="!loaded">
				<i class="fas fa-stop fa-fw"></i>
			</button>
			<button class="btn btn-outline-primary rounded-circle mb-2" @click="mute()" :disabled="!loaded">
				<i class="fas fa-volume-up fa-fw" v-if="!isMuted"></i>
				<i class="fas fa-volume-off fa-fw" v-else></i>
			</button>
			<button class="btn btn-outline-primary rounded-circle mb-2" @click="loaded ? download() : reload()">
				<i class="fas fa-redo-alt fa-fw" v-if="!loaded"></i>
				<i class="fas fa-file-download fa-fw" v-if="loaded && canDownload"></i>
			</button>
			<div class="progress mb-2" @click="setPosition($event)">
				<div class="progress-bar" height="5" role="progressbar" :aria-valuenow="percentage" aria-valuemin="0" aria-valuemax="100" v-bind:style="{width: percentage + '%'}" :disabled="!loaded"></div>
			</div>
			<p>{{ currentTime }} / {{ duration }}</p>
		</div>
		<audio id="player" ref="player" v-on:ended="ended" v-on:canplay="canPlay">
			<source :src="file">
		</audio>
	</div>
</template>
<script>
const formatTime = second => new Date(second * 1000).toISOString().substr(11, 8);

export default {
	name: 'vue-bootstrap-audio',
	props: {
		file: {
			type: String,
			default: null
		},
		autoPlay: {
			type: Boolean,
			default: false
		},
		ended: {
			type: Function,
			default: () => {},
		},
		canPlay: {
			type: Function,
			default: () => {},
		},
		canDownload: {
			type: Boolean,
			default: true
		}
	},
	computed: {
		duration: function () {
			return this.audio ? formatTime(this.totalDuration) : ''
		},
	},
	data () {
		return {
			firstPlay: true,
			isMuted: false,
			loaded: false,
			playing: false,
			paused: false,
			percentage: 0,
			currentTime: '00:00:00',
			audio: undefined,
			totalDuration: 0,
		}
	},

	methods: {
		setPosition (event) {
			this.percentage = event.offsetX / event.target.clientWidth * 100;
			this.audio.currentTime = parseInt(this.audio.duration / 100 * this.percentage);
		},
		stop () {
			this.paused = this.playing = false
			this.audio.pause()
			this.audio.currentTime = 0
		},
		play () {
			if (this.playing) return
			this.paused = false
			this.audio.play().then(_ => this.playing = true)
		},
		pause () {
			this.paused = !this.paused;
			(this.paused) ? this.audio.pause() : this.audio.play()
		},
		download () {
			this.audio.pause()
			window.open(this.file, 'download')
		},
		mute () {
			this.isMuted = !this.isMuted
			this.audio.muted = this.isMuted
			this.volumeValue = this.isMuted ? 0 : 75
		},
		reload () {
			this.audio.load();
		},
		_handleLoaded: function () {
			if (this.audio.readyState >= 2) {
				if (this.audio.duration === Infinity) {
					// Fix duration for streamed audio source or blob based
					// https://stackoverflow.com/questions/38443084/how-can-i-add-predefined-length-to-audio-recorded-from-mediarecorder-in-chrome/39971175#39971175
					this.audio.currentTime = 1e101;
					this.audio.ontimeupdate = () => {
						this.audio.ontimeupdate = () => {}
						this.audio.currentTime = 0
						this.totalDuration = parseInt(this.audio.duration)
						this.loaded = true;
					}
				} else {
					this.totalDuration = parseInt(this.audio.duration)
					this.loaded = true
				}

				if (this.autoPlay) this.audio.play()

			} else {
				throw new Error('Failed to load sound file')
			}
		},
		_handlePlayingUI: function (e) {
			this.percentage = this.audio.currentTime / this.audio.duration * 100
			this.currentTime = formatTime(this.audio.currentTime)
		},
		_handlePlayPause: function (e) {
			if (e.type === 'play' && this.firstPlay) {
				// in some situations, audio.currentTime is the end one on chrome
				this.audio.currentTime = 0;
				if (this.firstPlay) {
					this.firstPlay = false;
				}
			}
			if (e.type === 'pause' && this.paused === false && this.playing === false) {
				this.currentTime = '00:00:00'
			}
		},
		_handleEnded () {
			this.paused = this.playing = false;
		},
		init: function () {
			this.audio.addEventListener('timeupdate', this._handlePlayingUI);
			this.audio.addEventListener('loadeddata', this._handleLoaded);
			this.audio.addEventListener('pause', this._handlePlayPause);
			this.audio.addEventListener('play', this._handlePlayPause);
			this.audio.addEventListener('ended', this._handleEnded);
			this.audio.load();
		},
	},
	mounted () {
		this.audio = this.$refs.player;
		this.init();
	},
	beforeDestroy () {
		this.audio.removeEventListener('timeupdate', this._handlePlayingUI)
		this.audio.removeEventListener('loadeddata', this._handleLoaded)
		this.audio.removeEventListener('pause', this._handlePlayPause)
		this.audio.removeEventListener('play', this._handlePlayPause)
		this.audio.removeEventListener('ended', this._handleEnded);
	}
}
</script>
