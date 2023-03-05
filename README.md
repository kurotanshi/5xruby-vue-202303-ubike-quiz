# 5xRuby 202303 Vue 第一次作業

修改這份 YouBike 即時資訊表，並加上以下功能：

1. 站點名稱搜尋
2. 目前可用車輛 / 總停車格 的排序功能
3. 加入分頁功能，每頁 20 筆資料

欄位說明: https://data.taipei/dataset/detail?id=c6bc8aed-557d-41d5-bfb1-8da24f78f2fb

- sno：站點代號
- sna：場站名稱(中文)
- tot：場站總停車格
- sbi：場站目前車輛數量
- sarea：場站區域(中文) 
- mday：資料更新時間、
- lat：緯度
- lng：經度
- ar：地(中文)
- sareaen：場站區域(英文)
- snaen：場站名稱(英文)
- aren：地址(英文)
- bemp：空位數量
- act：全站禁用狀態

## 修改路徑： `/src/App.vue`

## Recommended IDE Setup

[VSCode](https://code.visualstudio.com/) + [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur) + [TypeScript Vue Plugin (Volar)](https://marketplace.visualstudio.com/items?itemName=Vue.vscode-typescript-vue-plugin).

## Customize configuration

See [Vite Configuration Reference](https://vitejs.dev/config/).

## Project Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Compile and Minify for Production

```sh
npm run build
```
