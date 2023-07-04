<template>
  <main class="d-flex justify-content-center" style="margin-top: 150px">
    <div style="width: 1000px">
      <div>
        location : <span class="text-primary"> {{ location }}</span> <br />
        placeId : <span class="text-primary"> {{ placeId }}</span> <br />
        lat : <span class="text-primary"> {{ lat }} </span> <br />
        lng : <span class="text-primary"> {{ lng }} </span> <br />
      </div>
      <div>
        <div class="mt-4"><AutoComplete @selectAutoComplate="getValueFromAutoComplete" /></div>
        <div class="my-4">dynamic map <GoogleMap v-if="showGGMAp" :dataGGMap="{ lat, lng }" /></div>
        <div>
          static map
          <iframe
            width="100%"
            height="100%"
            :src="`http://maps.google.com/maps?q=${lat},${lng}&z=15&output=embed&language=th`"
          ></iframe>
        </div>
      </div>
    </div>
  </main>
</template>
<script setup lang="ts">
import { onMounted, ref } from 'vue'
import AutoComplete from './components/AutoComplete/AutoCompleteGGMap.vue'
import GoogleMap from './components/GGMap/GoogleMap.vue'
import type { AutoComplateGGMap } from './components/AutoComplate/AutoComplate'
const apiKey = import.meta.env.VITE_GG_MAP_API_KEY

const location = ref()
const placeId = ref()
const lat = ref()
const lng = ref()
const showGGMAp = ref(false)
let getPostitionSuccess: any = null

function getValueFromAutoComplete(payload: AutoComplateGGMap) {
  location.value = payload.location
  placeId.value = payload.placeId
  lat.value = payload.lat
  lng.value = payload.lng
}

function successLoadLocation(position: any) {
  lat.value = position.coords.latitude
  lng.value = position.coords.longitude
  showGGMAp.value = true
  getPostitionSuccess()
}
function errorLoadLocation() {
  console.log('something error')
}

function getClientPosition() {
  navigator.geolocation.getCurrentPosition(successLoadLocation, errorLoadLocation)
  return new Promise((resolve) => {
    getPostitionSuccess = resolve
  })
}

onMounted(async () => {
  await getClientPosition()
})
</script>
