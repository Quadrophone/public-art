<template>
    <div id="app">

        <l-map :zoom="zoom" :center="center" v-if="artWorks">
            <l-tile-layer :url="url" :attribution="attribution"></l-tile-layer>
            <l-marker @click="selectedArtwork=artWork" v-for="artWork in artWorks"  :key="artWork.id" :lat-lng="artWork.coordinates">
                <l-popup v-if="artWork.photo">
                    <img :src="artWork.photo">
                </l-popup>
            </l-marker>
        </l-map>
        <div class="description" id="description" v-if="selectedArtwork">
            <span class="close" @click="selectedArtwork=null">&times;</span>
            <h2>{{selectedArtwork.title}}</h2>
            <h3>{{artistNames(selectedArtwork.artists)}}</h3>
            <p>{{selectedArtwork.description}}</p>
            <p>{{selectedArtwork.statement}}</p>
        </div>
    </div>
</template>
<script>
import axios from 'axios'
import L from 'leaflet'
delete L.Icon.Default.prototype._getIconUrl  
L.Icon.Default.mergeOptions({  
  iconRetinaUrl: require('./assets/marker.png'),  
  iconUrl: require('./assets/marker.png'),  
  iconSize: [ 45, 45 ],

})
import "leaflet/dist/leaflet.css"
import { LMap, LTileLayer, LMarker, LPopup } from 'vue2-leaflet'
import artWorks from './assets/art.json';


export default {
    name: 'app',
    data() {
        return {
            zoom: 16,
            center: L.latLng(49.2827, -123.1207),
            url: 'https://maps.wikimedia.org/osm-intl/{z}/{x}/{y}.png',
            attribution: '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
            artWorks: null,
            artistList:null,
            selectedArtwork: null
        }
    },
    components: { LMap, LTileLayer, LMarker, LPopup },
    mounted() {
        var self = this;
        navigator.geolocation.getCurrentPosition(function(position) {
            self.center =  L.latLng(position.coords.latitude, position.coords.longitude);
        });
        this.artistList = artWorks[0].features;
        this.artWorks = artWorks[1].features
            .filter(function(a) { return a.properties.Latitude && a.properties.Longitude})
            .map(function(a) {
                return {
                    id: a.properties.RegistryID,
                    coordinates: L.latLng(a.geometry.coordinates[0] / 10000, a.geometry.coordinates[1] / 100000),
                    coordinates: L.latLng(a.properties.Latitude, a.properties.Longitude),
                    photo: a.properties.PhotoURL,
                    title:a.properties.TitleOfWork,
                    description: a.properties.DescriptionOfwork,
                    statement: a.properties.ArtistProjectStatement,
                    artists: a.properties.Artists.split(';').map(function(a) { return parseInt(a)})
                }
            });
    },
    methods: {
        artistNames(ids) {
            var self = this;
            let artists = [];
            ids.forEach(function(id) {
                let artist = self.artistList.find(function(a) { return a.properties.ARTISTID == id});
                if (artist) {
                    artists.push((artist.properties.FIRSTNAME || '') + ' ' + artist.properties.LASTNAME)
                }
            })
            return artists.join(', ')
        }
    },
    watch: {
        selectedArtwork(work) {
            this.$nextTick(function() {
                document.getElementById("description").scrollTop = 0;
            })
        }
    },
    computed: {
 
    }
  
}
</script>
<style>
* {box-sizing: border-box}
@import url('https://fonts.googleapis.com/css?family=Open+Sans:400,700');

html,
body {
    margin: 0;
    padding: 0;
    font-family: 'Open Sans',sans-serif;
}

#app {
    height: 100vh;
    width: 100%;
    margin: 0;
}
.description {
    position: fixed;
    bottom:0;
    left:0;
    background:#fff;
    z-index: 9999999999;
    max-height:40vh;
    overflow-y:auto;
    max-width: 100vw;
    width:100%;
    padding:1rem;
    box-shadow: 0px -4px 14px -2px rgba(0,0,0,0.26);

}
.leaflet-popup-content-wrapper, .leaflet-popup-content {
    min-width: 170px;
    text-align:center;
    border-radius: 2px;
}

.description p {
    max-width: 90%;
    line-height: 1.6em;
}
.description .close {
    font-size:1.2rem;
    font-weight: 700;
    position: absolute;
    top: 0;
    right: 8px;
}

.leaflet-popup-content img {
  
    height:100px;
     width:100%;
    object-fit:contain;
}

</style>