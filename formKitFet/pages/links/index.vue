<script setup lang="ts">
import axios from "axios";
import { Link } from "~~/types";
import { ref, watch } from 'vue';
import { TailwindPagination } from "laravel-vue-pagination";
import { useRoute, useRouter } from "vue-router";

const route = useRoute();
const router = useRouter();

definePageMeta({
  middleware: ["auth"],
});

const links = ref<Link[]>([]);
const data = ref()

const queries = ref({
  page: Number(route.query.page) || 1,
  "filter[full_link]": route.query["filter[full_link]"] || "",
  ...route.query
});

async function getLinks() {
  try {
    //@ts-expect-error
    const qs = new URLSearchParams(queries.value).toString();
    const { data: res } = await axios.get(`/links?${qs}`);
    
    data.value = res;
    links.value = res.data;
    
    queries.value.page = res.meta.current_page;

  } catch (error) {
    console.error("Error carregant enllaços:", error);
  }
}

watch(queries, async () => {
  await router.push({ query: queries.value }); 
  getLinks();
}, { deep: true }); 

watch(
  () => route.query,
  (newQuery) => {
    queries.value = { 
      page: Number(newQuery.page) || 1,
      "filter[full_link]": newQuery["filter[full_link]"] || "",
      ...newQuery 
    };
  }
);


onMounted(() => {
  getLinks()
})

</script>
<template>
  <div>
    <nav class="flex justify-between mb-4 items-center">
      <h1 class="mb-0">My Links</h1>
      <div class="flex items-center">
        <SearchInput 
        v-model="queries['filter[full_link]']" placeholder="Search my URL..." />
        <NuxtLink to="/links/create" class="ml-4">
          <IconPlusCircle class="inline" /> Create New
        </NuxtLink>
      </div>
    </nav>

    <div v-if="links.length > 0">
      <table class="table-fixed w-full">
        <thead>
          <tr>
            <th class="w-[35%]">Full Link</th>
            <th class="w-[35%]">Short Link</th>
            <th class="w-[10%]">Views</th>
            <th class="w-[10%]">Edit</th>
            <th class="w-[10%]">Trash</th>
            <th class="w-[6%] text-center">
              <button>
                <IconRefresh />
              </button>
            </th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="link in links" :key="link.short_link">
            <td>
              <a :href="link.full_link" target="_blank">
                {{ link.full_link.replace(/^http(s?):\/\//, "") }}</a>
            </td>
            <td>
              <a :href="`${useRuntimeConfig().public.appURL}/${link.short_link}`" target="_blank">
                {{
                  useRuntimeConfig().public.appURL.replace(
                    /^http(s?):\/\//,
                    ""
                  )
                }}/{{ link.short_link }}
              </a>
            </td>
            <td>{{ link.views }}</td>
            <td>
              <NuxtLink class="no-underline" :to="`/links/${link.short_link}`">
                <iconEdit />
              </NuxtLink>
            </td>
            <td>
              <button>
                <IconTrash />
              </button>
            </td>
            <td></td>
          </tr>
        </tbody>
      </table>
      <TailwindPagination :data="data" @pagination-change-page="queries.page = $event"/>
      <div class="mt-5 flex justify-center gap-2">

      </div>
    </div>

    <!-- No links message for when table is empty -->
    <div v-else class="border-dashed border-gray-500 p-3 border-[1px] text-center">
      <div class="flex justify-center">
        <IconLink />
      </div>
      <p>
        <!-- Show this if reason for no links is none found in search -->
        <span v-if="false"> No links matching links found. </span>

        <!-- Show this if reason for no links is User has none -->
        <span v-else>
          No links created yet
          <NuxtLink to="/links/create" class="text-green-600">Go create your first link!</NuxtLink>
        </span>
      </p>
    </div>
  </div>
</template>
