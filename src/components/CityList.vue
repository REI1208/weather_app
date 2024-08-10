<template>
    <div v-for="city in savedCities" :key="city.id">
        <CityCard :city="city" @click="goToCityView(city)" />
    </div>
    <p v-if="savedCities.length === 0">
        列表中暂时没有保存城市，请选择城市进行保存
    </p>
</template>

<script setup>
import axios from 'axios';
import { ref } from 'vue';
import CityCard from './CityCard.vue';
import { useRouter } from 'vue-router';


const savedCities = ref([]);
const getCities = async () => {
    if (localStorage.getItem("savedCities")) {
        savedCities.value = JSON.parse(
            localStorage.getItem("savedCities")
        )
        const requsets = [];
        savedCities.value.forEach((city) => {
            requsets.push(axios.get(`https://api.openweathermap.org/data/2.5/weather?lat=${city.coords.lat}&lon=${city.coords.lng}&appid=7efa332cf48aeb9d2d391a51027f1a71&units=metric`))
        })

        const weatherData = await Promise.all(requsets);

        await new Promise((res) => setTimeout(res, 1000))

        weatherData.forEach((value, index) => {
            savedCities.value[index].weather = value.data
        })
    }
}
await getCities();

const router = useRouter()
const goToCityView = (city) => {
    router.push({
        name: "cityView",
        params: {
            state: city.state, city: city.city
        },
        query: {
            id: city.id,
            lat: city.coords.lat,
            lng: city.coords.lng
        }
    })
}
</script>
