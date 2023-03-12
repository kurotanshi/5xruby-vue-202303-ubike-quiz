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

// 目前的排序選項
const currentSort = ref('sno');
// 是否為降冪排序
const isSortDesc = ref(false);

// 所有站點資料
const uBikeStops = ref([]);
// 搜尋文字
const searchText = ref('');

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

// 監聽搜尋文字，若有變動則將頁碼歸 1
watch(searchText, () => {
  currentPage.value = 1;
});

// 篩選後的站點資料
const filtedUbikeStops = computed(() => {
  return uBikeStops.value.length === 0
    ? []
    : uBikeStops.value.filter(d => d.sna.includes(searchText.value));
});

// 排序後的站點資料
const sortedUbikeStops = computed(() => {
  const filtedStops = [...filtedUbikeStops.value];

  return isSortDesc.value
    ? filtedStops.sort((a, b) => b[currentSort.value] - a[currentSort.value])
    : filtedStops.sort((a, b) => a[currentSort.value] - b[currentSort.value]);
});

// 目前頁碼
const currentPage = ref(1);
// 一頁幾筆資料
const COUNT_OF_PAGE = 10;
// 最多顯示幾頁
const PAGINATION_MAX = 10;

// 分頁後的站點資料
const slicedUbikeStops = computed(() => {
  const start = (currentPage.value - 1) * COUNT_OF_PAGE;
  const end =
    start + COUNT_OF_PAGE <= sortedUbikeStops.value.length
      ? start + COUNT_OF_PAGE
      : sortedUbikeStops.value.length;
  return sortedUbikeStops.value.slice(start, end);
});

// 總頁數
const totalPageCount = computed(() => {
  return Math.ceil(filtedUbikeStops.value.length / COUNT_OF_PAGE);
});

// 分頁的尾端
const pagerEnd = computed(() => {
  return totalPageCount.value <= PAGINATION_MAX
    ? totalPageCount.value
    : PAGINATION_MAX;
});

// 分頁的位移，用來確保目前的頁碼在中間
const pagerAddAmount = computed(() => {
  const tmp =
    totalPageCount.value <= PAGINATION_MAX
      ? 0
      : currentPage.value + 4 - pagerEnd.value;
  return tmp <= 0
    ? 0
    : totalPageCount.value - (PAGINATION_MAX + tmp) < 0
      ? totalPageCount.value - PAGINATION_MAX
      : tmp;
});

// 換頁
const setPage = page => {
  if (page < 1 || page > totalPageCount.value) {
    return;
  }
  currentPage.value = page;
};

// 指定排序
const setSort = sortType => {
  if (sortType === currentSort.value) {
    isSortDesc.value = !isSortDesc.value;
  } else {
    currentSort.value = sortType;
    isSortDesc.value = false;
  }
};

// 關鍵字 Highlight
const keywordsHighlight = (text, keyword) => {
  if(keyword === '') return text;
  const reg = new RegExp(keyword, 'gi');
  return text.replace(reg, `<span style="color: red;">${keyword}</span>`);
};
</script>

<template>
  <div class="app">
    <p>
      站點名稱搜尋: <input type="text" v-model="searchText">
    </p>

    <table class="table table-striped">
      <thead>
        <tr>
          <th @click="setSort('sno')">
            #
            <span v-show="currentSort === 'sno'">
              <i class="fa" :class="isSortDesc ? 'fa-sort-desc' : 'fa-sort-asc'" aria-hidden="true"></i>
            </span>
          </th>
          <th>
            場站名稱
          </th>
          <th>
            場站區域
          </th>
          <th @click="setSort('sbi')" class="pointer">
            目前可用車輛
            <span v-show="currentSort === 'sbi'">
              <i class="fa" :class="isSortDesc ? 'fa-sort-desc' : 'fa-sort-asc'" aria-hidden="true"></i>
            </span>
          </th>
          <th @click="setSort('tot')" class="pointer">
            總停車格
            <span v-show="currentSort === 'tot'">
              <i class="fa" :class="isSortDesc ? 'fa-sort-desc' : 'fa-sort-asc'" aria-hidden="true"></i>
            </span>
          </th>
          <th>
            資料更新時間
          </th>
        </tr>
      </thead>
      <tbody>
        <!-- 替換成 slicedUbikeStops -->
        <tr v-for="s in slicedUbikeStops" :key="s.sno">
          <td>{{ s.sno }}</td>
          <!-- <td>{{ keywordsHighlight(s.sna, searchText) }}</td> -->
          <td v-html="keywordsHighlight(s.sna, searchText)"></td>
          <td>{{ s.sarea }}</td>
          <td>{{ s.sbi }}</td>
          <td>{{ s.tot }}</td>
          <td>{{ (s.mday) }}</td>
        </tr>
      </tbody>
    </table>
  </div>

  <!-- 頁籤 -->
  <nav v-if="pagerEnd > 0">
    <ul class="pagination">
      <li @click.prevent="setPage(currentPage - 1)" class="page-item">
        <a class="page-link" href>Previous</a>
      </li>

      <li v-for="i in pagerEnd" :class="{ active: i + pagerAddAmount === currentPage }" :key="i"
        @click.prevent="setPage(i + pagerAddAmount)" class="page-item">
        <a class="page-link" href>{{ i + pagerAddAmount }}</a>
      </li>

      <li @click.prevent="setPage(currentPage + 1)" class="page-item">
        <a class="page-link" href>Next</a>
      </li>
    </ul>
  </nav>
</template>

<style lang="scss" scoped>
.app {
  padding: 1rem;
}

.pointer {
  cursor: pointer;
}

.pagination {
  display: flex;
  justify-content: center;
}

@media (max-width: 768px) {
  .sno {
    max-width: 50px; word-wrap: break-word;
  }
  .table td, .table th {
    padding: .5rem .25rem;
  }
}
</style>
