<template>
  <div id="app">
    <div class="row no-gutters">
      <!-- 選擇地區 -->
      <div class="toolbox col-sm-3 p-2 bg-white">
        <div class="form-group d-flex">
          <label for="city" class="col-form-label mr-2 text-right">縣市</label>
          <div class="flex-fill">
            <select id="city" class="form-control" v-model="select.city">
              <!-- 製作下拉選單 -->
              <option :value="city.name" v-for="city in cityName" :key="city.name">
                 {{city.name }}
              </option>
            </select>
          </div>
        </div>
        <div class="form-group d-flex">
          <label for="dist" class="col-form-label mr-2 text-right">地區</label>
          <div class="flex-fill">
            <select id="dist" class="form-control" v-model="select.dist">
               <option :value="dist.name"
                  v-for="dist in cityName.find((city) => city.name === select.city).districts"
                  :key="dist.name">
                 {{ dist.name }}
               </option>
              <!-- 製作下拉選單 -->
            </select>
          </div>
        </div>
      </div>

      <!-- 顯示地圖和 UBike 站點 -->
      <div class="col-sm-9">
        <div id="map"></div>
      </div>
    </div>
  </div>
</template>

<script>
import L from 'leaflet';
import cityName from './assets/cityName.json';


export default {
  name: 'App',
  data: () => ({
    cityName,
    // select 先暫時設定為空物件
    select: {
      city: '臺北市',
      dist: '中正區',
    },
    ubikes: [],
    osmMap: {},
  }),
  computed: {
    // 防止不必要的 render
    youbikes() {
      return this.ubikes.filter((bike) => bike.sarea === this.select.dist);
    },
  },
  watch: {
    // 監聽變化過後的樣子
    youbikes() {
      this.updateMap();
    },
  },
  methods: {
    updateMap() {
      this.osmMap.eachLayer((layer) => {
        if (layer instanceof L.Marker) {
          // this.osmMap.removeLayer(layer);
          layer.remove();
        }
      });

      this.youbikes.forEach((bike) => {
        L.marker([bike.lat, bike.lng]).addTo(this.osmMap).bindPopup(`
          <p>
          <strong style="font-size: 20px;">${bike.sna}</strong></p>
          <strong style="font-size: 16px; color: #d45345;">可租借車輛剩餘：${bike.sbi} 台</strong>
          <br>
            可停空位剩餘: ${bike.bemp}
          <br>
          <small>最後更新時間: ${bike.mday}</small>
        `);
      });

      this.cityName[0].districts.find((dist) => {
        if (dist.name === this.select.dist) {
          this.osmMap.panTo(new L.LatLng(dist.latitude, dist.longitude));
        }
        return dist.name === this.select.dist;
      });
    },

  },
  created() {
    const api = 'https://tcgbusfs.blob.core.windows.net/blobyoubike/YouBikeTP.json';
    this.$http.get(api).then((response) => {
      console.log(response.data);
      // this.ubikes = Object.keys(response.data.retVal).map((key) => response.data.retVal[key]);
      this.ubikes = Object.values(response.data.retVal);
    });
  },
  mounted() {
    this.osmMap = L.map('map', {
      center: [25.041956, 121.508791],
      zoom: 18,
    });

    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png?', {
      attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
      maxZoom: 18,
    }).addTo(this.osmMap);
  },
};
</script>

<style lang="scss">
 @import 'bootstrap/scss/bootstrap';

 #map {
   position: relative;
   height: 100vh;
 }
</style>
