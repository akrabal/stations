<script setup>
import {ref,computed } from 'vue'
import haversine from 'haversine'
import "leaflet/dist/leaflet.css"
import { LMap, LTileLayer,LMarker,LPolyline,LIcon  } from "@vue-leaflet/vue-leaflet"
import redCar from './assets/redCar.png'

let zoom = ref(30)
let center = ref([ 6.21790086131689, 1.1200842442283416 ])

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



stations.value = stations.value.map(station => {
  return {...station,distanse:haversine(user.value.location,station.location) }
});

stations.value = stations.value.sort((a,b) => a.distanse - b.distanse)
stations.value[0].isNear = true

const tab = computed(()=>{
  if(res.value){
    return res.value.map(point => {
      return [point[1],point[0]]
    })
  }
  return []
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

    console.log(user.value.bat)

  if(i <= tab.value.length -1&&user.value.bat > 0){
    i++
  }
  if(i == tab.value.length || user.value.bat <=0){

    clearInterval(intervalID);
  }
  }, 1000)
}




async function  callDirectionApi({startLatitude,startLongitude},{endLatitude,endLongitude}) {
  try {
      const res = await fetch(`https://api.openrouteservice.org/v2/directions/driving-car?api_key=5b3ce3597851110001cf624816ebe9921e6846be81ba3deb45ca3eba&start=${startLongitude},${startLatitude}&end=${endLongitude},${endLatitude}`)
      const body= await res.json()
      if(body.error){
        return body.error.message      
      }
       return body?.features[0]?.geometry?.coordinates
  } catch (error) {
      return error
  }
  
}

async function handleClick (){
   res.value =  await callDirectionApi({
    startLatitude:user.value.location.latitude,
    startLongitude: user.value.location.longitude
  },{
    endLatitude: stations.value[0].location.latitude,
    endLongitude: stations.value[0].location.longitude
  })
}

</script>

<template>
    <li v-for="station in stations">
     <strong> {{ station.nom }}</strong>  : {{ station.distanse }}
    </li>    

  
    <button @click="handleClick"> click</button>
    <button @click="moveToStation"> move</button>
    <main>
      <l-map ref="map" v-model:zoom="zoom" v-model:center="center" :useGlobalLeaflet="false">
        <l-tile-layer url="https://tiles.stadiamaps.com/tiles/alidade_smooth_dark/{z}/{x}/{y}{r}.png"
                    layer-type="base"
                    name="Stadia Maps Basemap">
        </l-tile-layer>
        <l-polyline :lat-lngs="tab" color="green"></l-polyline>
        <l-marker   :lat-lng="[user.location.latitude,user.location.longitude]" >
          
          <l-icon
            :icon-anchor="staticAnchor"
            class-name="someExtraClass"
          >
          <img :src="redCar">
          <div class="headline" >
              {{ user.bat }}
          </div>
          
        </l-icon>
        </l-marker>
      </l-map>
    </main>
      


  
</template>

<style scoped>
   main {
  height: 100vh;
  width: 100vw;
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
</style>
