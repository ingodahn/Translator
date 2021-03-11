<template>
  <div>
    <Selector :Select="select" @change-page="changePage(1)" />
    <Editor
      :CurrentRow="currentRow"
      v-if="currentRow.group != 'none'"
      @saveAll="saveAll"
    />
    <DataTable :header-fields="headerFields" :data="data" :css="datatableCss">
      <!-- Action button slot -->
      <template v-slot:actions="props">
        <input
          type="button"
          class="btn btn-info"
          value="Edit"
          @click="dtEditClick(props)"
        />
      </template>
      <!--
        Pagination component as a slot, but could be drag out from Database element
      -->
      <template v-slot:pagination>
        <Pagination
          :page="currentPage"
          :total-items="totalItems"
          :items-per-page="itemsPerPage"
          :css="paginationCss"
          @on-update="changePage"
          @update-current-page="updateCurrentPage"
        />
      </template>

      <!--
        ItemsPerPage component as a slot, but could be drag out from Database element
      -->
      <div class="items-per-page" slot="ItemsPerPage">
        <label>Items per page</label>
        <ItemsPerPageDropdown
          :list-items-per-page="listItemsPerPage"
          :items-per-page="itemsPerPage"
          :css="itemsPerPageCss"
          @on-update="updateItemsPerPage"
        />
      </div>
    </DataTable>
  </div>
</template>

<script>
import { DataTable, ItemsPerPageDropdown, Pagination } from "v-datatable-light";
import { saveAs } from "file-saver";
import Selector from "./Selector.vue";
import Editor from "./Editor.vue";
export default {
  name: "TranslatorTable",
  props: ["Source", "Target"],
  data() {
    return {
      select: {
        expr: /.*/,
        selectBy: "source"
      },
      headerFields: [],
      allData: [],
      data: [],
      datatableCss: {
        table: "table table-bordered table-hover table-striped table-center",
        theadTh: "header-item",
        thWrapper: "th-wrapper",
        thWrapperCheckboxes: "th-wrapper checkboxes",
        arrowsWrapper: "arrows-wrapper",
        arrowUp: "arrow up",
        arrowDown: "arrow down",
        footer: "footer"
      },
      paginationCss: {
        paginationItem: "pagination-item",
        moveFirstPage: "move-first-page",
        movePreviousPage: "move-previous-page",
        moveNextPage: "move-next-page",
        moveLastPage: "move-last-page",
        pageBtn: "page-btn"
      },
      itemsPerPageCss: {
        select: "item-per-page-dropdown"
      },
      totalItems: 0,
      listItemsPerPage: [5, 10, 20, 50, 100],
      itemsPerPage: 10,
      currentPage: 1,
      currentRow: {
        id: "none",
        group: "none",
        sourceTerm: "",
        targetTerm: ""
      }
    };
  },
  components: {
    Selector,
    Editor,
    DataTable,
    ItemsPerPageDropdown,
    Pagination
  },
  created: function() {
    this.headerFields = [
      //"__slot:checkboxes",
      {
        name: "group",
        label: "Group",
        sortable: false
      },
      {
        name: "sourceTerm",
        label: this.Source.lang,
        sortable: false
      },
      {
        name: "targetTerm",
        label: this.Target.lang,
        sortable: false
      },
      "__slot:actions"
    ];
    this.makeAllData(this.Source.data, this.Target.data);
    this.data = this.allData.slice(0, this.itemsPerPage);
    this.totalItems = this.allData.length;
  },
  methods: {
    makeAllData: function(source, target, groups = []) {
      Object.keys(source).forEach(k => {
        if (typeof source[k] == "string") {
          const tt = Object.prototype.hasOwnProperty.call(target, k)
            ? target[k]
            : "";
          const idgg = groups.slice();
          idgg.push(k);
          const idg = idgg.join(".");
          this.allData.push({
            id: idg,
            group: idg,
            sourceTerm: source[k],
            targetTerm: tt
          });
        } else {
          let to = {};
          if (Object.prototype.hasOwnProperty.call(target, k)) to = target[k];
          const group1 = groups.slice();
          group1.push(k);
          this.makeAllData(source[k], to, group1);
        }
      });
    },

    updateItemsPerPage: function(itemsPerPage) {
      this.itemsPerPage = parseInt(itemsPerPage, 10);
      if (itemsPerPage >= this.allData.length) {
        this.data = this.allData;
      } else {
        this.data = this.allData.slice(0, itemsPerPage);
      }
    },

    changePage: function(currentPage) {
      this.currentPage = currentPage;
      const start = (currentPage - 1) * this.itemsPerPage;
      const end = currentPage * this.itemsPerPage;
      this.data = this.filteredData().slice(start, end);
    },

    updateCurrentPage: function(currentPage) {
      this.currentPage = currentPage;
    },

    filteredData: function() {
      const filteredBy = this.select.selectBy + "Term",
        expr = this.select.expr;
      const filteredData = this.allData.filter(entry =>
        entry[filteredBy].match(expr)
      );
      this.totalItems = filteredData.length;
      return filteredData;
    },

    dtEditClick: function(props) {
      this.currentRow = props.rowData;
    },

    saveAll: function() {
      const newTrans = new Object();
      this.allData.forEach(p => {
        if (p.targetTerm.length) {
          const curGroups = p.group.split(".");
          let curTrans = newTrans;
          for (let i = 0; i < curGroups.length - 1; i++) {
            const item = curGroups[i];
            if (!Object.prototype.hasOwnProperty.call(curTrans, item))
              curTrans[item] = {};
            curTrans = curTrans[item];
          }
          curTrans[curGroups.slice(-1)[0]] = p.targetTerm;
        }
      });
      // Writing newTrans
      const outData = JSON.stringify(newTrans, null, 2);
      const blob = new Blob([outData], {
        type: "text/plain; charset=utf-8"
      });
      saveAs(blob, this.Target.lang + ".json");
    }
  },
  computed: {}
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

#app .title {
  margin-bottom: 30px;
}

#app .items-per-page {
  height: 100%;
  display: flex;
  align-items: center;
  color: #337ab7;
}

#app .items-per-page label {
  margin: 0 15px;
}

/* Datatable CSS */
.v-datatable-light .header-item {
  cursor: pointer;
  color: #337ab7;
  transition: color 0.15s ease-in-out;
}

.v-datatable-light .header-item:hover {
  color: #ed9b19;
}

.v-datatable-light .header-item.no-sortable {
  cursor: default;
}
.v-datatable-light .header-item.no-sortable:hover {
  color: #337ab7;
}

.v-datatable-light .header-item .th-wrapper {
  display: flex;
  width: 100%;
  height: 100%;
  font-weight: bold;
  align-items: baseline;
}

.v-datatable-light .header-item .th-wrapper.checkboxes {
  justify-content: center;
}

.v-datatable-light .header-item .th-wrapper .arrows-wrapper {
  display: flex;
  flex-direction: column;
  margin-left: 10px;
  justify-content: space-between;
}

.v-datatable-light .header-item .th-wrapper .arrows-wrapper.centralized {
  justify-content: center;
}

.v-datatable-light .arrow {
  transition: color 0.15s ease-in-out;
  width: 0;
  height: 0;
  border-left: 8px solid transparent;
  border-right: 8px solid transparent;
}

.v-datatable-light .arrow.up {
  border-bottom: 8px solid #337ab7;
  margin-bottom: 5px;
}

.v-datatable-light .arrow.up:hover {
  border-bottom: 8px solid #ed9b19;
}

.v-datatable-light .arrow.down {
  border-top: 8px solid #337ab7;
}

.v-datatable-light .arrow.down:hover {
  border-top: 8px solid #ed9b19;
}

.v-datatable-light .footer {
  display: flex;
  justify-content: space-between;
  width: 500px;
}
/* End Datatable CSS */

/* Pagination CSS */
.v-datatable-light-pagination {
  list-style: none;
  display: flex;
  align-items: center;
  justify-content: flex-start;
  margin: 0;
  padding: 0;
  width: 300px;
  height: 30px;
}

.v-datatable-light-pagination .pagination-item {
  width: 30px;
  margin-right: 5px;
  font-size: 16px;
  transition: color 0.15s ease-in-out;
}

.v-datatable-light-pagination .pagination-item.selected {
  color: #ed9b19;
}

.v-datatable-light-pagination .pagination-item .page-btn {
  background-color: transparent;
  outline: none;
  border: none;
  color: #337ab7;
  transition: color 0.15s ease-in-out;
}

.v-datatable-light-pagination .pagination-item .page-btn:hover {
  color: #ed9b19;
}

.v-datatable-light-pagination .pagination-item .page-btn:disabled {
  cursor: not-allowed;
  box-shadow: none;
  opacity: 0.65;
}
/* END PAGINATION CSS */

/* ITEMS PER PAGE DROPDOWN CSS */
.item-per-page-dropdown {
  background-color: transparent;
  min-height: 30px;
  border: 1px solid #337ab7;
  border-radius: 5px;
  color: #337ab7;
}
.item-per-page-dropdown:hover {
  cursor: pointer;
}
/* END ITEMS PER PAGE DROPDOWN CSS */
</style>
