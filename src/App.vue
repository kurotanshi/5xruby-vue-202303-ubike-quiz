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
const currentSort = ref("");
const isOrderAsc = ref(true);
const currentPage = ref(1);
const headingPage = ref(null);

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
  if (!currentSort.value) {
    return filteredStops.value;
  } else {
    return isOrderAsc.value
      ? [...filteredStops.value].sort(
          (aStop, bStop) => aStop[currentSort.value] - bStop[currentSort.value]
        )
      : [...filteredStops.value].sort(
          (aStop, bStop) => bStop[currentSort.value] - aStop[currentSort.value]
        );
  }
});

const paginatedStops = computed(() =>
  sortedStops.value.filter(
    (_, index) =>
      index > 20 * (currentPage.value - 1) && index <= 20 * currentPage.value
  )
);

const totalPages = computed(() => Math.ceil(sortedStops.value.length / 20));

const handleSort = (sortType, sortWay) => {
  if (sortType === "availability") {
    currentSort.value = "sbi";
  } else {
    currentSort.value = "tot";
  }
  isOrderAsc.value = sortWay === "asc";
};

const sortByNumbers = () => {
  currentSort.value = "sno";
  isOrderAsc.value = true;
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
      <div class="order-section">
        <p v-if="currentSort === 'sbi'" class="sort-hint">
          目前按「可用車輛」數{{ isOrderAsc ? "升冪" : "降冪" }}排序
        </p>
        <p v-if="currentSort === 'tot'" class="sort-hint">
          目前按「總停車格」數{{ isOrderAsc ? "升冪" : "降冪" }}排序
        </p>
        <button
          v-if="currentSort === 'sbi' || currentSort === 'tot'"
          type="button"
          class="number-sort"
          @click="sortByNumbers"
        >
          按編號排序
        </button>
      </div>
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

    <div v-if="totalPages >= 1" class="page-section">
      <div
        class="page-control more-page"
        v-if="currentPage > 10 && totalPages >= 11"
        @click="currentPage -= 10"
      >
        前十頁
      </div>
      <div
        class="page-control"
        v-if="currentPage > 1 && totalPages >= 1"
        @click="currentPage -= 1"
      >
        上一頁
      </div>
      <ul>
        <template v-for="(_, index) in 9">
          <li
            v-if="currentPage + index <= totalPages"
            :style="
              currentPage === currentPage + index
                ? 'background-color: blue; color: white'
                : ''
            "
            @click="currentPage = currentPage + index"
          >
            {{ currentPage + index }}
          </li>
        </template>
      </ul>
      <div
        class="page-control"
        v-if="totalPages > currentPage"
        @click="currentPage += 1"
      >
        下一頁
      </div>
      <div
        class="page-control more-page"
        v-if="totalPages > currentPage + 10"
        @click="currentPage += 10"
      >
        後十頁
      </div>
    </div>
    <div v-if="totalPages > 9" class="total-page">
      <p>共 {{ totalPages }} 頁</p>
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

.order-section {
  display: flex;
  align-items: center;
  margin-bottom: 16px;
}

.sort-hint {
  display: block;
  margin: 0;
  margin-right: 12px;
  font-weight: 700;
}

.number-sort {
  background-color: antiquewhite;
  border: 1px solid gold;
  padding: 4px 10px;
  border-radius: 8px;
}

.page-section {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: 10px;
}

ul {
  display: flex;
  justify-content: start;
  flex-wrap: wrap;
  padding-left: 10px;
  margin: 0 10px;
}

li {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-right: 6px;
  width: 30px;
  height: 30px;
  list-style: none;
  background-color: rgb(127, 197, 255);
  border-radius: 5px;
  cursor: pointer;
}

.page-control {
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  padding: 0 10px;
  height: 30px;
}

.page-control:hover {
  padding: 8px 10px;
  border-radius: 8px;
  background-color: rgb(174, 214, 247);
  color: white;
}

.more-page:hover {
  background-color: rgb(84, 87, 235);
  color: white;
}

.total-page {
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 1rem 0;
}
</style>
