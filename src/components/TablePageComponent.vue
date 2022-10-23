<script setup>
import TableComponent from './TableComponent.vue';
</script>
<template>
  <TableComponent
    @send-filter="setFilter"
    @change-data-cell="setDataCell"
    @change-filter-mode="setFilterMode"
    :columns="pageElementsComputed[currentPageId]"
  />
  <div class="pagination">
    <select
      class="paginator-tool"
      @change="setCurrentPageId(0)"
      v-model="pageCounter"
    >
      <option value="10">10</option>
      <option value="20">20</option>
      <option value="30">30</option>
    </select>
    <ul>
      <li
        @click="setCurrentPageId(index)"
        v-for="(item, index) in pageElementsComputed"
      >
        Page&nbsp{{ index + 1 }}
      </li>
    </ul>
  </div>
</template>
<script>
export default {
  props: ['url'],
  data() {
    return {
      placeholderData: [
        { id: 1, name: 'John', completed: 'false' },
        { id: 2, name: 'John Die', completed: 'false' },
        { id: 3, name: 'John Doe', completed: 'true' },
        { id: 4, name: 'John Dead', completed: 'false' },
      ],
      currentPageId: 0,
      pageCounter: 10,
      filterWord: '',
      filterProperty: '',
      filterMode: '',
      filterModeArray: ['contain', 'equally', 'more', 'less'],
    };
  },
  computed: {
    pageElementsComputed() {
      let paginated = [],
        mainArray = [...this.placeholderDataComputed],
        lengthCondition = mainArray.length;
      while (lengthCondition) {
        let innerArray = [];
        let counter = this.pageCounter;
        while (counter && lengthCondition) {
          innerArray.push(mainArray.pop());
          counter--;
          lengthCondition--;
        }
        paginated.push(innerArray);
      }
      return paginated;
    },

    placeholderDataComputed() {
      const filterWord = this.filterWord,
        filterKey = this.filterProperty,
        filterMode = this.filterModeComputed;

      function ff(e) {
        return this.reg.test(e[this.filterKey]) ? e : false;
      }

      const reg = {
        contain: new RegExp(filterWord),
        equally: new RegExp(`^${filterWord}$`),
        more: new RegExp(filterWord, 'g'),
        less: new RegExp(filterWord, 'g'),
      }[filterMode];

      if ((filterMode == 'more' || filterMode == 'less') && filterWord != '') {
        const filtered = this.filterRegSort(
          [...this.placeholderData],
          filterKey,
          reg
        );
        return filterMode == this.filterModeArray[2]
          ? filtered
          : filtered.reverse();
      }

      return this.placeholderData.filter(ff.bind({ filterKey, reg })).reverse();
    },

    filterModeComputed() {
      const filterMode = this.filterModeArray.find((e) => e == this.filterMode);
      return filterMode || this.filterModeArray[0];
    },
  },
  methods: {
    setDataCell(dataCell, key, value) {
      const cellId = this.placeholderData.findIndex((e) => e == dataCell);
      this.placeholderData[cellId][key] = value;
      // console.log(dataCell, key, value, cellId);
    },
    setFilterMode(filterMode) {
      this.filterMode = filterMode;
    },
    setCurrentPageId(id) {
      this.currentPageId = id;
    },
    setFilter(filterWord, filterProperty) {
      this.filterWord = filterWord;
      this.filterProperty = filterProperty;
    },
    /*
      arr = iterable array
      key = the arr property
      reg = /condition/g
      */
    filterRegSort(arr, key, reg) {
      let newArr = [];
      let arrLength = arr.length;
      while (arrLength) {
        const elem = arr.pop();
        const index = (elem[key].toString().match(reg) || []).length;
        newArr[index] = elem;
        arrLength--;
      }
      newArr[0] = false;

      return newArr.filter((e) => e);
    },
  },
  mounted() {
    fetch(this.url)
      .then((response) => response.json())
      .then((json) => {
        this.placeholderData = json;
        // this.placeholderData.length = 10;
      });
  },
};
</script>
<style>
.pagination {
  display: flex;
}

.paginator-tool {
  align-self: center;
}
.pagination li {
  list-style: none;
  margin: 10px;
  padding: 0 10px;
  border: 1px solid black;
}

.pagination li:hover {
  background-color: black;
  color: white;
  transition-duration: 2s;
}
.pagination ul {
  display: flex;
  flex-wrap: wrap;
}
</style>
