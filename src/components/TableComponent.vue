<template>
  <div class="options">
    <div class="options-data">
      <input
        v-if="!filterLeather"
        @input="
          $emit(
            'changeDataCell',
            editCell.user,
            editCell.propName,
            $event.target.value
          )
        "
        :disabled="editCell ? false : true"
        :value="(editCell && editCell.user[editCell.propName].toString()) || ''"
        placeholder="Tap on a list element to edit it"
        type="text"
        class="top-input"
      />
      <input
        v-if="filterLeather"
        @input="sendFilterEmit($event.target.value, filterPropertyComputed)"
        :disabled="filterPropertyComputed ? false : true"
        class="top-input"
        placeholder="Enter a word to filter results under it"
        type="text"
      />
      <select
        v-if="filterLeather && exObjProps.length"
        @change="sendFilterEmit(filterWord, $event.target.value)"
        :value="filterPropertyComputed"
      >
        <option v-for="prop in exObjProps" :value="prop">
          {{ prop }}
        </option>
      </select>
      <select
        v-if="filterLeather"
        @change="sendFilterModeEmit($event.target.value)"
        :value="filterMode"
      >
        <option value="equally">equal</option>
        <option value="contain">contain</option>
        <option value="more">more</option>
        <option value="less">less</option>
      </select>
      <button
        v-if="!filterLeather"
        @click="filterLeatherOff"
        class="change-mode-button"
      >
        ./search
      </button>
      <button
        v-if="filterLeather"
        @click="filterLeatherOn"
        class="change-mode-button"
      >
        ./edit
      </button>
    </div>
  </div>
  <div v-if="!dataArray.length">There is nothing to show!</div>
  <ul class="table" :style="{ width: `fit-content`, height: `fit-content` }">
    <ul class="table-row" v-for="user in dataArray">
      <li
        v-for="(value, propName) in user"
        @click.ctrl="setSortProp(propName)"
        :class="`cell ${classPrefix}${propName.toLowerCase()}`"
        :style="`width:${
          columnSizes[propName.toLowerCase()]
            ? columnSizes[propName.toLowerCase()].width
            : ''
        };`"
      >
        <div
          @click="
            () => {
              editCell = { user, propName };
            }
          "
        >
          {{ value }}
        </div>
        <div
          @dragstart="dragStart($event, propName.toLowerCase())"
          @dragend="dragEnd($event, propName.toLowerCase())"
          draggable="true"
          class="draggable"
        ></div>
      </li>
    </ul>
  </ul>
</template>
<script>
export default {
  props: ['columns'],
  emits: ['sendFilter', 'changeDataCell', 'changeFilterMode'],
  data() {
    return {
      // cssProps
      columnWidth: 0,
      columnSizes: {},
      classPrefix: 'column-',

      // sort
      sortPropName: false,
      sortMode: true,
      lastSortKey: false,

      //filtering names
      filterWord: '',
      filterProperty: false,
      filterLeather: false,
      filterMode: 'contain',

      // dataChange
      editCell: false,

      //drag and drop attributes
      pointer: 0,
    };
  },
  computed: {
    filterPropertyComputed() {
      return this.filterProperty || this.exObjProps[0];
    },
    columnsPropComputed() {
      return this.columns || [];
    },
    exObj() {
      return this.columnsPropComputed[0];
    },
    exObjProps() {
      let props = [];
      for (let prop in this.exObj) {
        if (this.exObj.hasOwnProperty(prop)) {
          props.push(prop);
        }
      }
      return props;
    },
    dataArray() {
      let dataArray = this.columnsPropComputed;

      if (this.sortPropName) {
        dataArray = this.sortBy(this.sortPropName, dataArray);
      }

      return dataArray;
    },
  },
  //contains:
  // methods of
  // - sorting
  // - changing scales of the table (dragging)
  methods: {
    filterLeatherOn() {
      this.filterLeather = !this.filterLeather;
    },
    filterLeatherOff() {
      this.filterWord = '';
      this.filterLeather = !this.filterLeather;
    },
    sendFilterEmit(filterWord, filterProperty) {
      this.filterWord = filterWord;
      this.filterProperty = filterProperty;
      this.$emit('sendFilter', filterWord, filterProperty);
    },
    sendFilterModeEmit(filterMode) {
      this.filterMode = filterMode;
      this.$emit('changeFilterMode', filterMode);
    },
    setSortProp(propName) {
      this.sortPropName = false;
      this.sortPropName = propName;
    },
    sortBy(key, array) {
      const isNumeric = Number.isInteger(this.exObj[key]);

      this.lastSortKey ? '' : (this.lastSortKey = key);
      if (this.lastSortKey == key) {
        this.sortMode = !this.sortMode;
      }

      array = array.sort(this.decideSort(key, isNumeric, this.sortMode));

      this.lastSortKey = key;

      return array;
    },
    decideSort(key, isNumeric, sortMode) {
      function num(a, b) {
        const aNum = a[this.key];
        const bNum = b[this.key];
        return this.sortMode ? aNum - bNum : bNum - aNum;
      }
      function alp(a, b) {
        const aLower = a[this.key].toString().toLowerCase();
        const bLower = b[this.key].toString().toLowerCase();
        return this.sortMode ? aLower > bLower : aLower < bLower;
      }
      return isNumeric
        ? num.bind({ key, sortMode })
        : alp.bind({ key, sortMode });
    },
    dragStart(e, propName) {
      this.pointer = e.clientX;
      this.columnWidth = document.querySelector(
        `.${this.classPrefix}${propName}`
      ).clientWidth;
    },
    dragEnd(e, propName) {
      const columnWidth = `${this.columnWidth - (this.pointer - e.clientX)}px`;

      this.columnSizes[propName] = { width: columnWidth };

      document
        .querySelectorAll(`.${this.classPrefix}${propName}`)
        .forEach((node) => {
          node.style.width = columnWidth;
        });
    },
  },
};
</script>
<style scoped>
.table {
  border: 1px solid black;
  /* overflow: scroll; */
  padding: 0;
  margin: 0 auto;
}
.table ul {
  padding: 0;
  display: flex;
}
.cell {
  list-style: none;
  width: 100px;
  height: 20px;
  overflow: hidden;
  padding: 1px;
  border: 1px solid black;
  position: relative;
  flex-shrink: 0;
}

.top-input {
  width: 500px;
}

.options {
  display: flex;
  flex-direction: column;
}
.options select {
  flex: 2;
}

.options button {
  flex: 1;
}

.change-mode-button {
  background-color: lightcoral;
  height: 23px;
  padding: 0 10px;
}

.draggable {
  width: 0.5vh;
  height: 100%;
  position: absolute;
  right: 0;
  top: 0;
  cursor: pointer;
}
</style>
