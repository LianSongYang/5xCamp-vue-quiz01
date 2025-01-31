<script setup>
import { ref,  computed, watch} from 'vue';
const uBikeStops = ref([]);
const countPerPage = ref(700);
const search = ref('');
const searchResult = computed(() => {
  return uBikeStops.value.filter(s => s.sna.includes(search.value));
});


const sortBy = ref('tot');
const isSortAsc = ref(true);
const sortedData = computed(() => {
  return searchResult.value.sort((a, b) => {
    return isSortAsc.value ? a[sortBy.value] - b[sortBy.value] : b[sortBy.value] - a[sortBy.value];
  });
});
const setSort = (sortby) => {
  if (sortby === sortBy.value) {
    isSortAsc.value = !isSortAsc.value;
  } else {
    sortBy.value = sortby;
    isSortAsc.value = true;
  }
}
const isLoading = ref(false);

const totalCounts = 684;
const currentPage = ref(1);
const totalPages = computed(() => {
  return Math.ceil(totalCounts / countPerPage.value);
});
watch(countPerPage, () => {
  currentPage.value = 1;
});
watch([countPerPage, currentPage], fetchData, { immediate: true });

// 資料來源: https://data.ntpc.gov.tw/openapi/swagger-ui/index.html?configUrl=%2Fapi%2Fv1%2Fopenapi%2Fswagger%2Fconfig&urls.primaryName=%E6%96%B0%E5%8C%97%E5%B8%82%E6%94%BF%E5%BA%9C%E4%BA%A4%E9%80%9A%E5%B1%80(94)#/JSON/get_010e5b15_3823_4b20_b401_b1cf000550c5_json

// 欄位說明: 
// sno：站點代號、 sna：場站名稱(中文)、 total：場站總停車格、
// available_rent_bikes：場站目前可用車輛數量、 
// sarea：場站區域(中文)、 mday：資料更新時間、
// lat：緯度、 lng：經度、 ar：地(中文)、 sareaen：場站區域(英文)、
// snaen：場站名稱(英文)、 aren：地址(英文)、 bemp：空位數量、 act：全站禁用狀態

// page: 頁碼, size: 每頁筆數, 全部 349 筆.
function fetchData() {
  isLoading.value = true;

  fetch(`/api/datasets/010e5b15-3823-4b20-b401-b1cf000550c5/json?page=${currentPage.value}&size=${countPerPage.value}`)
  .then(res => res.text())
  .then(data => {
    uBikeStops.value = JSON.parse(data);
    isLoading.value = false;
  });
}

const timeFormat = (val) => {
  // 時間格式
  const pattern = /(\d{4})(\d{2})(\d{2})(\d{2})(\d{2})(\d{2})/;
  return val.replace(pattern, '$1/$2/$3 $4:$5:$6');
};
</script>

<template>  
<!--
練習： YouBike 新北市公共自行車即時資訊，實作以下功能
  1. 站點名稱搜尋
  2. 目前可用車輛 / 總停車格 的排序功能
  3. 分頁功能, 已知總筆數為 349 筆
-->

  <div class="">
    <div class="grid grid-cols-2 my-4 px-4 w-full mx-auto">
      <div class="pl-2">
        目前頁面的站點名稱搜尋: <input v-model="search"  type="text" class="border w-60 p-1 ml-2">
      </div>
      <div class="pl-2">
        每頁顯示筆數: 
        <select class="border w-20 p-1 ml-2" v-model="countPerPage">
          <option value="10">10</option>
          <option value="30">30</option>
          <option value="50">50</option>
          <option value="500">500</option>
          <option value="700">700</option>
        </select>
      </div>
    </div>


    <div v-if="isLoading">
      Loading...
    </div>
    <table v-else class="table table-striped">
      <thead>
        <tr>
          <th class="w-12">#</th>
          <th>場站名稱</th>
          <th>場站區域</th>
          <th @click="setSort('sbi')">
            目前可用車輛
            <template v-if="sortBy === 'sbi'">
              <i v-if="isSortAsc" class="fa fa-sort-asc" aria-hidden="true"></i>
              <i v-else class="fa fa-sort-desc" aria-hidden="true"></i>
            </template>
          </th>
          <th
            @click="setSort('tot')">
              總停車格
            <template v-if="sortBy === 'tot'">
              <i v-if="isSortAsc" class="fa fa-sort-asc" aria-hidden="true"></i>
              <i v-else class="fa fa-sort-desc" aria-hidden="true"></i>
            </template>

          </th>
          <th>資料更新時間</th>          
        </tr>
      </thead>
      <tbody>
        <tr v-for="(s, idx) in sortedData" :key="s.sno">
          <td>{{ idx +1 }}</td>
          <td>{{ s.sna }}</td>
          <td>{{ s.sarea }}</td>
          <td>{{ s.sbi }}</td>
          <td>{{ s.tot }}</td>
          <td>{{ timeFormat(s.mday) }}</td>
        </tr>
      </tbody>
    </table>
    
    <!-- 頁籤 -->
    <ul class="my-4 flex justify-center">
      <li 
        @click="currentPage = 1"
        class="page-item cursor-pointer">
        <span class="page-link">第一頁</span>
      </li>
      <li 
        @click="currentPage = (currentPage > 1) ? currentPage - 1 : 1"
        class="page-item cursor-pointer">
        <span class="page-link">&lt;</span>
      </li>

      <li 
        v-for="page in totalPages"
        @click="currentPage = page"
        :class="{ 'active': page === currentPage }"
        class="page-item cursor-pointer">
        <span class="page-link">{{ page }}</span>
      </li>

      <li 
        @click="currentPage = (currentPage < totalPages) ? currentPage + 1 : totalPages"
        class="page-item cursor-pointer">
        <span class="page-link" href>&gt;</span>
      </li>      
      <li 
        @click="currentPage = totalPages"
        class="page-item cursor-pointer">
        <span class="page-link">最末頁</span>
      </li>
    </ul>

  </div>
</template>
