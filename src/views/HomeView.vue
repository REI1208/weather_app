<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input type="text" placeholder="Search for a city or state" v-model="searchQuery" @input="getSearchResults" class="py-2 px-2 w-full bg-transparent border-b 
      focus:border-weather-secondary focus:outline-none
      focus:shadow-[0px_1px_0_0#004E71]">
      <ul class="absolute bg-weather-secondary
       text-white w-full shadow-md py-2 px-1 top-[66px]" v-if="mapboxSearchResults">
        <p v-if="searchError">Something Wrong, please try again</p>
        <p v-if="!searchError && mapboxSearchResults.length === 0">
          No results match,please try different term.
        </p>
        <template v-else>
          <li class="py-2 cursor-pointer" v-for="searchResult in mapboxSearchResults" :key="searchResult.id"
            @click="previewCity(searchResult)">
            {{ searchResult.properties.full_address }}
          </li>
        </template>
      </ul>
    </div>
    <div class="flex flex-col gap-4">
      <Suspense>
        <CityList />
        <template #fallback>
          <CityCardSkeleton />
        </template>
      </Suspense>
    </div>
  </main>
</template>

<script setup>
import CityCardSkeleton from '@/components/CityCardSkeleton.vue';
import CityList from '@/components/CityList.vue';
import axios from 'axios';
import { ref } from 'vue';
import { useRouter } from 'vue-router';
const router = useRouter();
const previewCity = (searchResult) => {
  console.log(searchResult);
  const [city, state] = searchResult.properties.full_address.split(",");
  console.log(city, state);
  router.push({
    name: "cityView",
    params: { state: state.replaceAll(" ", ""), city: city },
    query: {
      lat: searchResult.geometry.coordinates[1],
      lng: searchResult.geometry.coordinates[0],
      preview: true,
    },
  })
}

const searchQuery = ref("")
const queryTimeout = ref(null)
const mapboxAPIKey = 'pk.eyJ1IjoiOTg1OTQ3MzQ4IiwiYSI6ImNsejJwOGxtOTFwem0ycnB4dndldWdjNHYifQ.qA8dgjdYkTjCxAmnLJ206Q';
const mapboxSearchResults = ref(null)
const type = "place"
const searchError = ref(null)
const getSearchResults = () => {
  clearTimeout(queryTimeout.value);
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== '') {
      try {
        const result = await axios.get(
          `https://api.mapbox.com/search/geocode/v6/forward`, {
          params: {
            q: searchQuery.value,
            access_token: mapboxAPIKey,
            types: "place"
          }
        });
        mapboxSearchResults.value = result.data.features;
      } catch {
        searchError.value = true;
      }
      return;
    }
    mapboxSearchResults.value = null;
  }, 300);
}


</script>
