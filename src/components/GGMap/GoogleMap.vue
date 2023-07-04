<template>
  <div v-if="showMap">
    <div
      ref="map"
      style="height: 100%; border-radius: 20px; height: 200px; border: 1px solid black"
    />
  </div>
  lat:{{ lat }} &emsp; lng:{{ lng }}
</template>
<script setup lang="ts">
import { computed, onMounted, ref, watch } from 'vue'
import { Loader } from '@googlemaps/js-api-loader'
import type { GGMap } from './ggmap'
const props = defineProps<{
  dataGGMap: {
    lat: number
    lng: number
  }
}>()
const emit = defineEmits<{ (e: 'update:dataGGMap', data: GGMap): void }>()
const apiKey = import.meta.env.VITE_GG_MAP_API_KEY

const lat = ref(props.dataGGMap.lat)
const lng = ref(props.dataGGMap.lng)
const map = ref()
const markerTitle = ref()
const marker = ref()
const mapObj = ref()

const showMap = computed(
  () =>
    !(
      props.dataGGMap.lat === undefined ||
      props.dataGGMap.lat === null ||
      props.dataGGMap.lng === undefined ||
      props.dataGGMap.lng === null
    )
)

async function setupMap() {
  const dec = new google.maps.Geocoder()
  const latlng = { lat: lat.value, lng: lng.value }
  await dec.geocode({ location: latlng }, async (res: any, status) => {
    if (status !== 'ZERO_RESULTS') {
      markerTitle.value = await res[0].formatted_address
    }
  })
  const { Map } = (await google.maps.importLibrary('maps')) as google.maps.MapsLibrary
  const mapOptions: google.maps.MapOptions = {
    center: latlng,
    zoom: 12,
    fullscreenControl: false,
    mapTypeControl: false,
    streetViewControl: false,
    zoomControl: false,
    keyboardShortcuts: false,
    gestureHandling: 'greedy'
  }
  const maps = new Map(map.value, mapOptions)
  const markerOptions = {
    position: latlng,
    draggable: true,
    map: maps,
    title: markerTitle.value
  }

  marker.value = await new google.maps.Marker(markerOptions)

  maps.addListener('click', (e: any) => {
    marker.value.setPosition(e.latLng)
    emit('update:dataGGMap', { lat: e.latLng.lat(), lng: e.latLng.lng() })
    lat.value = e.latLng.lat()
    lng.value = e.latLng.lng()
  })
  mapObj.value = maps
  marker.value.addListener('dragend', (e: any) => {
    const position = marker.value.getPosition()!
    emit('update:dataGGMap', { lat: position.lat(), lng: position.lng() })

    lat.value = position.lat()
    lng.value = position.lng()
    const dec = new google.maps.Geocoder()
    dec.geocode({ location: position }, (res: any, status) => {
      markerTitle.value = res[0].formatted_address
    })
  })
}

watch(
  () => props.dataGGMap,
  () => {
    marker.value.setPosition({ lat: props.dataGGMap.lat, lng: props.dataGGMap.lng })
    mapObj.value.setCenter({ lat: props.dataGGMap.lat, lng: props.dataGGMap.lng })
    lat.value = props.dataGGMap.lat
    lng.value = props.dataGGMap.lng
  },
  { deep: true }
)

onMounted(() => {
  const loader = new Loader({
    apiKey: apiKey,
    language: 'th'
  })
  loader.load().then(async () => {
    await setupMap()
  })
})
</script>
