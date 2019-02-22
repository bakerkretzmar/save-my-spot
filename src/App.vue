<template>
    <div id="app" class="container flex pt-24 pb-8">

        <div class="flex flex-col lg:flex-row xl:ml-16 items-start">

            <Logo />

            <div>

                <h1 class="text-8xl font-bold italic mb-10">Save my spot!</h1>

                <p class="text-2xl mb-16">Download your Spotify playlists as spreadsheets. Just in case.</p>

                <div v-if="!token">
                    <a
                        class="connect inline-block p-6 text-xl font-bold bg-green-spotify-dark rounded-full no-underline text-white text-shadow hover:bg-green-spotify-light active:bg-green-spotify-dark"
                        :href="`https://accounts.spotify.com/authorize?client_id=${url.client_id}&response_type=${url.response_type}&redirect_uri=${url.redirect_uri}&scope=${url.scope}`"
                    >
                        Connect to Spotify
                    </a>
                </div>

                <template v-else>
                    <div v-if="loading" class="text-2xl mb-10">
                        Loading...
                    </div>

                    <div class="flex flex-col max-w-md -mx-4">

                        <div
                            v-for="(playlist, index) in playlists"
                            :key="index"
                            class="text-2xl mb-3 py-4 px-6 border-2 border-transparent hover:border-grey-light rounded"
                            :class="{ 'border-grey-light' : playlist.id == selection.id }"
                        >

                            <div class="cursor-pointer" @click="toggleSelect(playlist)">{{ playlist.name }}</div>

                            <transition-expand>
                                <div v-if="playlist.id == selection.id" class="details overflow-hidden max-h-full">

                                    <p v-if="playlist.description" class="text-lg text-grey-light italic mt-4 leading-normal">{{ playlist.description }}</p>

                                    <p class="text-xl text-grey-light my-4 leading-normal">By {{ playlist.owner.display_name }}. {{ playlist.tracks.total }} tracks.</p>

                                    <a class="inline-flex items-center text-xl text-green-spotify-dark font-bold mb-2 mr-8" :href="playlist.external_urls.spotify" target="_blank">
                                        Open
                                        <svg class="inline-flex items-center w-5 h-5 pt-px ml-3" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><path fill="currentColor" d="M18,10.82a1,1,0,0,0-1,1V19a1,1,0,0,1-1,1H5a1,1,0,0,1-1-1V8A1,1,0,0,1,5,7h7.18a1,1,0,0,0,0-2H5A3,3,0,0,0,2,8V19a3,3,0,0,0,3,3H16a3,3,0,0,0,3-3V11.82A1,1,0,0,0,18,10.82Zm3.92-8.2a1,1,0,0,0-.54-.54A1,1,0,0,0,21,2H15a1,1,0,0,0,0,2h3.59L8.29,14.29a1,1,0,0,0,0,1.42,1,1,0,0,0,1.42,0L20,5.41V9a1,1,0,0,0,2,0V3A1,1,0,0,0,21.92,2.62Z"/></svg>
                                    </a>

                                    <a href="#" class="inline-flex items-center text-xl text-green-spotify-dark font-bold mb-2" :class="{ 'opacity-75 cursor-loading' : !download.ready }" @click.prevent="xl">
                                        Download
                                        <svg v-if="!download.ready" class="loading inline-flex items-center w-5 h-5 ml-3" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><path fill="currentColor" d="M11.9,2C9.3,2,6.7,3,4.8,5.1C1,9.1,1.1,15.4,5.1,19.2s10.3,3.7,14.1-0.3C21,17,22,14.6,22,12c0-0.6-0.4-1-1-1s-1,0.4-1,1 c0,4.4-3.6,8.1-8,8.1s-8.1-3.6-8.1-8S7.5,4,11.9,4l0,0c0.6,0,1-0.4,1-1S12.5,2,11.9,2"/></svg>
                                        <svg v-else class="inline-flex items-center w-6 h-6 pt-px ml-3" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><path fill="currentColor" d="M20,8.94a1.31,1.31,0,0,0-.06-.27l0-.09a1.07,1.07,0,0,0-.19-.28h0l-6-6h0a1.07,1.07,0,0,0-.28-.19l-.1,0A1.1,1.1,0,0,0,13.06,2H7A3,3,0,0,0,4,5V19a3,3,0,0,0,3,3H17a3,3,0,0,0,3-3V9S20,9,20,8.94ZM14,5.41,16.59,8H15a1,1,0,0,1-1-1ZM18,19a1,1,0,0,1-1,1H7a1,1,0,0,1-1-1V5A1,1,0,0,1,7,4h5V7a3,3,0,0,0,3,3h3Zm-4.71-4.71-.29.3V12a1,1,0,0,0-2,0v2.59l-.29-.3a1,1,0,0,0-1.42,1.42l2,2a1,1,0,0,0,.33.21.94.94,0,0,0,.76,0,1,1,0,0,0,.33-.21l2-2a1,1,0,0,0-1.42-1.42Z"/></svg>
                                    </a>

                                </div>
                            </transition-expand>

                        </div>

                    </div>

                </template>

            </div>

        </div>

    </div>
</template>

<script>
import Logo from './components/Logo'
import TransitionExpand from './components/TransitionExpand'

import axios from 'axios'
import zipcelx from 'zipcelx'

const PLAYLIST_LIMIT = 50
const TRACK_LIMIT = 100

export default {
    components: {
        Logo,
        TransitionExpand,
    },

    data: () => ({
        download: {
            ready: false,
            data: {},
        },
        loading: false,
        playlists: [],
        selection: {},
        token: '',
        url: {
            client_id: 'ee16c68e3b3d4b56ad7aca14bce05b68',
            response_type: 'token',
            redirect_uri: process.env.NODE_ENV === 'production' ? 'https%3A%2F%2Fsavemyspot.ca' : 'http%3A%2F%2Flocalhost%3A8080',
            scope: ['playlist-read-private', 'playlist-read-collaborative', 'user-library-read', 'user-top-read', 'user-read-email'].join('+'),
        },
        user: {},
    }),

    created() {
        if (window.location.hash && window.location.hash.substring(0, 13) == '#access_token') {
            this.token = window.location.hash.substring(14)
            this.loading = true
            history.pushState(null, null, '/')
        }
    },

    mounted() {
        if (!this.token) return

        axios({
            method: 'get',
            url: 'https://api.spotify.com/v1/me',
            headers: { Authorization: 'Bearer ' + this.token },
        })
        .then(response => {
            this.user = response.data
            this.getAllPlaylists()
        })
    },

    methods: {
        async getAllPlaylists() {
            let items = []
            let keepGoing = true
            let offset = 0

            while (keepGoing) {
                let newItems = await this.getPlaylists(offset)

                items.push(...newItems)
                offset += PLAYLIST_LIMIT

                if (newItems.length < PLAYLIST_LIMIT) {
                    keepGoing = false
                    this.playlists = items
                    this.loading = false
                }
            }
        },

        async getPlaylists(offset) {
            let response = await axios({
                method: 'get',
                url: `https://api.spotify.com/v1/users/${this.user.id}/playlists`,
                headers: { Authorization: 'Bearer ' + this.token },
                params: {
                    limit: PLAYLIST_LIMIT,
                    offset: offset,
                },
            })

            return response.data.items
        },

        async getAllTracks(id) {
            let items = []
            let keepGoing = true
            let offset = 0

            while (keepGoing) {
                let newItems = await this.getTracks(offset, id)

                items.push(...newItems)
                offset += TRACK_LIMIT

                if (newItems.length < TRACK_LIMIT) {
                    keepGoing = false
                    this.download.data = items
                    this.download.ready = true
                }
            }
        },

        async getTracks(offset, id) {
            let response = await axios({
                method: 'get',
                url: `https://api.spotify.com/v1/users/${this.user.id}/playlists/${id}/tracks`,
                headers: { Authorization: 'Bearer ' + this.token },
                params: {
                    limit: TRACK_LIMIT,
                    offset: offset,
                    fields: 'items(track(name,artists,album(name),duration_ms))',
                },
            })

            return response.data.items
        },

        toggleSelect(playlist) {
            if (this.selection == playlist) {
                return this.selection = {}
            }

            this.selection = playlist
            this.getAllTracks(playlist.id)
        },

        xl() {
            if (!this.download.ready) return

            let playlist = this.download.data

            let blank = [{ value: '', type: 'string' }]
            let separator = [
                { value: '----------', type: 'string' },
                { value: '----------', type: 'string' },
                { value: '----------', type: 'string' },
                { value: '----------', type: 'string' },
            ]

            let tracks = playlist.map(item => {
                let artists = item.track.artists.map(artist => {
                    return artist.name
                })

                let seconds = Math.floor(item.track.duration_ms/1000)
                let duration_min = Math.floor(seconds/60)
                let duration_sec = Math.floor(seconds % 60)

                return [
                    { value: item.track.name, type: 'string' },
                    { value: artists.join(', '), type: 'string' },
                    { value: item.track.album.name, type: 'string' },
                    { value: `${duration_min}:${duration_sec.toString().padStart(2, '0')}`, type: 'string' },
                ]
            })

            let rows = [
                [
                    { value: 'Name', type: 'string' },
                    { value: this.selection.name, type: 'string' },
                ],
                [
                    { value: 'Owner', type: 'string' },
                    { value: this.selection.owner.display_name, type: 'string' },
                ],
                [
                    { value: 'Description', type: 'string' },
                    { value: this.selection.description, type: 'string' },
                ],
                [
                    { value: 'URL', type: 'string' },
                    { value: this.selection.external_urls.spotify, type: 'string' },
                ],
                blank, separator, blank,
                [
                    { value: 'Exported at', type: 'string', },
                    { value: `${new Date().toUTCString()}`, type: 'string', },
                ],
                blank, separator, blank,
                [
                    { value: 'Name', type: 'string' },
                    { value: 'Artist', type: 'string' },
                    { value: 'Album', type: 'string' },
                    { value: 'Duration', type: 'string' },
                ],
            ]

            rows.push(...tracks)

            let data = {
                filename: this.selection.name,
                sheet: {
                    data: rows,
                },
            }

            console.log(data)

            zipcelx(data)
        },
    },
}
</script>

<style>
@tailwind preflight;

@font-face {
    font-family: 'Circular';
    src: url('./assets/fonts/CircularStd-Book.woff2') format('woff2'),
         url('./assets/fonts/CircularStd-Book.woff') format('woff'),
         url('./assets/fonts/CircularStd-Book.ttf') format('truetype');
    font-style: normal;
    font-weight: 400;
}

@font-face {
    font-family: 'Circular';
    src: url('./assets/fonts/CircularStd-BookItalic.woff2') format('woff2'),
         url('./assets/fonts/CircularStd-BookItalic.woff') format('woff'),
         url('./assets/fonts/CircularStd-BookItalic.ttf') format('truetype');
    font-style: italic;
    font-weight: 400;
}

@font-face {
    font-family: 'Circular';
    src: url('./assets/fonts/CircularStd-Bold.woff2') format('woff2'),
         url('./assets/fonts/CircularStd-Bold.woff') format('woff'),
         url('./assets/fonts/CircularStd-Bold.ttf') format('truetype');
    font-style: normal;
    font-weight: 700;
}

@font-face {
    font-family: 'Circular';
    src: url('./assets/fonts/CircularStd-BoldItalic.woff2') format('woff2'),
         url('./assets/fonts/CircularStd-BoldItalic.woff') format('woff'),
         url('./assets/fonts/CircularStd-BoldItalic.ttf') format('truetype');
    font-style: italic;
    font-weight: 700;
}

@tailwind components;
@tailwind utilities;

.expand-enter-active,
.expand-leave-active {
    transition-property: opacity, height;
}
.expand-enter,
.expand-leave-to {
    opacity: .6;
}

.loading {
    animation: spin .75s linear infinite;
}
@keyframes spin {
    from { transform: rotate(0deg); }
    to { transform: rotate(360deg); }
}

.connect {
    transform: scale(1);
    transition: all .1s;
}
.connect:hover {
    transform: scale(1.04);
}

/*.details {
    flex: 0;
    transition: flex 1s;
}
.details .show {
    flex: 1;
}*/
</style>
