<script setup>
import { ref, computed, watch } from "vue";

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
  //如果總頁數 <10 ，回傳 1~總頁數 的陣列
  if (totalPageNumber <= 10) {
    return Array.from(
      { length: totalPageNumber.value },
      (_, i) => totalPageNumber.value - i
    ).sort();
  }
  //如果目前頁數-5 大於等於1 且  +5小於等於總頁數，那從目前頁數-5的數字開始推10位做顯示
  if (
    currentPage.value - 5 >= 1 &&
    currentPage.value + 5 <= totalPageNumber.value
  ) {
    return Array.from({ length: 10 }, (_, i) => currentPage.value - 5 + i);
  }
  //如果目前頁數-5 比一小，回傳1~10頁
  if (currentPage.value - 5 < 1) {
    return Array.from({ length: 10 }, (_, i) => 1 + i);
  }
  //都不是的話，回傳最後一頁往前推10頁，然後排序一下
  return Array.from({ length: 10 }, (_, i) => totalPageNumber.value - i).sort();
});
fetch(
  "https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json"
)
  .then((res) => res.text())
  .then((data) => {
    uBikeStops.value = JSON.parse(data);
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
const sbistatus = ref(false);
const totstatus = ref(false);
const sortCondition = (data) => {
  switch (data) {
    case "sbi":
      sortBy(data, sbistatus.value);
      sbistatus.value = !sbistatus.value;
      break;
    case "tot":
      sortBy(data, totstatus.value);
      totstatus.value = !totstatus.value;
  }
};

const sortBy = (data, sb) => {
  if (!sb) {
    uBikeStops.value = uBikeStops.value.sort((a, b) => a[data] - b[data]);
  } else {
    uBikeStops.value = uBikeStops.value.sort((a, b) => b[data] - a[data]);
  }
};

watch([searchData, isHideStop], () => {
  currentPage.value = 1;
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

            <i class="fa fa-sort-asc" aria-hidden="true"></i>
            <i class="fa fa-sort-desc" aria-hidden="true"></i>
          </th>
          <th @click="sortCondition('tot')">
            總停車格

            <i class="fa fa-sort-asc" aria-hidden="true"></i>
            <i class="fa fa-sort-desc" aria-hidden="true"></i>
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
        :class="{ 'current-page': currentPage === number }"
        @click="currentPage = number"
        v-for="(number, index) in showPageNumber"
        :key="index + 'stoplength'"
      >
        {{ number }}
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
</style>
