<script setup>
import { ref, reactive, computed, watch } from "vue";

// 修改這份 YouBike 即時資訊表，並加上
// 1. 站點名稱搜尋 v
// 2. 目前可用車輛 / 總停車格 的排序功能 v
// 3. 加入分頁功能，每頁 20 筆資料

// 欄位說明:
// https://data.taipei/dataset/detail?id=c6bc8aed-557d-41d5-bfb1-8da24f78f2fb
// sno：站點代號、 sna：場站名稱(中文)、 tot：場站總停車格、
// sbi：場站目前車輛數量、 sarea：場站區域(中文)、 mday：資料更新時間、
// lat：緯度、 lng：經度、 ar：地(中文)、 sareaen：場站區域(英文)、
// snaen：場站名稱(英文)、 aren：地址(英文)、 bemp：空位數量、 act：全站禁用狀態

const uBikeStops = ref([]);
const orignalBikeStops = ref([]);
const currentPage = ref(1);
const isHideStop = ref(false);
const searchData = ref("");
const uniPageCount = ref(20);
const totalPageNumber = computed(() =>
  !searchData.value && !isHideStop.value
    ? Math.ceil(uBikeStops.value.length / uniPageCount.value)
    : Math.ceil(filterSearchStop.value.length / uniPageCount.value)
);
const showPageNumber = computed(() => {
  //如果目前頁數-5 比一小，回傳1~10頁
  if (currentPage.value - 5 < 1) {
    return 0;
  }
  //如果目前頁數-5 大於等於1 且  +5小於等於總頁數，那從目前頁數-5的數字開始推10位做顯示
  if (
    currentPage.value - 5 >= 1 &&
    currentPage.value + 5 <= totalPageNumber.value
  ) {
    return currentPage.value - 6;
  }

  if (currentPage.value + 10 > totalPageNumber.value) {
    return totalPageNumber.value - 10;
  }
});
fetch(
  "https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json"
)
  .then((res) => res.text())
  .then((data) => {
    uBikeStops.value = JSON.parse(data);
    orignalBikeStops.value = JSON.parse(data);
  });

const filterZeroStop = computed(() => {
  return isHideStop.value
    ? uBikeStops.value.filter((data) => data.sbi > 0)
    : uBikeStops.value;
});
const filterSearchStop = computed(() => {
  return filterZeroStop.value.filter((data) =>
    data.sna.includes(searchData.value)
  );
});

const showData = computed(() => {
  if (currentPage.value != totalPageNumber) {
    return filterSearchStop.value.slice(
      (currentPage.value - 1) * uniPageCount.value,
      currentPage.value * uniPageCount.value
    );
  } else {
    return filterSearchStop.value.slice(currentPage.value - 1);
  }
});

//false : ASC  ,true: DESC
const sbiOrder = reactive({
  currentOrder: null,
  sortOrder: ["asc", "desc", null],
});
const totOrder = reactive({
  currentOrder: null,
  sortOrder: ["asc", "desc", null],
});

const sortCondition = (data) => {
  switch (data) {
    case "sbi":
      sbiOrder.currentOrder =
        sbiOrder.sortOrder.indexOf(sbiOrder.currentOrder) !== 2
          ? sbiOrder.sortOrder[
              sbiOrder.sortOrder.indexOf(sbiOrder.currentOrder) + 1
            ]
          : sbiOrder.sortOrder[0];
      sortBy(data, sbiOrder.currentOrder);
      break;
    case "tot":
      totOrder.currentOrder =
        totOrder.sortOrder.indexOf(totOrder.currentOrder) !== 2
          ? totOrder.sortOrder[
              totOrder.sortOrder.indexOf(totOrder.currentOrder) + 1
            ]
          : totOrder.sortOrder[0];
      sortBy(data, totOrder.currentOrder);
  }
};

const sortBy = (data, currentOrder) => {
  switch (currentOrder) {
    case "asc":
      uBikeStops.value = uBikeStops.value.sort((a, b) => a[data] - b[data]);
      break;
    case "desc":
      uBikeStops.value = uBikeStops.value.sort((a, b) => b[data] - a[data]);
      break;
    default:
      uBikeStops.value = [...orignalBikeStops.value];
      break;
  }
};

watch([searchData, isHideStop], () => {
  currentPage.value = 1;
});
watch(sbiOrder, (nVal) => {
  totOrder.currentOrder =
    nVal.currentOrder !== null ? null : totOrder.currentOrder;
});
watch(totOrder, (nVal) => {
  sbiOrder.currentOrder =
    nVal.currentOrder !== null ? null : sbiOrder.currentOrder;
});

const timeFormat = (val) => {
  // 時間格式
  const pattern = /(\d{4})(\d{2})(\d{2})(\d{2})(\d{2})(\d{2})/;
  return val.replace(pattern, "$1/$2/$3 $4:$5:$6");
};
</script>

<template>
  <div class="app">
    <div class="search-condition">
      <label>
        <input type="checkbox" v-model="isHideStop" /> 隱藏無車輛站點
      </label>
      <label> 站點名稱搜尋: <input type="text" v-model="searchData" /> </label>
    </div>

    <table class="table table-striped">
      <thead>
        <tr>
          <th>#</th>
          <th>場站名稱</th>
          <th>場站區域</th>
          <th @click="sortCondition('sbi')">
            目前可用車輛

            <i
              class="fa fa-sort-asc"
              aria-hidden="true"
              v-if="sbiOrder.currentOrder === 'asc'"
            ></i>

            <i
              class="fa fa-sort-desc"
              aria-hidden="true"
              v-if="sbiOrder.currentOrder === 'desc'"
            ></i>
          </th>
          <th @click="sortCondition('tot')">
            總停車格

            <i
              class="fa fa-sort-asc"
              aria-hidden="true"
              v-if="totOrder.currentOrder === 'asc'"
            ></i>
            <i
              class="fa fa-sort-desc"
              aria-hidden="true"
              v-if="totOrder.currentOrder === 'desc'"
            ></i>
          </th>
          <th>資料更新時間</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="s in showData" :key="s.sno">
          <td>{{ s.sno }}</td>
          <td>{{ s.sna }}</td>
          <td>{{ s.sarea }}</td>
          <td>{{ s.sbi }}</td>
          <td>{{ s.tot }}</td>
          <td>{{ timeFormat(s.mday) }}</td>
        </tr>
      </tbody>
    </table>

    <div class="page-number">
      <template v-if="currentPage !== 1">
        <button @click="currentPage = 1">
          <i class="fa fa-chevron-left" aria-hidden="true"></i>
          <i class="fa fa-chevron-left" aria-hidden="true"></i>
        </button>
        <button @click="currentPage -= 1">
          <i class="fa fa-chevron-left" aria-hidden="true"></i>
        </button>
      </template>

      <button
        v-if="totalPageNumber < 10"
        :class="{ 'current-page': currentPage === number }"
        @click="currentPage = number"
        v-for="(number, index) in totalPageNumber"
        :key="index + 'stoplength'"
      >
        {{ number }}
      </button>

      <button
        v-else
        :class="{ 'current-page': currentPage === number + showPageNumber }"
        @click="currentPage = number + showPageNumber"
        v-for="(number, index) in 10"
        :key="index + 'st'"
      >
        {{ number + showPageNumber }}
      </button>

      <template v-if="currentPage !== totalPageNumber">
        <button @click="currentPage += 1">
          <i class="fa fa-chevron-right" aria-hidden="true"></i>
        </button>
        <button @click="currentPage = totalPageNumber">
          <i class="fa fa-chevron-right" aria-hidden="true"></i>
          <i class="fa fa-chevron-right" aria-hidden="true"></i>
        </button>
      </template>
    </div>
  </div>
</template>

<style scoped>
.app {
  display: flex;
  flex-direction: column;
  padding: 1rem;
  height: 100%;
}
label:first-of-type {
  margin-right: 1rem;
}
.page-number {
  display: flex;
  flex-wrap: wrap;
  margin-top: auto;
  justify-content: center;
}

.page-number button {
  width: 3em;
  padding: 0.25rem;
  border: 1px solid #000;
  text-align: center;
  margin-left: 0.5rem;
  margin-bottom: 0.5rem;
  background-color: #fff;
  outline: none;
}

.page-number .current-page {
  background-color: #f2f2f2;
}

.fa-sort-desc,
.fa-sort-asc {
  position: relative;
}
.fa-sort-desc {
  bottom: 3px;
}

.fa-sort-asc {
  top: 3px;
}
</style>
