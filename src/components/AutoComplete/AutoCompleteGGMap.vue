<template>
  <input type="text" class="form-control" ref="inputAutoComplate" />
</template>
<script setup lang="ts">
import type { AutoCompleteGGMap } from './AutoComplete'
import { Loader } from '@googlemaps/js-api-loader'
import { onMounted, ref } from 'vue'

const emit = defineEmits<{ (e: 'selectAutoComplate', data: AutoCompleteGGMap): void }>()

const apiKey = import.meta.env.VITE_GG_MAP_API_KEY

const lat = ref()
const lng = ref()
const placeId = ref()
const location = ref()
const inputAutoComplate = ref()
async function setUpAutoComplate() {
  const { Autocomplete } = (await google.maps.importLibrary('places')) as google.maps.PlacesLibrary
  const center = { lat: 50.064192, lng: -130.605469 }

  // set option
  const defaultBounds = {
    north: center.lat + 0.1,
    south: center.lat - 0.1,
    east: center.lng + 0.1,
    west: center.lng - 0.1
  }
  const options = {
    bounds: defaultBounds,
    componentRestrictions: { country: 'th' },
    fields: ['address_components', 'geometry', 'icon', 'name'],
    strictBounds: false,
    types: ['establishment']
  }

  // นำ input element เเละ option ให้ gg map autocomplate ไป สร้าง auto complate
  const autoComplate = new Autocomplete(inputAutoComplate.value, options)

  // เพิ่ม event เมื่อกดเลือกใน autocomplate
  autoComplate.addListener('place_changed', () => {
    const _location = autoComplate.getPlace().geometry?.location
    const latlng = { lat: _location?.lat(), lng: _location?.lng() }
    location.value = inputAutoComplate.value.value
    lat.value = latlng.lat
    lng.value = latlng.lng
    const dec = new google.maps.Geocoder()
    dec.geocode({ location: { lat: lat.value, lng: lng.value } }, (res: any, status) => {
      placeId.value = res[0].place_id
      emit('selectAutoComplate', {
        lat: lat.value,
        lng: lng.value,
        location: location.value,
        placeId: placeId.value
      })
    })
  })
}
onMounted(() => {
  const loader = new Loader({
    apiKey: apiKey,
    language: 'th'
  })
  loader.load().then(async () => {
    await setUpAutoComplate()
  })
})
</script>
<style >
.pac-container {
  border:2px solid #E3E7F0;
  border-radius: 10px;
  padding:12px;
}
.pac-container.pac-logo::after {
display: none;
}
.pac-item {
  border-radius: 8px;
  border:0px solid black;
  padding:12px 16px;
  cursor: pointer;
}
.pac-icon{
  margin-right: 20px;
}
</style>
