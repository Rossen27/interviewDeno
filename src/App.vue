<template>
  <div class="row">
    <h1 class="col-10">
      取得資料至表格建立完成（含事件綁定）期間，共耗費 {{ elapsedTime }} ms
    </h1>
    <p class="col-2">第 {{ currentPage }} 頁 / 共 {{ totalPages }} 頁</p>
    <table>
      <thead>
        <tr>
          <th>Key</th>
          <th>cell1</th>
          <th>cell2</th>
          <th>cell3</th>
          <th>cell4</th>
          <th>cell5</th>
          <th>cell6</th>
          <th>cell7</th>
          <th>cell8</th>
          <th>cell9</th>
        </tr>
      </thead>
      <tbody>
        <template v-if="tableData.length > 0">
          <tr
            v-for="row in displayedData"
            :key="row.key"
            :class="{ active: selectedRow === row.key }"
            @click="selectRow(row.key)"
          >
            <td>
              <button @click.stop="toggleStar(row.key)">
                <img
                  :src="
                    row.starred
                      ? '/src/assets/star-filled.png'
                      : '/src/assets/star-empty.png'
                  "
                  alt="Star"
                />
              </button>
              {{ row.key }}
            </td>
            <td>{{ row.cell1 }}</td>
            <td>{{ row.cell2 }}</td>
            <td>{{ row.cell3 }}</td>
            <td>{{ row.cell4 }}</td>
            <td>{{ row.cell5 }}</td>
            <td>{{ row.cell6 }}</td>
            <td>{{ row.cell7 }}</td>
            <td>{{ row.cell8 }}</td>
            <td>{{ row.cell9 }}</td>
          </tr>
        </template>
        <template v-else>
          <tr>
            <td colspan="10">目前沒有資料</td>
          </tr>
        </template>
      </tbody>
    </table>
    <div
      class="btn-toolbar justify-content-between m-2"
      role="toolbar"
      aria-label="Toolbar with button groups"
    >
      <div class="btn-group" role="group" aria-label="First group">
        <ul class="pagination justify-content-end">
          <li class="page-item" :class="{ disabled: currentPage === 1 }">
            <a class="page-link" @click="prevPage" href="#"
              ><span aria-hidden="true">&laquo;</span></a
            >
          </li>
          <li
            v-for="page in totalPagesToShow"
            :key="page"
            class="page-item"
            :class="{ active: currentPage === page }"
          >
            <a class="page-link" @click="goToPage(page)" href="#">{{ page }}</a>
          </li>
          <li
            class="page-item"
            :class="{ disabled: currentPage === totalPages }"
          >
            <a class="page-link" @click="nextPage" href="#"
              ><span aria-hidden="true">&raquo;</span></a
            >
          </li>
        </ul>
      </div>
      <div class="btn-group">
        <button type="button" class="btn btn-sm btn-outline-secondary">
          選擇頁碼
        </button>
        <button
          type="button"
          class="btn btn-sm dropdown-toggle dropdown-toggle-split btn-outline-secondary"
          data-bs-toggle="dropdown"
          aria-expanded="false"
        >
          <span class="visually-hidden">Toggle Dropdown</span>
        </button>
        <ul class="dropdown-menu">
          <li v-for="page in totalPages" :key="page">
            <a class="dropdown-item" @click="goToPage(page)" href="#"
              >第 {{ page }} 頁</a
            >
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue';

// 這裡是定義所需的變量
const tableData = ref([]);
const selectedRow = ref(null);
const startTime = ref(null);
const elapsedTime = ref(0);
const itemsPerPage = ref(10);
const currentPage = ref(1);
const selectedPage = ref(1);
const maxPagesToShow = ref(10);

// 合併三份資料
function mergeData(data1, data2, data3) {
  console.log('Data1:', data1);
  console.log('Data2:', data2);
  console.log('Data3:', data3);

  // 檢查資料格式是否正確
  if (
    !Array.isArray(data1) ||
    !Array.isArray(data2) ||
    typeof data3 !== 'object'
  ) {
    throw new TypeError('其中一筆資料不符合預期格式');
  }

  // 將 data2 和 data3 轉換為 Map，方便查找
  const data2Map = Object.fromEntries(
    data2.map((item) => [item.key, item.cell8])
  );
  const data3Map = Object.fromEntries(
    Object.values(data3).map((item) => [item.cell4, item.cell9])
  );

  // 將 data1 的每一行資料合併 data2 和 data3 的資料
  return data1.map((row) => ({
    ...row,
    cell8: data2Map[row.key] || '',
    cell9: data3Map[row.cell4] || '',
    starred: false, // 新增 starred 屬性，用來標記是否為星號
  }));
}

// 取得資料並計算經過時間
async function fetchData() {
  try {
    startTime.value = Date.now(); // 記錄開始時間
    const [data1, data2, data3] = await Promise.all([
      fetch('/data/data1.json').then((res) => res.json()),
      fetch('/data/data2.json').then((res) => res.json()),
      fetch('/data/data3.json').then((res) => res.json()),
    ]);

    tableData.value = mergeData(data1, data2, data3); // 合併資料
    elapsedTime.value = Date.now() - startTime.value; // 計算經過時間
  } catch (error) {
    console.error('取得數據時出錯！錯誤訊息為:', error);
  }
}

// 選擇某一行
function selectRow(key) {
  selectedRow.value = key;
}

// 切換星號
function toggleStar(key) {
  const row = tableData.value.find((row) => row.key === key);
  if (row) row.starred = !row.starred;
}

// 使用 onMounted 鉤子來取得資料
onMounted(fetchData);

// 計算總頁數
const totalPages = computed(() =>
  Math.ceil(tableData.value.length / itemsPerPage.value)
);

// 計算要顯示的頁數
const totalPagesToShow = computed(() => {
  const pages = [];
  const start = Math.max(
    1,
    currentPage.value - Math.floor(maxPagesToShow.value / 2)
  );
  const end = Math.min(totalPages.value, start + maxPagesToShow.value - 1);

  for (let i = start; i <= end; i++) {
    pages.push(i);
  }

  return pages;
});

// 計算要顯示的資料
const displayedData = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage.value;
  const end = start + itemsPerPage.value;
  return tableData.value.slice(start, end);
});

// 當按下上一頁時監聽 currentPage 的變化，更新經過時間
function prevPage() {
  if (currentPage.value > 1) {
    currentPage.value--;
    updateElapsedTime();
  }
}

// 當按下下一頁時監聽 currentPage 的變化，更新經過時間
function nextPage() {
  if (currentPage.value < totalPages.value) {
    currentPage.value++;
    updateElapsedTime();
  }
}

// 跳轉到指定頁碼
function goToPage(page) {
  currentPage.value = page;
  selectedPage.value = page;
  updateElapsedTime();
}

// 當選擇下拉選單時跳轉到指定頁碼
function jumpToPage() {
  goToPage(selectedPage.value);
}

// 更新經過時間
function updateElapsedTime() {
  const start = Date.now();
  setTimeout(() => {
    elapsedTime.value = Date.now() - start;
  }, 0); // 使用 setTimeout 確保 DOM 更新後再計算時間
}
</script>

<style scoped>
table {
  width: 100%;
  border-collapse: collapse;
}

th,
td {
  border: 1px solid black;
  padding: 8px;
  text-align: center;
}

tr.active {
  background-color: yellow;
}

button {
  background: none;
  border: none;
  cursor: pointer;
}

img {
  width: 25px;
  height: 25px;
}
</style>
