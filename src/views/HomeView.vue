<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
        @input="getSearchResults"
        v-model="searchQuery"
        type="text"
        placeholder="Search for a city or state"
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]" />
      <ul
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
        v-if="mapboxSearchResults">
        <p v-if="searchError">Sorry, something went wrong, please try again.</p>
        <p v-if="!searchError && mapboxSearchResults.length === 0">
          No results match your query, try a different term.
        </p>
        <template v-else>
          <li
            v-for="searchResult in mapboxSearchResults"
            @click="previewCity(searchResult)"
            :key="searchResult.id"
            class="py-2 cursor-pointer">
            {{ searchResult.place_name }}
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
import { ref } from "vue";
import axios from "axios";
import { useRouter } from "vue-router";
import CityList from "@/components/CityList.vue";
import CityCardSkeleton from "@/components/CityCardSkeleton.vue";

const router = useRouter();

const mapboxAPIKey =
  "pk.eyJ1Ijoiam9obmtvbWFybmlja2kiLCJhIjoiY2t5NjFzODZvMHJkaDJ1bWx6OGVieGxreSJ9.IpojdT3U3NENknF6_WhR2Q";

const searchQuery = ref("");
const queryTimeout = ref(null);
const mapboxSearchResults = ref(null);
const searchError = ref(null);

const getSearchResults = () => {
  queryTimeout.value = setTimeout(async () => {
    clearTimeout(queryTimeout.value);
    if (searchQuery.value !== "") {
      try {
        const result = await axios.get(
          `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxAPIKey}&types=place`
        );
        mapboxSearchResults.value = result.data.features;
        console.log(mapboxSearchResults.value);
      } catch {
        searchError.value = true;
      }
      return;
    }
    mapboxSearchResults.value = null;
  }, 300);
};

const previewCity = (searchResult) => {
  // console.log(searchResult)
  const [city, state] = searchResult.place_name.split(",");
  // console.log(city, state);
  let exist = false;
  let id = null;
  if (localStorage.getItem("savedCities")) {
    const cities = JSON.parse(localStorage.getItem("savedCities"));
    cities.forEach((item) => {
      if (item.city === city && item.state === state) {
        exist = true;
        id = item.id
      }
    });
  }
  let query = {
    lat: searchResult.geometry.coordinates[1],
    lng: searchResult.geometry.coordinates[0],
  };
  if (!exist) {
    query.preview = true;
  } else {
    query.id = id;
  }
  console.log(query)
  router.push({
    name: "cityView",
    params: { state: state, city: city },
    query: query,
  });
};
</script>
