<script setup>
import { ref } from "vue";
import axios from "axios";
import { useRouter } from "vue-router";


const router = useRouter();
const previewCity = (searchResult) => {
  console.log(searchResult);
  const [city, state] = searchResult.place_name.split(",");
  router.push({
    name: "cityView",
    params: { state: state.replaceAll(" ", ""), city: city },
    query: {
      lat: searchResult.geometry.coordinates[1],
      lng: searchResult.geometry.coordinates[0],
      preview: true,
    },
  });
};

const mapboxAPIKEY = import.meta.env.VITE_APP_API_URL;
const searchQuery = ref("");
const queryTimeout = ref(null);
const mapboxSearchResult = ref(null);
const searchError = ref(null);

const getSearchResults = () => {
  clearTimeout(queryTimeout.value);
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== "") {
      try {
        const result = await axios.get(
          `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxAPIKEY}&types=place`
        );
        mapboxSearchResult.value = result.data.features;
      } catch {
        searchError.value = true;
      }

      return;
    }
    mapboxSearchResult.value = null;
  }, 300);
};
</script>

<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
        type="text"
        v-model="searchQuery"
        @input="getSearchResults"
        placeholder="Search for city or state"
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]"
      />
      <ul
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
        v-if="mapboxSearchResult"
      >
        <p v-if="searchError">Sorry, something went wrong. Please try again.</p>
        <p v-if="!searchError && mapboxSearchResult.length === 0">
          No results match your query, please try a different term.
        </p>
        <template v-else>
          <li
            v-for="searchResult in mapboxSearchResult"
            :key="searchResult.id"
            class="py-2 cursor-pointer"
            @click="previewCity(searchResult)"
          >
            {{ searchResult.place_name }}
          </li>
        </template>
      </ul>
    </div>
  </main>
</template>
