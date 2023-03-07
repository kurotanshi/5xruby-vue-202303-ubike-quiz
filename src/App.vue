<script setup>
import { ref, computed, watch } from "vue";

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
const searchWord = ref("");
const sortedByAvailable = ref("");
const sortedByTotal = ref("");
const currentPage = ref(1);

fetch(
  "https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json"
)
  .then((res) => res.text())
  .then((data) => {
    uBikeStops.value = JSON.parse(data);
  });

const timeFormat = (val) => {
  // 時間格式
  const pattern = /(\d{4})(\d{2})(\d{2})(\d{2})(\d{2})(\d{2})/;
  return val.replace(pattern, "$1/$2/$3 $4:$5:$6");
};

const filteredStops = computed(() =>
  uBikeStops.value.filter((stop) =>
    stop.sna.toLowerCase().includes(searchWord.value.toLowerCase())
  )
);

const sortedStops = computed(() => {
  if (!sortedByAvailable.value && !sortedByTotal.value) {
    return filteredStops.value;
  } else if (!sortedByAvailable.value && sortedByTotal.value) {
    return sortedByTotal.value === "asc"
      ? [...filteredStops.value].sort((aStop, bStop) => aStop.tot - bStop.tot)
      : [...filteredStops.value].sort((aStop, bStop) => bStop.tot - aStop.tot);
  } else if (sortedByAvailable.value && !sortedByTotal.value) {
    return sortedByAvailable.value === "asc"
      ? [...filteredStops.value].sort((aStop, bStop) => aStop.sbi - bStop.sbi)
      : [...filteredStops.value].sort((aStop, bStop) => bStop.sbi - aStop.sbi);
  }
});

const paginatedStops = computed(() =>
  sortedStops.value.filter(
    (_, index) =>
      index > 20 * (currentPage.value - 1) && index <= 20 * currentPage.value
  )
);

const isDivisible = computed(() => sortedStops.value.length % 20 === 0);

const totalPages = computed(() =>
  isDivisible.value
    ? sortedStops.value.length / 20
    : Math.floor(sortedStops.value.length / 20) + 1
);

const handleSort = (sortType, sortWay) => {
  if (sortType === "availability") {
    sortedByAvailable.value = sortWay;
    sortedByTotal.value = "";
  } else {
    sortedByAvailable.value = "";
    sortedByTotal.value = sortWay;
  }
};

const sortByNumbers = () => {
  console.log(
    "sortedByAvailable:",
    sortedByAvailable.value,
    "sortedByTotal:",
    sortedByTotal.value
  );
  sortedByAvailable.value = "";
  sortedByTotal.value = "";
};

watch(
  () => searchWord.value,
  (newWord) => {
    if (newWord) {
      currentPage.value = 1;
    }
  }
);
</script>

<template>
  <div class="app">
    <div class="upper-section">
      <p>站點名稱搜尋: <input type="text" v-model.trim="searchWord" /></p>
      <button type="button" class="number-sort" @click="sortByNumbers">
        按編號排序
      </button>
    </div>

    <table class="table table-striped">
      <thead>
        <tr>
          <th>#</th>
          <th>場站名稱</th>
          <th>場站區域</th>
          <th>
            目前可用車輛
            <i
              class="fa fa-sort-asc"
              aria-hidden="true"
              @click="handleSort('availability', 'asc')"
              :style="'cursor: pointer'"
            ></i>
            <i
              class="fa fa-sort-desc"
              aria-hidden="true"
              @click="handleSort('availability', 'desc')"
              :style="'cursor: pointer'"
            ></i>
          </th>
          <th>
            總停車格
            <i
              class="fa fa-sort-asc"
              aria-hidden="true"
              @click="handleSort('total', 'asc')"
              :style="'cursor: pointer'"
            ></i>
            <i
              class="fa fa-sort-desc"
              aria-hidden="true"
              @click="handleSort('total', 'desc')"
              :style="'cursor: pointer'"
            ></i>
          </th>
          <th>資料更新時間</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="s in paginatedStops" :key="s.sno">
          <td>{{ s.sno }}</td>
          <td>{{ s.sna }}</td>
          <td>{{ s.sarea }}</td>
          <td>{{ s.sbi }}</td>
          <td>{{ s.tot }}</td>
          <td>{{ timeFormat(s.mday) }}</td>
        </tr>
      </tbody>
    </table>

    <div>
      <ul>
        <li
          v-for="n in totalPages"
          :style="
            currentPage === n ? 'background-color: blue; color: white' : ''
          "
          @click="currentPage = n"
        >
          {{ n }}
        </li>
      </ul>
    </div>
  </div>
</template>

<style scoped>
.app {
  padding: 1rem;
}

.upper-section {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.number-sort {
  background-color: antiquewhite;
  border: 1px solid gold;
  padding: 4px 10px;
  border-radius: 8px;
}

ul {
  display: flex;
  justify-content: start;
  flex-wrap: wrap;
  padding-left: 10px;
}

li {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-right: 6px;
  margin-top: 6px;
  width: 30px;
  height: 30px;
  list-style: none;
  background-color: rgb(127, 197, 255);
  border-radius: 5px;
  cursor: pointer;
}
</style>
