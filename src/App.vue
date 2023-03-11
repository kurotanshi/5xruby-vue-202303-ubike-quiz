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


// 搜尋站點名稱
const searchName = ref('');
//筆記： 使用computed會儲存filter過後的data，如果資料沒有變動，不會重複filter
const filteredStopName = computed(() =>{
  return uBikeStops.value.filter( s => s.sna.includes(searchName.value) )
});

// 是否隱藏空站點
const isHideEmptyStops = ref(false);
const filteredStops = computed(() =>{
  return isHideEmptyStops.value
  ? filteredStopName.value.filter( s => s.sbi > 0)
  : filteredStopName.value;
});

// click
const isSortedBy = ref('')
const isOrderAsc = ref(null)
const sortByAttr = (attr) => {
  if( isSortedBy.value !== attr ){
    isSortedBy.value = attr
    isOrderAsc.value = false //優先降冪
  }else{
    isOrderAsc.value = !isOrderAsc.value
  }
  // console.log(isSortedBy.value, isOrderAsc.value)
}

// 排序
const sortedStops = computed(() =>{
  if(isSortedBy){
    return isOrderAsc.value
    ? [...filteredStops.value].sort((a, b) => a[isSortedBy.value] - b[isSortedBy.value])
    : [...filteredStops.value].sort((a, b) => b[isSortedBy.value] - a[isSortedBy.value])
  }else{
    return filteredStops.value
  }
});

// 分頁
const currentPage = ref(1)
const totalPage = computed(() =>  Math.ceil( sortedStops.value.length / 20 ))
const pageStart = computed(() =>  currentPage.value < 6 ? 1 : currentPage.value - 5)
const pageEnd = computed(() => {
  if( currentPage.value < 6 && totalPage.value > 10 ) return 11
  else if(currentPage.value < totalPage.value - 5) return currentPage.value + 5
  else return totalPage.value 
})
const setPage = (page) => {
  if(1 > page || page > totalPage.value) return
  currentPage.value = page
}

const pageSortedStops = computed(() =>{
  return sortedStops.value.slice( 20 *(currentPage.value-1), 20 * currentPage.value )
})


</script>

<template>
<div class="app">
  <h1>YouBike2.0 臺北市公共自行車即時資訊</h1>
  <div>
    <label>
      站點名稱搜尋：
      <input type="text" v-model="searchName">
    </label>
    <label>
      <input type="checkbox" v-model="isHideEmptyStops">
      隱藏空站點
    </label>
  </div>

  <table class="table table-striped">
    <thead>
      <tr>
        <th>#</th>
        <th>場站名稱</th>
        <th>場站區域</th>
        <th class="sort" @click="sortByAttr('sbi')">目前可用車輛
          <i class="fa fa-sort" aria-hidden="true"></i>
        </th>
        <th class="sort" @click="sortByAttr('tot')">總停車格
          <i class="fa fa-sort" aria-hidden="true"></i>
        </th>
        <th>資料更新時間</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for=" s in pageSortedStops" :key="s.sno">
        <td>{{ s.sno }}</td>
        <td>{{ s.sna }}</td>
        <td>{{ s.sarea }}</td>
        <td>{{ s.sbi }}</td>
        <td>{{ s.tot }}</td>
        <td>{{ timeFormat(s.mday) }}</td>
      </tr>
    </tbody>
  </table>
  <div class="perpage">
    <ul>
      {{ pageStart }}
      {{ pageEnd }}
      <li class="totalPage">共 {{ totalPage }} 頁</li>
      <li @click="setPage(currentPage-1)">
        <i class="fa fa-angle-left" aria-hidden="true"></i>
      </li>
      <li v-for="page in totalPage " 
          :class="{'active': (currentPage === page)}"
          @click="setPage(page)"
      >{{ page }}</li>
      <li @click="setPage(currentPage+1)">
        <i class="fa fa-angle-right" aria-hidden="true"></i>
      </li>
    </ul>
  </div>
</div>
</template>

<style scoped>
.app {
  padding: 1rem;
}
label{
  margin: 2rem 2rem 2rem 0;
}
.sort{
  cursor: pointer;
}

.perpage ul{
  display: flex;
    justify-content: end;
    margin: auto;
    flex-wrap: wrap;
}
.perpage li {
  list-style-type: none;
  border: 1px solid #d5d5d5;
  width: 30px;
  padding: 6px;
  text-align: center;
  margin: 4px;
  cursor: pointer;
}
.perpage li:hover {
  background-color: #d5d5d5;
}
.perpage .totalPage{
  width: auto;
}
.perpage .active{
  background-color: #9bceff;
  color: #fff;
  border: 1px solid #9bceff;
}

</style>