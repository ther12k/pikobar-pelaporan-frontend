<template>
  <v-card class="chart mx-auto" outlined>
    <v-card-text class="pa-0">
      <div id="map" style="height: 450px; z-index: 0;" />
    </v-card-text>
  </v-card>
</template>

<script>
import { mapGetters } from 'vuex'
import 'leaflet/dist/leaflet.css'
import L from 'leaflet'
import jsonCity from '../../../json/kota.json'
import jsonDistrict from '../../../json/kecamatan.json'
import jsonVillage from '../../../json/kelurahan.json'

export default {
  name: 'MapPoint',
  props: {
    filterData: {
      type: Object,
      default: null
    }
  },
  data() {
    return {
      jsonCity,
      jsonDistrict,
      jsonVillage,
      map: null,
      url: 'https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png',
      zoom: 8,
      center: [-6.932694, 107.627449],
      layerGroup: null,
      dataLayer: [],
      dataMarker: [],
      styleBorderline: {
        fillColor: 'red',
        fillOpacity: 0.01,
        weight: 0.5,
        opacity: 0.5,
        color: 'red'
      }
    }
  },
  computed: {
    ...mapGetters('user', [
      'roles',
      'fullname',
      'district_user',
      'district_name_user'
    ])
  },
  watch: {
    'filterData': {
      handler: function(value, oldValue) {
        if (value.isVillage) {
          this.removeLayer()
          this.createLayerVillageSingle(value.village)
        } else if (value.isDistrict) {
          this.removeLayer()
          this.createLayerVillage(value.district)
        } else if (value.isCity) {
          this.removeLayer()
          this.createLayerDistrict(value.city)
        } else {
          this.removeLayer()
          this.createLayerCity()
        }
      },
      deep: true
    }
  },
  mounted() {
    this.initMap()

    if (this.roles[0] === 'dinkesprov') {
      this.createLayerCity()
    } else if (this.roles[0] === 'dinkeskota') {
      this.createLayerDistrict(this.filterData.city)
    }
  },
  methods: {
    initMap() {
      this.map = L.map('map', {
        zoomControl: false
      }).setView(this.center, this.zoom)

      L.tileLayer(
        this.url,
        {
          attribution: '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
          maxZoom: 18
        }
      ).addTo(this.map)

      L.control
        .zoom({
          position: 'bottomright'
        })
        .addTo(this.map)

      // this.map.on('zoomend', () => {
      //   if (this.map.getZoom() <= 10) {
      //     this.removeLayer()
      //     this.createLayerCity()
      //   } else if (this.map.getZoom() > 10 && this.map.getZoom() <= 13) {
      //     this.removeLayer()
      //     this.createLayerDistrict(this.district_user)
      //   } else if (this.map.getZoom() > 13) {
      //     this.removeLayer()
      //     this.createLayerVillage()
      //   }
      // })

      this.layerGroup = L.layerGroup().addTo(this.map)
    },
    removeLayer() {
      this.dataLayer.forEach(element => {
        this.map.removeLayer(element)
      })
      this.dataLayer = []
    },
    createLayerCity() {
      const geojsonLayer = L.geoJSON(this.jsonCity, {
        style: feature => {
          return this.styleBorderline
        },
        onEachFeature: (feature, layer, element) => {
          const polygon = layer.addTo(this.map)
          this.dataLayer.push(polygon)

          const nameCity = feature.properties.kemendagri_kabupaten_nama
          layer.bindTooltip(this.titleize(nameCity))
        }
      })

      if (geojsonLayer.getLayers() && geojsonLayer.getLayers().length > 1) {
        this.map.fitBounds(geojsonLayer.getBounds())
      } else if (geojsonLayer.getLayers() && geojsonLayer.getLayers().length === 1) {
        this.map.setView(geojsonLayer.getLayers()[0].getLatLng(), 8)
      }
    },
    createLayerDistrict(value) {
      const geojsonLayer = L.geoJSON(this.jsonDistrict, {
        style: feature => {
          return this.styleBorderline
        },
        filter: function(feature, layer) {
          return feature.properties.kemendagri_kabupaten_kode === value
        },
        onEachFeature: (feature, layer, element) => {
          const polygon = layer.addTo(this.map)
          this.dataLayer.push(polygon)

          const nameCity = feature.properties.kemendagri_kabupaten_nama
          const nameDistrict = feature.properties.kemendagri_kecamatan_nama
          layer.bindTooltip(`
            ${this.titleize(nameCity)} <br>
            ${this.$t('label.select_sub_district')} ${nameDistrict}
          `)
        }
      })

      if (geojsonLayer.getLayers() && geojsonLayer.getLayers().length > 1) {
        this.map.fitBounds(geojsonLayer.getBounds())
      } else if (geojsonLayer.getLayers() && geojsonLayer.getLayers().length === 1) {
        this.map.setView(geojsonLayer.getLayers()[0].getLatLng(), 12)
      }
    },
    createLayerVillage(value) {
      const geojsonLayer = L.geoJSON(this.jsonVillage, {
        style: feature => {
          return this.styleBorderline
        },
        filter: function(feature, layer) {
          return feature.properties.kemendagri_kecamatan_kode === value
        },
        onEachFeature: (feature, layer, element) => {
          const polygon = layer.addTo(this.map)
          this.dataLayer.push(polygon)

          const nameCity = feature.properties.kemendagri_kabupaten_nama
          const nameDistrict = feature.properties.kemendagri_kecamatan_nama
          const nameVillage = feature.properties.kemendagri_desa_nama
          layer.bindTooltip(`
            ${this.titleize(nameCity)} <br>
            ${this.$t('label.select_sub_district')} ${nameDistrict} <br>
            ${this.$t('label.select_village')} ${nameVillage}
          `)
        }
      })

      if (geojsonLayer.getLayers() && geojsonLayer.getLayers().length > 1) {
        this.map.fitBounds(geojsonLayer.getBounds())
      } else if (geojsonLayer.getLayers() && geojsonLayer.getLayers().length === 1) {
        this.map.setView(geojsonLayer.getLayers()[0].getLatLng(), 12)
      }
    },
    createLayerVillageSingle(value) {
      const geojsonLayer = L.geoJSON(this.jsonVillage, {
        style: feature => {
          return this.styleBorderline
        },
        filter: function(feature, layer) {
          return feature.properties.kemendagri_desa_kode === value
        },
        onEachFeature: (feature, layer, element) => {
          const polygon = layer.addTo(this.map)
          this.dataLayer.push(polygon)

          const nameCity = feature.properties.kemendagri_kabupaten_nama
          const nameDistrict = feature.properties.kemendagri_kecamatan_nama
          const nameVillage = feature.properties.kemendagri_desa_nama
          layer.bindTooltip(`
            ${this.titleize(nameCity)} <br>
            ${this.$t('label.select_sub_district')} ${nameDistrict} <br>
            ${this.$t('label.select_village')} ${nameVillage}
          `)
        }
      })

      if (geojsonLayer.getLayers() && geojsonLayer.getLayers().length > 1) {
        this.map.fitBounds(geojsonLayer.getBounds())
      } else if (geojsonLayer.getLayers() && geojsonLayer.getLayers().length === 1) {
        this.map.fitBounds(geojsonLayer.getBounds())
      }
    },
    titleize(sentence) {
      if (!sentence.split) {
        return sentence
      }

      const titleizeWord = (name) => {
        return name.charAt(0).toUpperCase() + name.slice(1).toLowerCase()
      }

      const result = []
      sentence.split(' ').forEach(w => {
        result.push(titleizeWord(w))
      })
      return result.join(' ')
    }
  }
}
</script>

<style scoped>
</style>
