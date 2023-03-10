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

// 目前的排序欄位 ('sbi', 'tot')
const currentSort = ref('');
// 目前的排序方向 ('', 'asc', 'desc')
const currentSortDir = ref('');

// 排序功能
const sortBy = (key) => {
  if (currentSort.value === key) {
    currentSortDir.value = currentSortDir.value === 'asc' ? 'desc' : 'asc';
  } else {
    currentSort.value = key;
    currentSortDir.value = 'asc';
  }
};

// 排序樣式狀態
const sortIcon = (key) => {
  if (currentSort.value === key) {
    return currentSortDir.value === 'asc' ? 'bi-caret-up-fill' : 'bi-caret-down-fill';
  } else {
    return 'bi-caret-up';
  }
};

// sort完結果
const sortedResult = computed(() => {
  const results = searchSnoResult.value.slice(); 

  // 如果有設定排序欄位，就依照欄位排序，並判斷 asc or desc
  if (currentSort.value) {
    results.sort((a, b) => {
      if (a[currentSort.value] > b[currentSort.value]) {
        return currentSortDir.value === 'asc' ? 1 : -1;
      } else if (a[currentSort.value] < b[currentSort.value]) {
        return currentSortDir.value === 'asc' ? -1 : 1;
      } else {
        return 0;
      }
    });
  }

    // 回傳排序後的結果
    return results;
});

// 分頁功能
const currentPage = ref(1);
const perPage = ref(20);
const totalPages = computed(() => {
  return Math.ceil(sortedResult.value.length / perPage.value);
});

// 當 searchSno 改變時，currentPage 也要重置為 1，並且清除排序
watch(searchSnoResult, () =>  {
  currentPage.value = 1;
  currentSort.value = '';
  currentSortDir.value = '';
});

// 分頁結果
const sortedResultByPage = computed(() => {
  const start = (currentPage.value - 1) * perPage.value;
  const end = start + perPage.value;
  // slice() 方法可從已有的陣列中回傳選定的元素。
  return sortedResult.value.slice(start, end);
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
          <i :class="sortIcon('sbi')" aria-hidden="true" @click="sortBy('sbi')"></i>
        </th>
        <th>總停車格
          <i :class="sortIcon('tot')" aria-hidden="true" @click="sortBy('tot')"></i>
        </th>
        <th>資料更新時間</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="s in sortedResultByPage" :key="s.sno">
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