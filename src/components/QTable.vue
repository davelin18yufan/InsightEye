<template>
  <q-table
    class="q-mt-sm bg"
    row-key="name"
    :columns="columns"
    :rows="rows"
    separator="cell"
    dense
    selection="multiple"
    v-model:selected="selected"
    v-model:pagination="pagination"
    :rows-per-page-label="'每頁筆數：'"
    :pagination-label="getPaginationLabel"
    @update:selected="onSelectChange"
    :sort-method="customSort"
    binary-state-sort
    :loading="loading"
    :filter="filter"
    @request="onRequest"
  >
    <!-- search -->
    <template v-slot:top-right>
      <q-input
        borderless
        dense
        debounce="300"
        v-model="filter"
        placeholder="Search for anything.."
        class="input"
      >
        <template v-slot:append>
          <q-icon name="search" />
        </template>
      </q-input>
    </template>

    <!-- tooltip -->
    <template v-slot:body-cell="props">
      <q-td :props="props">
        <p>
          {{ props.value }}
          <q-tooltip
            anchor="bottom middle"
            self="top right"
            transition-show="flip-right"
            transition-hide="flip-left"
          >
            {{ props.value }}
          </q-tooltip>
        </p>
      </q-td>
    </template>
  </q-table>
</template>
<script>
import { ref, toRefs } from "vue";

export default {
  props: {
    columns: {
      type: Array,
      default: () => [],
    },

    rows: {
      type: Array,
      default: () => [],
    },
  },

  setup(props, { emit }) {
    const selected = ref([]);
    const filter = ref("");
    const loading = ref(false);
    const { rows, columns } = toRefs(props);

    function onSelectChange() {
      // data passed from QTable is snapshot without updated
      const existingRow = rows.value.filter((row) =>
        selected.value.some((s) =>
          Object.keys(row).every((key) => row[key] === s[key])
        )
      );
      selected.value = existingRow;

      // console.log("send", selected.value)
      emit("update:selected", existingRow);
    }
    const pagination = ref({
      sortBy: "desc",
      descending: false,
      rowsPerPage: 5,
    });

    function customSort(rows, sortBy, descending) {
      const data = [...rows];

      if (sortBy) {
        data.sort((a, b) => {
          const x = descending ? b : a;
          const y = descending ? a : b;

          if (sortBy === "name" || "gender" || "email") {
            // string sort
            return x[sortBy] > y[sortBy] ? 1 : x[sortBy] < y[sortBy] ? -1 : 0;
          } else {
            // numeric sort
            return parseFloat(x[sortBy]) - parseFloat(y[sortBy]);
          }
        });
      }

      return data;
    }

    function filterAndSort(startRow, count, filter, sortBy, descending) {
      const data = filter
        ? rows.value.filter((row) =>
            ["name", "cellphone", "email", "gender"].some((key) =>
              row[key].includes(filter)
            )
          )
        : rows.value.slice();

      // handle sortBy
      if (sortBy) {
        const sortFn =
          sortBy === "desc"
            ? descending
              ? (a, b) => (a.name > b.name ? -1 : a.name < b.name ? 1 : 0)
              : (a, b) => (a.name > b.name ? 1 : a.name < b.name ? -1 : 0)
            : descending
            ? (a, b) => parseFloat(b[sortBy]) - parseFloat(a[sortBy])
            : (a, b) => parseFloat(a[sortBy]) - parseFloat(b[sortBy]);
        data.sort(sortFn);
      }

      return data.slice(startRow, startRow + count);
    }

    // calculate rows length
    function getRowsNumberCount(filter) {
      if (!filter) return rows.value.length;

      let count = 0;
      rows.value.forEach((item) => {
        if (item.name.includes(filter)) ++count;
      });
      return count;
    }

    function onRequest(props) {
      const { page, rowsPerPage, sortBy, descending } = props.pagination;
      const filter = props.filter;

      loading.value = true;

      // debounced delayed fn
      setTimeout(() => {
        // update rowsCount
        pagination.value.rowsNumber = getRowsNumberCount(filter);

        // get all rows if "All" is selected
        const fetchCount =
          rowsPerPage === 0 ? pagination.value.rowsNumber : rowsPerPage;

        // calculate starting row
        const startRow = (page - 1) * rowsPerPage;

        // re-format
        const formatData = filterAndSort(
          startRow,
          fetchCount,
          filter,
          sortBy,
          descending
        );

        // replace existing rows
        rows.value.splice(0, rows.value.length, ...formatData);

        // update local pagination object
        pagination.value.page = page;
        pagination.value.rowsPerPage = rowsPerPage;
        pagination.value.sortBy = sortBy;
        pagination.value.descending = descending;

        loading.value = false;
      }, 1500);
    }

    return {
      columnsState: columns.value,
      rowsState: rows.value,
      selected,
      onSelectChange,
      pagination,
      getPaginationLabel: () => `共${rows.value.length}筆資料`,
      customSort,
      filter,
      loading,
      onRequest,
    };
  },
};
</script>
<style lang="scss" scoped>
.bg {
  background-color: #f5f2f2;
}
.input {
  background-color: #d7d5d5;
  padding: 0 0.5rem;
  border-radius: 4px;
}
</style>
