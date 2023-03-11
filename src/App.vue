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

const bicycle = ref('');
const pageActive = ref(1);
const listPages = ref('');
const Site = computed(() => {

  const rangeRow = uBikeStops.value.filter(all => all.sna.includes(bicycle.value));
    if(listPages.value == 'sbi'){
      rangeRow.sort((a, b) => a.sbi - b.sbi);
    }
    else if(listPages.value == 'tot'){
      rangeRow.sort((a, b) => a.tot - b.tot);
    }
    return rangeRow.slice((pageActive.value-1)*20,(pageActive.value*20));

  return uBikeStops.value.filter(all => all.sna.includes(bicycle.value)).slice((pageActive.value-1)*20,(pageActive.value*20));

}); 
//const Site = computed(() => uBikeStops.value.filter(all => all.sna.includes(bicycle.value))); 

const upDeta = (val) => {listPages.value=val}

const pageChange = (val) => {pageActive.value++};
const pagePrecious = (val) => {pageActive.value--};

// watch(() => console.log(pageActive.value));


const timeFormat = (val) => {
  // 時間格式
  const pattern = /(\d{4})(\d{2})(\d{2})(\d{2})(\d{2})(\d{2})/;
  return val.replace(pattern, '$1/$2/$3 $4:$5:$6');
};
</script>

<template>
<div class="app">
  <p>
    站點名稱搜尋: <input v-model="bicycle" type="text">
  </p>

  <table class="table table-striped">
    <thead>
      <tr>
        <th>#</th>
        <th>場站名稱</th>
        <th>場站區域</th>
        <th>目前可用車輛
          <i @click="upDeta('sbi')" class="fa fa-sort-asc" aria-hidden="true"></i>
          <i class="fa fa-sort-desc" aria-hidden="true"></i>
        </th>
        <th>總停車格
          <i @click="upDeta('tot')" class="fa fa-sort-asc" aria-hidden="true"></i>
          <i class="fa fa-sort-desc" aria-hidden="true"></i>
        </th>
        <th>資料更新時間</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="s in Site" :key="s.sno">
        <td>{{ s.sno }}</td>
        <td>{{ s.sna }}</td>
        <td>{{ s.sarea }}</td>
        <td>{{ s.sbi }}</td>
        <td>{{ s.tot }}</td>
        <td>{{ timeFormat(s.mday) }}</td>
      </tr>
    </tbody>
  </table>
  <ul>
    <li class="pager" @click="pagePrecious"><a href="#">&lt;</a></li>
    <li v-for="pages in 10" @click="pageActive=pages" class="pager">
      <a :class="{'page-color': pages === pageActive}" href="#">{{ pages }}</a>
    </li>
    <li class="pager" @click="pageChange"><a href="#">&gt;</a></li>
  </ul>
</div>
</template>

<style scoped>
.app {
  padding: 1rem;
}
.page-color {
  color: red;
}
.pager {
      float: left;
      display: block;
      width: 4rem;
      height: 20px;
      text-align: center;
      line-height: 0;
      margin-right: 5px;
      margin-bottom: 30px;
      padding: 5px;
      border: 1px solid #aaa;
      padding: 1rem;
      font-size: 1.33rem;
    }
.pager a {
  text-decoration: none;
}
</style>