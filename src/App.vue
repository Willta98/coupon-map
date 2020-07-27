<template>
  <div id="app">
    <div class="row no-gutters">
       <div class="col-sm-4">
        <div class="container">
         <div class="toolbox"> 
          <h2 class="text-center title bg-info">振興三倍券-領取地圖</h2>
          <div class="form-group d-flex">
              <select id="cityName" class="form-control" v-model="select.city"
              @change="updatetown()">
                <option value="">-- Select One --</option>
                <option :value="c" v-for="c in city" :key="c">{{ c }}</option>
              </select>
          </div>
          <div class="form-group d-flex">
              <select id="townName" class="form-control" v-model="select.area"
               @change="removeMarker(); updatstore()">
                <option value="">-- Select One --</option>
                <option :value="a" v-for="a in area" :key="a">{{ a }}</option>
              </select>
          </div>
          <div class="card" width=100 v-for="(item) in store" :key="item.id">
            <div class="card-body">
             <h5 class="card-title">分局 :{{ item.storeNm }}</h5>
              <p class="card-text">地址:{{ item.addr }}</p>
              <p class="card-text">電話:{{ item.tel }}</p>
              <p class="coupword bg-primary text-center">振興券存量:{{ item.total }}</p>
            </div>
          </div>
        </div>
       </div>  
      </div>
     <div class="col-sm-8">
         <div id="map"></div>
    </div>
   </div>
  </div>
</template>
<script>
import $ from "jquery";
import L from "leaflet";

let osmMap = {};

export default {
  data() {
    return {
      alldata: [],
      hsnNm: [],
      town: [],
      store: [],
      city: {},
      area: {},
      select: {
        city:"",
        area:"",
      },
    };
  },
  methods: {
    getcoupon() {
      const vm = this;
      let url = "https://3000.gov.tw/hpgapi-openmap/api/getPostData";
      vm.$http.get(url).then((response) => {
        vm.alldata = response.data
        response.data.forEach((item) => {
          vm.hsnNm.push(item.hsnNm);
        });
        vm.city = vm.hsnNm.filter(function (element, index, arr) {
          return arr.indexOf(element) === index;
        });
      });

      osmMap = L.map('map',{
        center: [25.03, 121.55],
        zoom: 15,
      });
       L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '<a target="_blank" href="https://www.openstreetmap.org/">© OpenStreetMap 貢獻者</a>',
      maxZoom: 18,
       }).addTo(osmMap);
    },
    updatetown(){
      const vm = this;
        vm.town = [];
        vm.alldata.forEach((item)=>{
           if (vm.select.city === item.hsnNm) {
             vm.town.push(item.townNm);
         }
      })
       vm.area = vm.town.filter(function (element, index, arr) {
          return arr.indexOf(element) === index;
      });
    },
    updatstore(){
      const vm = this;
      vm.storeall = vm.alldata.filter((item) => {
        return item.hsnNm === vm.select.city
      })
      console.log(vm.storeall)
      vm.store = vm.storeall.filter((item) => {
        return item.townNm === vm.select.area
      })
      // 
      vm.store.forEach(item => {
      L.marker([item.latitude , item.longitude]).addTo(osmMap).bindPopup(`<strong>${item.storeNm}</strong> <br>
    三倍卷剩餘數量：<strong>${item.total}</strong><br>
    地址: <a href="https://www.google.com.tw/maps/place/${item.addr}" target="_blank">${item.addr}</a><br>
    電話: ${item.tel}<br>
    <small>最後更新時間: ${item.updateTime}</small>`);
      });
      this.penTo(vm.store[0])
    },
     removeMarker() {
       osmMap.eachLayer((layer) => {
      if (layer instanceof L.Marker) {
        osmMap.removeLayer(layer);
      }
     });
    },
    penTo(item) {
      osmMap.panTo([item.latitude, item.longitude]);
    },
  },
  mounted() {
    this.getcoupon();
  },
};
</script>

<style lang="scss">
@import "bootstrap/scss/bootstrap";
#map {
  height: 100vh;
}
.title {
  color: white;
  padding-bottom: 25px;
  padding-top: 25px;
  border-radius: 10px;
  margin-top: 10px;
}
.coupword {
  color: white;
  border-radius: 10px;
}
.card{
  margin-bottom: 5px;
}
.toolbox{
  height: 100vh;
  overflow-y: auto;
}
</style>
