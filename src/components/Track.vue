<template>
    <div class="tracks">
        <div class="controls">
            <md-button class="md-icon-button" @click="skipToPrevious()"><md-icon>skip_previous</md-icon></md-button>
            <md-button class="md-icon-button" @click="play()" v-if="!started"><md-icon>play_arrow</md-icon></md-button>
            <md-button class="md-icon-button" @click="resume()" v-if="started && !isPlaying"><md-icon>play_arrow</md-icon></md-button>
            <md-button class="md-icon-button" @click="pause()" v-if="started && isPlaying"><md-icon>pause</md-icon></md-button>
            <md-button class="md-icon-button" @click="skipToNext()"><md-icon>skip_next</md-icon></md-button>
        </div>
        <md-card class="track">
            <md-card-media-cover>
                <md-card-media md-ratio="1:1">
                    <img :src="prevCover">
                </md-card-media>
            </md-card-media-cover>
        </md-card>
        <md-card class="track current">
            <md-card-media-cover>
                <md-card-media md-ratio="1:1">
                    <img :src="currentCover">
                </md-card-media>

                <md-card-area>
                    <md-card-header>
                        <span class="md-title">{{ this.current.name }}</span>
                        <span class="md-subhead">{{ this.current.artists[0].name }}</span>
                    </md-card-header>
                </md-card-area>
            </md-card-media-cover>
        </md-card>
        <md-card class="track">
            <md-card-media-cover>
                <md-card-media md-ratio="1:1">
                    <img :src="nextCover">
                </md-card-media>
            </md-card-media-cover>
        </md-card>
    </div>
</template>

<script>
import { mapState, mapGetters } from 'vuex'
import { PauseCircleIcon, SkipForwardIcon } from 'vue-feather-icons'

export default {
    name: 'Track',
    data () {
        return {
            isPlaying: false,
            started: false
        }
    },
    components: {
        PauseCircleIcon,
        SkipForwardIcon
    },
    computed: {
        ...mapState(['playing', 'tracks']),
        ...mapGetters(['int']),
        prev: function () {
            return this.tracks[this.int][this.playing - 1]
        },
        current: function () {
            return this.tracks[this.int][this.playing]
        },
        next: function () {
            return this.tracks[this.int][this.playing + 1]
        },
        prevCover: function() {
            if (this.prev) return this.prev.album.images[0].url
            else return "http://geantav.hol.es/wogypyb.png"
        },
        currentCover: function() {
            return this.current.album.images[0].url
        },
        nextCover: function() {
            return this.next.album.images[0].url
        }
    },
    methods: {
        async play () {
            console.log('hi')
            const devices = await this.spotify.getMyDevices()
            const id = devices.body.devices[0].id
            await this.spotify.play({ device_id: id, uris: this.tracks[this.int].map((elem) => elem.uri) })
            this.started = true
            this.isPlaying = true
        },
        async skipToNext () {
            await this.spotify.skipToNext()
            this.$store.commit('next')
        },
        async skipToPrevious () {
            await this.spotify.skipToPrevious()
            this.$store.commit('previous')
        },
        async resume () {
            await this.spotify.play()
            this.isPlaying = true
        },
        async pause () {
            await this.spotify.pause()
            this.isPlaying = false
        }
    },
}
</script>

<style scoped>
.controls {
    position: absolute;
    left: 50%;
    transform: translateX(-50%) translateY(-150%);
}

.tracks {
    margin-top: 50px;
}

.track {
    height: 250px;
    width: 250px;
    display: inline-block;
    vertical-align: middle;
}

.current {
    height: 400px;
    width: 400px;
    margin-left: -100px;
    margin-right: -100px;
    z-index: 1000;
}

.current img {
    transition: filter 0.2s;
}

.left {
    left: 30%;
}

.right {
    left: 70%;
}

.track img {
    filter: brightness(70%);
}
</style>
