<template>
  <div class="container">
    <div style="width:100%;">
      <h1 class="title">
        brewMap
      </h1>
      <h2 class="subtitle">
        {{ loading }}
      </h2>
      <div
        id="map"
        class="map"
      />
    </div>
  </div>
</template>

<script>
import Logo from '~/components/Logo.vue'
import fetchPaginate from 'fetch-paginate/bundle'
import 'ol/ol.css'
import { Map, View } from 'ol'
import TileLayer from 'ol/layer/Tile'
import VectorLayer from 'ol/layer/Vector'
import XYZSource from 'ol/source/XYZ'
import { fromLonLat } from 'ol/proj'
import OSM from 'ol/source/OSM'
import BingMaps from 'ol/source/BingMaps'
import Point from 'ol/geom/Point'
import { Circle as CircleStyle, Fill, Stroke, Style } from 'ol/style'
import Feature from 'ol/Feature'
import VectorSource from 'ol/source/Vector'

export default {
  name: '',
  components: {

  },
  data () {
    return {
      // model
      brews: [],
      coords: [],
      map: '',
      caliLongLat: [-119.975209, 37.393630],
      fStyle: '',
      loading: 'Fetching Cali Breweries, Please wait'
    }
  },
  computed: {
    // state
  },
  mounted () {
    this.createMap()
    this.nextAction()
  },

  methods: {
    async getMarkersCoords (USAstate) {
      const url = ('https://api.openbrewerydb.org/breweries?by_state=' + USAstate)
      // easy library to getting all the pages from api
      // a bit slow since it has to do it in sequence
      const { data, res } = await fetchPaginate(url, {
        params: true
      })

      this.loading = 'Cali Breweries Loaded!'
      this.brews = data

      // turn long lat coordinates into openlayers mercator format
      this.coords = data.map(b => {
        return fromLonLat([parseFloat(b.longitude), parseFloat(b.latitude)])
      })
    },
    createMap () {
      // add map to page using bing map api
      this.map = new Map({
        target: 'map',
        layers: [
          new TileLayer({
            preload: Infinity,
            source: new BingMaps({
              key: 'AsblHfUMMsDXMHr3aWBasxLNTM4HUXwMnFK-fulrUMGc7NrNXBqbys3HXwZvL8Bn',
              imagerySet: 'RoadOnDemand'
            })
          })
        ],
        view: new View({
          center: fromLonLat(this.caliLongLat),
          zoom: 5
        })
      })
    },
    // sets up the style for the map markers
    featureStyleSetup () {
      this.fstyle = new Style({
        image: new CircleStyle({
          radius: 6,
          fill: new Fill({
            color: '#3399CC'
          }),
          stroke: new Stroke({
            color: '#fff',
            width: 2
          })
        })
      })
    },
    addMarkers () {
      // create array of markers for the coordinates we got before
      let features = []
      for (let coordinates of this.coords) {
        let positionFeature = new Feature()
        positionFeature.setStyle(this.fStyle)
        positionFeature.setGeometry(coordinates[0]
          ? new Point(coordinates) : null)
        features.push(positionFeature)
      }

      // add markers to the map
      const v = new VectorLayer({
        map: this.map,
        source: new VectorSource({
          features: features
        })
      })
    },
    modelPresent (data) {},
    async nextAction () {
      await this.getMarkersCoords('california')
      this.featureStyleSetup()
      this.addMarkers()
    }

  }
}
</script>

<style>
.map {
        width: 100%;
        height:400px;
      }

.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family: "Quicksand", "Source Sans Pro", -apple-system, BlinkMacSystemFont,
    "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}
</style>
