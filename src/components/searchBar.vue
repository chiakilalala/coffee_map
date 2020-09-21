<template>
  <div class="category" :class="{'close' : !isOpen}">
    <div class="search city-selector-set">
      <select
        name="country"
        class="country"
        v-model="selected.country"
        @change="selected.dist = null"
      >
        <option :value="item" v-for="(item, index) in country" :key="index">{{ item }}</option>
      </select>
      <select name="dist" class="dist" v-model="selected.dist" @change="passLatLng">
        <option :value="null" selected>-- 請選擇 --</option>
        <option v-for="(item, index) in dist" :key="index" :value="item">{{ item }}</option>
      </select>
      <!-- <p>
        有取得口罩數量的藥局有
        <span>{{ list.filter(item => item.mask_child || item.mask_adult).length }}</span>
        家
      </p> -->
    </div>
    <ul class="list">
      <li
        v-for="(item, index) in list"
        :key="index"
        @click="moveToDragStore(item.latitude,item.longitude)"
      >
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
          <!-- <span>{{item.properties.note}}</span> -->
        </div>
        <!-- <div class="btnWrap">
          <button
            :class="{'soldout': !item.properties.mask_adult}"
          >成人：{{item.properties.mask_adult | mask}}</button>
          <button
            :class="{'soldout': !item.properties.mask_child}"
          >兒童：{{item.properties.mask_child | mask}}</button>
        </div> -->
      </li>
    </ul>
    <button class="toggleBtn" @click="isOpen = !isOpen">
      <i class="icon_arrow" :class="{'close' : !isOpen}"></i>
    </button>
  </div>
</template>

<script>
/* eslint-disable camelcase */
const cors = 'https://cors-anywhere.herokuapp.com/' // use cors-anywhere to fetch api data
const url = 'http://cafenomad.tw/api/v1.2/cafes' // origin api url
export default {
  name: 'searchBar',
  props: {
    properties: Array
  },
  data () {
    return {
      isOpen: true,
      twCity: [],
      api: [],
      selected: {
        country: '台北市',
        dist: null
      }
    }
  },
  created () {
    this.getAPI()
    this.getCountry()
  },
  methods: {
    async getAPI () {
      const { data } = await this.axios.get(
        `${cors}${url}`
      )
      // console.log('api =>', data)
      this.api = Object.freeze(data)
    },
    async getCountry () {
      try {
        const { data } = await this.axios.get('./latlng.json')
        this.twCity = data
        // console.log('tw =>', data)
      } catch (err) {
        console.log(err)
      }
    },
    passLatLng () {
      if (!this.selected.dist) {
        return
      }
      const apiPassArr = this.api.filter(
        (item) =>
          item.address.includes(this.selected.country) &&
          item.address.includes(this.selected.dist)
      )
      this.$emit('pass-evt', apiPassArr)
    },
    moveToDragStore (long, lon) {
      if (!this.selected.country || !this.selected.dist) {
        return console
      }
      this.$emit('pass-move-drag', long, lon)
    }
  },
  computed: {
    country () {
      return this.twCity
        .map((item) => item.city)
        .filter((item, index, arr) => arr.indexOf(item) === index)
    },
    dist () {
      return this.twCity
        .filter((item) => item.city === this.selected.country)
        .map((item) => item.district)
        .filter((item, index, arr) => arr.indexOf(item) === index)
    },
    list () {
      return this.api.filter((item) => {
        // console.log(item)
        if (!this.selected.dist) {
          return item.address.includes(this.selected.country)
        }
        return (
          item.address.includes(this.selected.country) &&
          item.address.includes(this.selected.dist)
        )
      })
    }
  },
  watch: {
    list (val) {
      console.log(val)
    },
    moveToDragStore (latitude, longitude) {
      console.log(latitude, longitude)
    }
  }

}

</script>
<style lang="scss" scoped>
@import '@/assets/scss/style';

</style>
