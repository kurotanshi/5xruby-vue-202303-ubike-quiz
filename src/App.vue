<script setup>
import { ref, computed, watch } from 'vue';

// 修改這份 YouBike 即時資訊表，並加上
// 1. 站點名稱搜尋
// 2. 目前可用車輛 / 總停車格 的排序功能
// 3. 加入分頁功能，每頁 20 筆資料

// 欄位說明:
// https://data.taipei/dataset/detail?id=c6bc8aed-557d-41d5-bfb1-8da24f78f2fb
// sno：站點代號、 sna：場站名稱(中文)、 tot：場站總停車格、
// sbi：場站目前車輛數量、 sarea：場站區域(中文)、 mday：資料更新時間、
// lat：緯度、 lng：經度、 ar：地(中文)、 sareaen：場站區域(英文)、
// snaen：場站名稱(英文)、 aren：地址(英文)、 bemp：空位數量、 act：全站禁用狀態

const uBikeStops = ref([]);
fetch('https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json')
  .then(res => res.text())
  .then(data => {
    uBikeStops.value = JSON.parse(data);
  });

const timeFormat = (val) => {
  // 時間格式
  const pattern = /(\d{4})(\d{2})(\d{2})(\d{2})(\d{2})(\d{2})/;
  return val.replace(pattern, '$1/$2/$3 $4:$5:$6');
};

// 站點名稱搜尋
const searchSno = ref('');
const searchSnoResult = computed(() => {
  return uBikeStops.value.filter((item) => {
    return item.sna.includes(searchSno.value);
  });
});

// 可用車輛的排序功能
const sortSareaType = ref('');
const sortedSearchSnoResult = computed(() => {
  // 因為 Vue 計算屬性是唯讀的，直接修改其值可能會導致意外行為
  // 所以這裡先複製一份出來
  const results = searchSnoResult.value.slice(); 
  if (sortSareaType.value === 'asc') {
    return results.sort((a, b) => a.sbi - b.sbi);
  } else {
    return results.sort((a, b) => b.sbi - a.sbi);
  }
});

// 總停車格的排序功能
const sortTotType = ref('');
const sortedSearchTotResult = computed(() => {
  const results = searchSnoResult.value.slice(); 
  if (sortTotType.value === 'asc') {
    return results.sort((a, b) => a.tot - b.tot);
  } else {
    return results.sort((a, b) => b.tot - a.tot);
  }
});

// filter完結果
const filteredResult = computed(() => {
  if (sortSareaType.value) {
    return sortedSearchSnoResult.value;
  } else if (sortTotType.value) {
    return sortedSearchTotResult.value;
  } else {
    return searchSnoResult.value;
  }
});

// 分頁功能
const currentPage = ref(1);
const perPage = ref(20);
const totalPages = computed(() => {
  return Math.ceil(filteredResult.value.length / perPage.value);
});

// 分頁結果
const filteredResultByPage = computed(() => {
  const start = (currentPage.value - 1) * perPage.value;
  const end = start + perPage.value;
  // slice() 方法可從已有的陣列中回傳選定的元素。
  return filteredResult.value.slice(start, end);
});

</script>

<template>
<div class="app">
  <div class="d-flex flex-column mb-3 flex-wrap">
    <h1>台北市 YouBike 即時資訊</h1>
  <p>
    站點名稱搜尋: <input type="text" v-model="searchSno">
  </p>

  <table class="table table-striped">
    <thead>
      <tr>  
        <th>#</th>
        <th>場站名稱</th>
        <th>場站區域</th>
        <th>目前可用車輛
          <i :class="{ 'bi bi-caret-up-fill': sortSareaType === 'asc', 'bi bi-caret-up': sortSareaType === 'desc' || sortSareaType === '' }" 
          aria-hidden="true" @click="{sortSareaType = 'asc'; sortTotType = ''}"></i>
          <i :class="{ 'bi bi-caret-down': sortSareaType === 'asc' || sortSareaType === '', 'bi bi-caret-down-fill': sortSareaType === 'desc'}" 
          aria-hidden="true" @click="{sortSareaType = 'desc'; sortTotType = ''}"></i>
        </th>
        <th>總停車格
          <i :class="{ 'bi bi-caret-up-fill': sortTotType === 'asc', 'bi bi-caret-up': sortTotType === 'desc' || sortTotType === '' }" 
          aria-hidden="true" @click="{sortTotType = 'asc';sortSareaType = '' }"></i>
          <i :class="{ 'bi bi-caret-down': sortTotType === 'asc' || sortTotType === '', 'bi bi-caret-down-fill': sortTotType === 'desc'}" 
          aria-hidden="true" @click="{sortTotType = 'desc';sortSareaType = '' }"></i>
        </th>
        <th>資料更新時間</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="s in filteredResultByPage" :key="s.sno">
        <td>{{ s.sno }}</td>
        <td>{{ s.sna }}</td>
        <td>{{ s.sarea }}</td>
        <td>{{ s.sbi }}</td>
        <td>{{ s.tot }}</td>
        <td>{{ timeFormat(s.mday) }}</td>
      </tr>
    </tbody>
  </table>

  <nav aria-label="...">
    <ul class="pagination justify-content-center flex-wrap">
      <li class="page-item" :class="{'disabled': currentPage === 1}" @click.prevent="currentPage!==1 ? currentPage+=-1 : 1">
        <a class="page-link">Previous</a>
      </li>
      <li class="page-item" v-for="page in totalPages" :key="page" :class="{ 'active': page === currentPage }">
        <a class="page-link" href="#" @click.prevent="currentPage = page">{{ page }}</a></li>
      <li class="page-item" :class="{'disabled': currentPage === totalPages }" @click.prevent="currentPage !== totalPages ? currentPage+=1 : totalPages">
        <a class="page-link" href="#">Next</a>
      </li>
    </ul>
  </nav>
  </div>

</div>
</template>

<style scoped>
.app {
  padding: 1rem;
}
</style>