<template>
  <header class="sticky top-0 bg-weather-primary shadow-lg">
    <nav class="container flex flex-col sm:flex-row items-center gap-4
         text-white py-6">
      <RouterLink :to="{ name: 'home' }">
        <div class="flex items-center gap-3  ">
          <i class="fa-solid fa-sun text-2xl" </i>
            <p class="text-2xl">The Local Weather</p>
        </div>
      </RouterLink>

      <div class="flex gap-3 flex-1 justify-end">
        <i class="fa-solid fa-circle-info text-xl
             hover:text-weather-secondary duration-150 cursor-pointer" @click="toggleModal"></i>
        <i class="fa-solid fa-plus text-xl
             hover:text-weather-secondary duration-150 cursor-pointer" @click="addCity" v-if="route.query.preview">
        </i>
      </div>

      <BaseModel :modalActive="modalActive" @close-modal="toggleModal">
        <div class="text-black">
          <h1 class="text-2xl mb-1">使用说明:</h1>
          <p class="mb-4">
            本地天气允许您跟踪所选城市的当前和未来天气。
          </p>
          <h2 class="text-2xl">如何使用:</h2>
          <ol class="list-decimal list-inside mb-4">
            <li>
              通过在搜索栏中输入名称来搜索您的城市。
            </li>
            <li>
              在结果中选择一个城市，这将带您选择当前的天气。
            </li>
            <li>
              通过单击右上角的“+”图标来跟踪城市。这将保存城市，以便稍后在主页上查看。
            </li>
          </ol>

          <h2 class="text-2xl">移除城市</h2>
          <p>
            如果您不再希望跟踪某个城市，只需在主页中选择该城市即可。在页面底部，将有删除城市的选项。
          </p>
        </div>
      </BaseModel>
    </nav>

  </header>
</template>

<script setup>
import { ref } from "vue"
import { RouterLink, useRoute, useRouter } from 'vue-router';
import BaseModel from './BaseModel.vue';
import { uid } from 'uid'
const route = useRoute();
const router = useRouter()
const savedCities = ref([]);
const addCity = () => {
  if (localStorage.getItem("savedCities")) {
    savedCities.value = JSON.parse(
      localStorage.getItem("savedCities")
    )
  }
  const locationObj = {
    id: uid,
    state: route.params.state,
    city: route.params.city,
    coords: {
      lat: route.query.lat,
      lng: route.query.lng,
    },
  }

  savedCities.value.push(locationObj);
  localStorage.setItem("savedCities", JSON.stringify(savedCities.value))

  let query = Object.assign({}, router.query)
  delete query.preview;
  query.id = locationObj.id;
  router.replace({ query });
}

const modalActive = ref(null);
const toggleModal = () => {
  modalActive.value = !modalActive.value
}
</script>
