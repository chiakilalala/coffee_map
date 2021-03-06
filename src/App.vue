<template>
  <div id="app">
    <searchBar @pass-evt="listMask " @pass-move-drag="moveDrag"></searchBar>
    <l-map
      :center="center"
      :zoom="zoom"
      :options="{zoomControl: false}"
      style="height: 100%; width: 100%"
      @update:zoom="zoomUpdated"
      @update:center="centerUpdated"
      @update:bounds="boundsUpdated"
    >
      <l-tile-layer :url="url"></l-tile-layer>
      <l-control-zoom position="topright"></l-control-zoom>
      <v-marker-cluster>
        <l-marker
          v-for="(item, index) in list"
          :key="index"
          :lat-lng="[item.longitude, item.latitude]"
        >
          <l-popup>
            <h3>{{item.name}}</h3>
            <p>
              <i class="icon_pin"></i>
              {{item.address}}
            </p>
            <p>
              <i class="icon_tel"></i>
              {{item.open_time}}
            </p>
            <div class="noteWrap">
              <i class="icon_remark"></i>
              <span>{{item.open_time}}</span>
            </div>
            <!-- <div class="btnWrap">
              <button :class="{'soldout': !item.properties.mask_adult}">
                成人：{{ item.properties.mask_adult | mask }}
              </button>
              <button :class="{'soldout': !item.properties.mask_child}">
                兒童：{{ item.properties.mask_child | mask }}
              </button>
            </div>-->
          </l-popup>
        </l-marker>
      </v-marker-cluster>
    </l-map>
  </div>
</template>

<script>
import L from 'leaflet'
import { LMap, LTileLayer, LMarker, LControlZoom, LPopup } from 'vue2-leaflet'
import Vue2LeafletMarkerCluster from 'vue2-leaflet-markercluster'
import searchBar from './components/searchBar.vue'

// let openStreetMap = {}

export default {
  name: 'App',
  components: {
    searchBar,
    LMap,
    LTileLayer,
    LMarker,
    LControlZoom,
    LPopup,
    'v-marker-cluster': Vue2LeafletMarkerCluster
  },
  data () {
    return {
      zoom: 13,
      center: [22.612961, 120.304167],
      bounds: null,
      list: [],
      map: [],
      marker: L.latLng(22.612961, 120.304167),
      url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
      attribution:
        '© <a href="http://osm.org/copyright">OpenStreetMap</a> contributors'
    }
  },
  methods: {
    updateMap () {
      this.list.forEach((item) => {
        // 透過藥局經緯度疊加標記
        L.Marker([item.latitude, item.longitude]).addTo(L.Map)
      })
    },
    zoomUpdated (zoom) {
      this.zoom = zoom
    },
    centerUpdated (center) {
      this.center = center
    },
    boundsUpdated (bounds) {
      this.bounds = bounds
    },
    listMask (passArr) {
      this.list = passArr

      this.center = [passArr.latitude, passArr.longitude]
      this.zoom = 13

      console.log(passArr, 'this arr')
      setTimeout(() => {
        this.center = [passArr.latitude, passArr.longitude]
      }, 500)
    },
    moveDrag (latitude, longitude) {
      this.zoom = 18
      this.center = [latitude, longitude]
    }
  },
  computed: {
    markList () {
      return this.list.map((item) => {
        return {
          latitude: item.latitude,
          longitude: item.longitude
        }
      })
    }
  },
  watch: {
    markList (val) {
      console.log(val)
    },
    list (val) {
      this.updateMap()
    }
  },
  mounted () {
    // openStreetMap = L.map('app', {
    //   center: [25.0401, 121.512],
    //   zoom: 13
    // })
    // L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    //   attribution:
    //       'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
    //   maxZoom: 20
    // }).addTo(openStreetMap)
  }
}
</script>

<style lang="scss">
@import "@/assets/scss/helpers/_reset";
@import "@/assets/scss/helpers/_breakpoint";
@import "~leaflet.markercluster/dist/MarkerCluster.css";
@import "~leaflet.markercluster/dist/MarkerCluster.Default.css";

html,
body {
  width: 100%;
  height: 100%;
  overflow: hidden;
}
#app {
  width: 100%;
  height: 100%;
}
.leaflet-popup-content-wrapper {
  border: 2px solid #668afe;
  padding: 0;
  div.leaflet-popup-content {
    margin: 0;
    width: 340px;
    padding: 20px 20px 15px;
    box-sizing: border-box;
    h3 {
      color: #333333;
      font-size: 18px;
      margin-bottom: 10px;
      letter-spacing: 1.5px;
    }
    p {
      font-size: 14px;
      color: #888888;
      margin: 0 0 8px 0;
      letter-spacing: 1px;
      white-space: nowrap;
      overflow-x: auto;
      overflow-y: hidden;
    }
    ::-webkit-scrollbar {
      display: none;
    }
    span {
      color: #888888;
      letter-spacing: 0.5px;
      line-height: 20px;
      font-size: 14px;
      width: calc(100% - 15px);
    }
    .noteWrap {
      display: flex;
      align-items: baseline;
    }
  }
  .btnWrap {
    margin-top: 10px;
  }
  button {
    padding: 12px 0;
    width: 49%;
    color: white;
    background: #668afe;
    border-radius: 0 5px 5px 0;
    border: none;
    letter-spacing: 1px;
    outline: none;
    user-select: none;
    font-size: 14px;
    max-width: 49%;
    max-height: 44px;
    &.soldout {
      background: #e2e2e2;
      color: #9c9c9c;
    }
    &:nth-child(1) {
      border-radius: 5px 0 0 5px;
      margin-right: 1px;
    }
    &.toggleBtn {
      position: absolute;
      top: 0;
      bottom: 0;
      right: -18px;
      border-radius: 0 12px 12px 0;
      margin: auto;
      width: 18px;
      height: 48px;
      padding: 13px 1.2px;
      cursor: pointer;
      transition: transform 0.3s;
      &:hover {
        transform: scale(1.2) translateX(2px);
      }
    }
  }
  i {
    display: inline-block;
    width: 15px;
    height: 15px;
    margin-right: 10px;
    transform: translateY(2px);
    &.icon_pin {
      background: url("~@/assets/pin.svg") center center / contain no-repeat;
    }
    &.icon_tel {
      background: url("~@/assets/tel.svg") center center / contain no-repeat;
    }
    &.icon_remark {
      background: url("~@/assets/remark.svg") center center / contain no-repeat;
      filter: invert(63%) sepia(5%) saturate(26%) hue-rotate(326deg)
        brightness(85%) contrast(85%);
    }
    &.icon_arrow {
      height: 20px;
      filter: invert(1);
      background: url("~@/assets/arrow.svg") center center / cover no-repeat;
      transition: all 0.5s;
      &.close {
        transform: rotate(180deg);
      }
    }
  }
  &::before {
    content: "";
    position: absolute;
    bottom: -11px;
    left: 0;
    right: 0;
    margin: auto;
    width: 21px;
    height: 21px;
    background: white;
    border: 2px solid #668afe;
    border-top: 0;
    border-left: 0;
    transform: rotate(45deg);
  }
}
.leaflet-popup-close-button {
  margin: 5px 5px 0 0;
}
@include rwd(576px) {
  .leaflet-popup-content-wrapper {
    div.leaflet-popup-content {
      max-width: 280px;
      padding: 15px 20px 10px 20px;
      h3 {
        font-size: 16px;
      }
    }
    button {
      padding: 8px 0;
      font-size: 12px;
    }
  }
}
</style>
