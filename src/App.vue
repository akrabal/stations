<script setup>
import {ref,computed } from 'vue'
import haversine from 'haversine'
import "leaflet/dist/leaflet.css"
import { LMap, LTileLayer,LMarker,LPolyline,LIcon,LControl } from "@vue-leaflet/vue-leaflet"
import redCar from './assets/redCar.png'
import bluefuel from './assets/bluefuel.png'  
import redFuel from './assets/redFuel.png'

let zoom = ref(30)
let center = ref([ 6.21790086131689, 1.1200842442283416 ])
let staticAnchor =  ref([0, 0])

const res = ref()
let  stations = ref([{
  id:1,
  nom: 'Station 1',
  location:{
    latitude:6.2116988423415,
    longitude: 1.150785350344227
  },
  isNear: false,
},
{
  id:2,
  nom: 'Station 2',
  location:{
    latitude:6.210537024669773,
    longitude: 1.1367865967361463
  },
  isNear: false
  
},
{ id:3,
  nom: 'Station 3',
  location:{
    latitude:6.2218159327722455,
    longitude:  1.1260122013083496
  },
  isNear: false
}
])
const user = ref({
  name:'akrabal',
  location:{
    latitude: 6.21790086131689, 
    longitude: 1.1200842442283416
  },
  distanse:0,
  bat:100,
  route:[]
}) 

const loader = ref(false)




const tab = computed(()=>{
  if(res.value){
    return res.value.map(point => {
      return [point[1],point[0]]
    })
  }
  return []
})

const sationsComputed = computed(()=>{
  let  newStations = stations.value.map(station => {
  return {...station,distanse:haversine(user.value.location,station.location) }
  });

  newStations = newStations.sort((a,b) => a.distanse - b.distanse)
  newStations[0].isNear = true

  return newStations
})

function moveToStation(){
   let i = 0
   const intervalID = setInterval(()=>{
    user.value.location = {
      latitude:  tab.value[i][0], 
      longitude: tab.value[i][1]
    }
    user.value.route.push(user.value.location)
    user.value.distanse += haversine(user.value.location,user.value.route.reverse()[1])
    user.value.bat =  user.value.bat - (haversine(user.value.location,user.value.route.reverse()[1])*7)


  if(i <= tab.value.length -1/* &&user.value.bat > 0 */){
    i++
  }
  if(i == tab.value.length /* || user.value.bat <=0 */){

    clearInterval(intervalID);
  }
  }, 500)
}




async function  callDirectionApi({startLatitude,startLongitude},{endLatitude,endLongitude}) {
  try {
      loader.value = true
      const res = await fetch(`https://api.openrouteservice.org/v2/directions/driving-car?api_key=5b3ce3597851110001cf624816ebe9921e6846be81ba3deb45ca3eba&start=${startLongitude},${startLatitude}&end=${endLongitude},${endLatitude}`)
      const body= await res.json()
      loader.value = false
      if(body.error){
        return body.error.message      
      }
       return body?.features[0]?.geometry?.coordinates

  } catch (error) {
    loader.value = false
      return error
  }
  
}

async function handleClick (){
   res.value =  await callDirectionApi({
    startLatitude:user.value.location.latitude,
    startLongitude: user.value.location.longitude
  },{
    endLatitude: sationsComputed.value[0].location.latitude,
    endLongitude: sationsComputed.value[0].location.longitude
  })
}



function moveUser($e){
   user.value.location = {
      latitude: $e.latlng.lat,
      longitude: $e.latlng.lng
   } 
}
</script>

<template>
    
    <main>
      <l-map ref="map" @click="($e)=>{console.log($e)}" v-model:zoom="zoom" v-model:center="center" :useGlobalLeaflet="false">
        <l-tile-layer url="https://tiles.stadiamaps.com/tiles/alidade_smooth_dark/{z}/{x}/{y}{r}.png"
                    layer-type="base"
                    name="Stadia Maps Basemap">
        </l-tile-layer>
        <l-control class="custom-control">
          <div class="btn-container">
            <button class="button-31" @click="handleClick"> route  <span v-if="loader" class="loader"></span></button>
            <button class="button-31" @click="moveToStation"> déplacer  </button>
          </div>
      </l-control>
        <l-polyline :lat-lngs="tab" color="green"></l-polyline>
        <l-marker  draggable   :lat-lng="[user.location.latitude,user.location.longitude]" @move="moveUser" >
          
          <l-icon
            :icon-anchor="staticAnchor"
            class-name="someExtraClass"
          >
            <img :src="redCar">
            <!-- <div class="headline" >
                {{ user.bat }}
            </div> -->
          
          </l-icon>
        </l-marker>

        <l-marker v-for="station in sationsComputed"     :lat-lng="[station.location.latitude,station.location.longitude]" :key="station.id" >
          
          <l-icon
            :icon-anchor="staticAnchor"
            class-name="someExtraClass"
          >
          <div class="headline" >
                {{  station.nom }}
            </div>
          <img :src=" station.isNear?redFuel:bluefuel">
          
        </l-icon>
        </l-marker>
      </l-map>
    </main>
      


  
</template>

<style scoped>
   main {
  height: 90vh;
  width: 98vw;
}
.someExtraClass {
  background-color: aqua;
  padding: 10px;
  border: 1px solid #333;
  border-radius: 0 20px 20px 20px;
  box-shadow: 5px 3px 10px rgba(0, 0, 0, 0.2);
  text-align: center;
  width: auto !important;
  height: auto !important;
  margin: 0 !important;
}
.headline{
  color:white;
}
.custom-control{
  background-color: white;
  padding: 0 0.5em;
  border: 1px solid #aaa;
  border-radius: 0.1em;
}
.btn-container{
}
.button-31 {
  background-color: #222;
  border-radius: 4px;
  border-style: none;
  box-sizing: border-box;
  color: #fff;
  cursor: pointer;
  display: inline-block;
  font-family: "Farfetch Basis","Helvetica Neue",Arial,sans-serif;
  font-weight: 700;
  max-width: none;
  margin-top: 2%;
  margin-bottom: 2%;
  outline: none;
  padding: 9px 20px 8px;
  width: 100%;
}

.button-31:hover,
.button-31:focus {
  opacity: .75;
}
.loader {
    width: 20px;
    height: 20px;
    border: 5px solid #FFF;
    border-bottom-color: transparent;
    border-radius: 50%;
    display: inline-block;
    box-sizing: border-box;
    animation: rotation 1s linear infinite;
    }

@keyframes rotation {
0% {
    transform: rotate(0deg);
}
100% {
    transform: rotate(360deg);
}
} 
</style>
