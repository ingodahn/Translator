<template>
  <div>
    <Selector :Select="select" @change-page="changePage(1)" />
    <Editor
      :CurrentRow="currentRow"
      v-if="currentRow.id != 'none'"
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
        group: "",
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
    this.data = this.initialData.slice(0, this.itemsPerPage);
    this.totalItems = this.initialData.length;
  },
  methods: {
    dtEditClick: function(props) {
      this.currentRow = props.rowData;
    },

    saveAll: function() {
      let newTrans = new Object();
      this.initialData.forEach(p => {
        let curGroups = p.group.split("/"),
          curSource = this.Source.data,
          curTrans = newTrans;
        let i = 0;
        while (i < curGroups.length - 1) {
          let g = curGroups[i];
          curSource = curSource[g];
          if (curTrans.hasOwnProperty(g)) curTrans = curTrans[g];
          else {
            curTrans[g] = new Object();
            curTrans = curTrans[g];
          }
          i++;
        }
        let gLast = curGroups[curGroups.length - 1];
        if (curTrans.hasOwnProperty(gLast))
          curTrans[gLast] += " | " + p.targetTerm;
        else {
          if (p.targetTerm.length) curTrans[gLast] = p.targetTerm;
        }
      });
      // Writing newTrans
      let outData = JSON.stringify(newTrans, null, 2);
      let blob = new Blob([outData], {
        type: "text/plain; charset=utf-8"
      });
      saveAs(blob, this.Target.lang + ".json");
    },

    updateItemsPerPage: function(itemsPerPage) {
      this.itemsPerPage = parseInt(itemsPerPage, 10);
      if (itemsPerPage >= this.initialData.length) {
        this.data = this.initialData;
      } else {
        this.data = this.initialData.slice(0, itemsPerPage);
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
      let filteredBy = this.select.selectBy + "Term",
        expr = this.select.expr;
      let filteredData = this.initialData.filter(entry =>
        entry[filteredBy].match(expr)
      );
      this.totalItems = filteredData.length;
      return filteredData;
    },
    addGroup: function(lines, groups, sourceObj) {
      let line = [];
      if (typeof sourceObj == "string") {
        line.push(groups.join("."));
        line.push(sourceObj);
        // Add translation if exists
        lines.push(line);
      }
    }
  },
  computed: {
    initialData: function() {
      let tableData = [],
        groups = Object.keys(this.Source.data),
        enr = 0;
      groups.forEach(g => {
        let td = this.Target.data.hasOwnProperty(g)
          ? this.Target.data[g]
          : "none";
        let gd = groupData(g, this.Source.data[g], td, enr);
        gd.forEach(e => tableData.push(e));
        enr += gd.length;
      });
      return tableData;
    },
    initialData1: function() {
      let tableData = [];
      return tableData;
    }
  }
};

function groupData(groups, gData, tData, enr) {
  let gEntries = [],
    myEnr = enr;
  if (typeof gData == "string") {
    let tts = gData.split(" | ");
    let tEntry =
      tData == "none" || tData == undefined
        ? new Array(tts.length).fill("")
        : tData.split(" | ");
    tts.forEach((ss, si) =>
      gEntries.push({
        id: myEnr,
        group: groups,
        sourceTerm: ss,
        targetTerm: tEntry[si]
      })
    );
    return gEntries;
  } else {
    Object.keys(gData).forEach(g => {
      let entries = groupData(
        groups + "/" + g,
        gData[g],
        tData == "none" ? "none" : tData[g],
        myEnr
      );
      entries.forEach(e => {
        gEntries.push(e);
        myEnr++;
      });
    });
  }
  return gEntries;
}
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
