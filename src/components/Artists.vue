<!-- Boijy made this and put his best here (Used some stuff like <style scoped> and a bit more from SongList.)-->

<template>
    <div class="artist-list">
        <div class="artist-row" v-for="(row, index) in rows" :key="index">
            <div class="artist-card" v-for="artist in row" :key="artist.id">
                <div class="artist-content">
                    <div class="artist-image-container">
                        <img v-if="artist.images.length > 0" :src="artist.images[0].url" alt="Artist Image" class="artist-image" />
                    </div>
                    <div class="artist-info">
                        <h3 style="font-weight: bold; margin-bottom: 5px; color: #b1b1b1">{{ artist.name }}</h3>
                        <p style="font-size: 0.9em; color: #b1b1b1" v-if="artist.followers">Followers: {{ artist.followers.total }}</p>
                    </div>
                </div>
            </div>
        </div>
        <div v-if="isLoading" class="loading-container">
            <div class="loading-spinner"></div>
        </div>
    </div>
</template>

<script>
    import axios from 'axios';

    export default {
        name: 'Artists',
        data() {
            return {
                token: null,
                artistsData: [],
                isLoading: true,
                rows: []
            };
        },
        mounted() {
            this.initializeSpotify();
        },
        methods: {
            initializeSpotify() {
                const storedToken = localStorage.getItem('spotifyToken');
                const storedTokenCreationTime = localStorage.getItem('tokenCreationTime');

                if (!storedToken || !storedTokenCreationTime) {
                    this.getAccessToken();
                } else {
                    this.token = storedToken;
                    const currentTime = Date.now() / 1000;
                    const tokenCreationTime = parseFloat(storedTokenCreationTime);

                    if (currentTime - tokenCreationTime > 3600) {
                        this.getAccessToken();
                    } else {
                        this.getArtists();
                    }
                }
            },
            getAccessToken() {
                const clientId = "d1d9de4848224f658c4c60c52724456a";
                const clientSecret = "57a178e09ba1495198719aabb20a8299";
                const authString = btoa(`${clientId}:${clientSecret}`);

                axios.post('https://accounts.spotify.com/api/token', new URLSearchParams({
                    grant_type: 'client_credentials'
                }), {
                    headers: {
                        'Content-Type': 'application/x-www-form-urlencoded',
                        'Authorization': `Basic ${authString}`
                    }
                }).then(response => {
                    this.token = response.data.access_token;
                    localStorage.setItem('spotifyToken', this.token);
                    localStorage.setItem('tokenCreationTime', Date.now() / 1000);
                    this.getArtists();
                }).catch(error => {
                    console.error(error);
                });
            },
            getArtists() {
                const playlistId = '6gxr6s1C9YImDsoJgiVGxC';
                const endpoint = `https://api.spotify.com/v1/playlists/${playlistId}/tracks`;

                axios.get(endpoint, {
                    headers: {
                        'Authorization': `Bearer ${this.token}`
                    }
                }).then(response => {
                    const trackItems = response.data.items;
                    const artistIds = [...new Set(trackItems.flatMap(item => item.track.artists.map(artist => artist.id)))];

                    const artistPromises = artistIds.map(id =>
                        axios.get(`https://api.spotify.com/v1/artists/${id}`, {
                            headers: {
                                'Authorization': `Bearer ${this.token}`
                            }
                        })
                    );

                    Promise.all(artistPromises).then(artistsResponse => {
                        this.artistsData = artistsResponse.map(res => res.data);
                        this.isLoading = false;
                        this.formatArtists();
                    }).catch(error => {
                        console.error(error);
                        this.isLoading = false;
                    });
                }).catch(error => {
                    console.error(error);
                    this.isLoading = false;
                });
            },
            formatArtists() {
                const numColumns = 3;
                this.rows = Array.from(
                    { length: Math.ceil(this.artistsData.length / numColumns) },
                    (_, index) => this.artistsData.slice(index * numColumns, (index + 1) * numColumns)
                );
            }
        }
    };
</script>

<style>
    .artist-list {
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    .artist-row {
        display: flex;
        flex-wrap: wrap;
        justify-content: center;
    }

    .artist-card {
        display: inline-block;
        margin: 10px;
        width: 350px;
        background-color: #434343;
        border: 1px solid #ddd;
        border-radius: 10px;
        box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
    }

    .artist-content {
        display: flex;
        align-items: center;
        place-items: center;
        font-synthesis-style: revert;
        flex-direction: row;
    }

    .artist-image-container {
        flex: 1;
        margin-right: 10px;
        scrollbar-color: initial;
    }

    .artist-image {
        height: fit-content;
        border-radius: 10%;
        margin-right: 10px;
        width: 100%;
        column-fill: balance-all;
        pitch-range: revert-layer;
        border-radius: 10px;
    }

    .artist-info {
        flex-grow: 1;
    }

    .loading-container {
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .loading-spinner {
        border: 4px solid #ccc;
        border-top: 4px solid #181818;
        border-radius: 50%;
        width: 40px;
        height: 40px;
        animation: spin 1s linear infinite;
    }

    @keyframes spin {
        0% {
            transform: rotate(0deg);
        }

        100% {
            transform: rotate(360deg);
        }
    }
</style>
