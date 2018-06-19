<template>
  <table class="table table-striped table-bordered" :id="[tableId]"
    v-bind:class="[tableClass]"
  ></table>
</template>

<script>
// DataTable stuff
import $ from 'jquery';
import 'datatables.net-bs4';
import 'datatables.net-colreorder';
import 'datatables.net-fixedcolumns';
import 'datatables.net-responsive';
import 'datatables.net-buttons-bs4';
import 'datatables.net-buttons';
import Vue from 'vue';

import 'bootstrap/dist/css/bootstrap.min.css';
import 'datatables.net-bs4/css/dataTables.bootstrap4.css';
import 'datatables.net-responsive-bs4/css/responsive.bootstrap4.min.css';
import 'datatables.net-fixedcolumns-bs4/css/fixedColumns.bootstrap4.min.css';
import 'datatables.net-buttons-bs4/css/buttons.bootstrap4.css';
import 'datatables.net-buttons/js/buttons.html5';

// FIXME: Should not make jszip globally accessible, should pass to buttons.html5 file.
// It is needed for datatable excel export.
window.JSZip = require('jszip');

// TODO:
// 1. Remove export functionality and all code, including jszip
// 2. Remove button functionality and all code.
// 3. Remove defaults that are provided. No default to be provided ... instead client should provide.
// 4. Add a watch on the tableData, so we refresh automatically.
// 5. Remove usage of $t module.
// 6. Is tableID necessary? It says it is necessary for state save, but that doesn't seem right. Discuss with me.
// 7. Write methods to return an instance of the dtInstance so users can use the instance to do whatever they want.
// 8. Remove BootStrap 4 related stuff.
// 9. Create examples.

export default {
  name: 'DataTable',
  props: {
    columns: {
      type: Array,
    },
    opts: {
      default: () => null,
      type: Object,
    },
    langauge: {
      type: Object,
      default: () => ({}),
    },
    tableData: {
      type: Array,
    },
    tableClass: {
      type: String,
    },
    tableId: {
      type: String,
    },
  },
  data: () => ({
    isLoaded: false,
    dtInstance: null,
    finalOpts: {},
    options: {},
    excelButton: {},
    currentPagination: 0,
  }),
  methods: {
    updateData: function updateData(tableData) {
      let finalData = tableData;
      if (!finalData) {
        finalData = [];
      }

      if (this.isLoaded) {
        // If table is already loaded, just redraw the table.
        this.dtInstance.clear().rows.add(finalData);
        this.dtInstance.draw(false);
        this.dtInstance.columns.adjust();
        return;
      }

      // Table is not loaded, initialize the table now.
      this.finalOpts.data = finalData;

      // We should select datatable by id for datatable state save functionality.
      this.dtInstance = $(`#${this.tableId}`).DataTable(
        Vue.util.extend(
          {
            columns: this.columns,
          },
          this.finalOpts,
        )
      );
      this.isLoaded = true;
    },
    updateColumnHeader(headerText, columnIndex) {
      $(this.dtInstance.columns(columnIndex).header()).text(headerText);
    },
  },
  mounted: function mounted() {
    try {
      if (this.opts === null) {
        this.options = this.$parent.getDataTableOptions();
      } else {
        this.options = this.opts;
      }
    } catch (err) {
      throw new Error('Please pass options for DataTable either as a property, or via a `getDataTableOptions` method.');
    }
    this.finalOpts = Vue.util.extend(
      {
        paging: false,
        dom: 'frtipB',
        pageLength: 25,
        order: [[0, 'desc']],
        searching: false,
        stateSave: true,
        scrollCollapse: true,
        language: {
          info: `${this.$t('dtMsgs.totalRecords')}: _TOTAL_`,
          infoEmpty: `${this.$t('dtMsgs.totalRecords')}: 0`,
          emptyTable: this.$t('dtMsgs.noRecords'),
          infoFiltered: this.$t('dtMsgs.filtered'),
        }
      },
      this.options
    );
    this.updateData(this.tableData);
  },
  beforeDestroy() {
    this.dtInstance.destroy(true);
    this.isLoaded = false;
    this.dtInstance = null;
    this.finalOpts = {};
  },
};
</script>
