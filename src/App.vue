<template>
    <div id="app" class="flex flex-col min-h-screen font-sans text-grey-900 antialiased">

        <div class="flex flex-col flex-grow">

            <!-- In case I want to put the playlists over on the right at some point... -->
            <div class="mt-6 md:mt-16 mx-8 md:mx-20">

                <h1 class="text-6xl md:text-hero font-black leading-none italic mb-10">Save my Spot!</h1>

                <p class="text-2xl mb-10">Download your Spotify playlists as spreadsheets. Just in case.</p>

                <a
                    v-if="!token"
                    class="connect-button inline-block px-8 py-6 text-xl font-bold rounded-full shadow-xl hover:shadow-2xl"
                    :href="`https://accounts.spotify.com/authorize?client_id=${url.client_id}&response_type=${url.response_type}&redirect_uri=${url.redirect_uri}&scope=${url.scope}`"
                >
                    Connect to Spotify
                </a>

                <template v-else>

                    <div v-if="loading" class="text-lg font-bold">
                        Loading...
                    </div>

                    <div v-else class="flex flex-col max-w-lg -mx-4">

                        <div
                            v-for="(playlist, index) in playlists"
                            :key="index"
                            class="text-2xl mb-2 py-4 px-6 border-3 border-transparent hover:border-gray-900 rounded-lg"
                            :class="{ 'border-gray-900' : playlist.id == selection.id }"
                        >

                            <div class="cursor-pointer" @click="toggleSelect(playlist)">{{ playlist.name }}</div>

                            <transition-expand>

                                <div v-if="playlist.id == selection.id" class="details overflow-hidden max-h-full">

                                    <p v-if="playlist.description" class="text-lg text-gray-600 italic mt-4">{{ playlist.description }}</p>

                                    <p class="text-lg text-gray-600 my-4 leading-normal">By {{ playlist.owner.display_name }}. {{ playlist.tracks.total }} tracks.</p>

                                    <a
                                        class="inline-flex items-center text-xl text-green-spotify font-bold mr-8"
                                        :href="playlist.external_urls.spotify"
                                        target="_blank"
                                    >
                                        Open
                                        <svg class="inline-flex items-center w-5 h-5 ml-2" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><path fill="currentColor" d="M18,10.82a1,1,0,0,0-1,1V19a1,1,0,0,1-1,1H5a1,1,0,0,1-1-1V8A1,1,0,0,1,5,7h7.18a1,1,0,0,0,0-2H5A3,3,0,0,0,2,8V19a3,3,0,0,0,3,3H16a3,3,0,0,0,3-3V11.82A1,1,0,0,0,18,10.82Zm3.92-8.2a1,1,0,0,0-.54-.54A1,1,0,0,0,21,2H15a1,1,0,0,0,0,2h3.59L8.29,14.29a1,1,0,0,0,0,1.42,1,1,0,0,0,1.42,0L20,5.41V9a1,1,0,0,0,2,0V3A1,1,0,0,0,21.92,2.62Z"/></svg>
                                    </a>

                                    <p
                                        class="inline-flex items-center cursor-pointer text-xl text-green-spotify font-bold"
                                        :class="{ 'opacity-75 cursor-loading' : !download.ready }"
                                        @click.prevent="xl"
                                    >
                                        Download
                                        <svg v-if="!download.ready" class="loading inline-flex items-center w-5 h-5 ml-3" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><path fill="currentColor" d="M11.9,2C9.3,2,6.7,3,4.8,5.1C1,9.1,1.1,15.4,5.1,19.2s10.3,3.7,14.1-0.3C21,17,22,14.6,22,12c0-0.6-0.4-1-1-1s-1,0.4-1,1 c0,4.4-3.6,8.1-8,8.1s-8.1-3.6-8.1-8S7.5,4,11.9,4l0,0c0.6,0,1-0.4,1-1S12.5,2,11.9,2"/></svg>
                                        <svg v-else class="inline-flex items-center w-6 h-6 pb-px ml-2" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><path fill="currentColor" d="M20,8.94a1.31,1.31,0,0,0-.06-.27l0-.09a1.07,1.07,0,0,0-.19-.28h0l-6-6h0a1.07,1.07,0,0,0-.28-.19l-.1,0A1.1,1.1,0,0,0,13.06,2H7A3,3,0,0,0,4,5V19a3,3,0,0,0,3,3H17a3,3,0,0,0,3-3V9S20,9,20,8.94ZM14,5.41,16.59,8H15a1,1,0,0,1-1-1ZM18,19a1,1,0,0,1-1,1H7a1,1,0,0,1-1-1V5A1,1,0,0,1,7,4h5V7a3,3,0,0,0,3,3h3Zm-4.71-4.71-.29.3V12a1,1,0,0,0-2,0v2.59l-.29-.3a1,1,0,0,0-1.42,1.42l2,2a1,1,0,0,0,.33.21.94.94,0,0,0,.76,0,1,1,0,0,0,.33-.21l2-2a1,1,0,0,0-1.42-1.42Z"/></svg>
                                    </p>

                                </div>

                            </transition-expand>

                        </div>

                    </div>

                </template>

            </div>

        </div>

        <div class="flex items-center justify-end w-full mb-2 text-xs text-white pr-3">

            <p class="pr-2">Save my Spot doesn’t store any of your data.</p>

            <a class="text-gray-300" href="https://github.com/bakerkretzmar/save-my-spot" target="_blank">
                <svg class="fill-current w-4 h-4" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 496 512"><path d="M165.9 397.4c0 2-2.3 3.6-5.2 3.6-3.3.3-5.6-1.3-5.6-3.6 0-2 2.3-3.6 5.2-3.6 3-.3 5.6 1.3 5.6 3.6zm-31.1-4.5c-.7 2 1.3 4.3 4.3 4.9 2.6 1 5.6 0 6.2-2s-1.3-4.3-4.3-5.2c-2.6-.7-5.5.3-6.2 2.3zm44.2-1.7c-2.9.7-4.9 2.6-4.6 4.9.3 2 2.9 3.3 5.9 2.6 2.9-.7 4.9-2.6 4.6-4.6-.3-1.9-3-3.2-5.9-2.9zM244.8 8C106.1 8 0 113.3 0 252c0 110.9 69.8 205.8 169.5 239.2 12.8 2.3 17.3-5.6 17.3-12.1 0-6.2-.3-40.4-.3-61.4 0 0-70 15-84.7-29.8 0 0-11.4-29.1-27.8-36.6 0 0-22.9-15.7 1.6-15.4 0 0 24.9 2 38.6 25.8 21.9 38.6 58.6 27.5 72.9 20.9 2.3-16 8.8-27.1 16-33.7-55.9-6.2-112.3-14.3-112.3-110.5 0-27.5 7.6-41.3 23.6-58.9-2.6-6.5-11.1-33.3 2.6-67.9 20.9-6.5 69 27 69 27 20-5.6 41.5-8.5 62.8-8.5s42.8 2.9 62.8 8.5c0 0 48.1-33.6 69-27 13.7 34.7 5.2 61.4 2.6 67.9 16 17.7 25.8 31.5 25.8 58.9 0 96.5-58.9 104.2-114.8 110.5 9.2 7.9 17 22.9 17 46.4 0 33.7-.3 75.4-.3 83.6 0 6.5 4.6 14.4 17.3 12.1C428.2 457.8 496 362.9 496 252 496 113.3 383.5 8 244.8 8zM97.2 352.9c-1.3 1-1 3.3.7 5.2 1.6 1.6 3.9 2.3 5.2 1 1.3-1 1-3.3-.7-5.2-1.6-1.6-3.9-2.3-5.2-1zm-10.8-8.1c-.7 1.3.3 2.9 2.3 3.9 1.6 1 3.6.7 4.3-.7.7-1.3-.3-2.9-2.3-3.9-2-.6-3.6-.3-4.3.7zm32.4 35.6c-1.6 1.3-1 4.3 1.3 6.2 2.3 2.3 5.2 2.6 6.5 1 1.3-1.3.7-4.3-1.3-6.2-2.2-2.3-5.2-2.6-6.5-1zm-11.4-14.7c-1.6 1-1.6 3.6 0 5.9 1.6 2.3 4.3 3.3 5.6 2.3 1.6-1.3 1.6-3.9 0-6.2-1.4-2.3-4-3.3-5.6-2z"></path></svg>
            </a>

        </div>

    </div>
</template>

<script>
import TransitionExpand from './TransitionExpand'

import axios from 'axios'
import zipcelx from 'zipcelx'

const PLAYLIST_LIMIT = 50
const TRACK_LIMIT = 100

export default {
    components: { TransitionExpand },

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
            scope: ['playlist-read-private', 'playlist-read-collaborative', 'user-library-read'].join('+'),
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

            // console.log(data)

            zipcelx(data)
        },
    },
}
</script>

<style>
@tailwind base;

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
    src: url('./assets/fonts/CircularStd-Bold.woff2') format('woff2'),
         url('./assets/fonts/CircularStd-Bold.woff') format('woff'),
         url('./assets/fonts/CircularStd-Bold.ttf') format('truetype');
    font-style: normal;
    font-weight: 700;
}

/*@font-face {
    font-family: 'Circular';
    src: url('./assets/fonts/CircularStd-Black.woff2') format('woff2'),
         url('./assets/fonts/CircularStd-Black.woff') format('woff'),
         url('./assets/fonts/CircularStd-Black.ttf') format('truetype');
    font-style: normal;
    font-weight: 900;
}*/

@font-face {
    font-family: 'Circular';
    src: url('./assets/fonts/CircularStd-BlackItalic.woff2') format('woff2'),
         url('./assets/fonts/CircularStd-BlackItalic.woff') format('woff'),
         url('./assets/fonts/CircularStd-BlackItalic.ttf') format('truetype');
    font-style: italic;
    font-weight: 900;
}

body {
    background: url(./assets/img/tidal.png);
    background-size: cover;
}

@tailwind components;

@tailwind utilities;

.loading {
    animation: spin .75s linear infinite;
}

@keyframes spin {
    from { transform: rotate(0deg); }
    to { transform: rotate(360deg); }
}

.connect-button {
    background: radial-gradient(ellipse farthest-corner at 14% 14%, rgba(246, 245, 109, 0.55) 50%, rgba(227, 171, 58, 0.55) 75%, rgba(218, 23, 201, 0.55));
    transform: scale(1.00);
    transition: all .25s ease;
}

.connect-button:hover {
    transform: scale(1.02);
}
</style>
