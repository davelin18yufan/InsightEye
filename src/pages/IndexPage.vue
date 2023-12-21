<template>
  <q-page class="row justify-left q-pa-lg">
    <div class="block-item">
      <div class="row justify-between items-center">
        <div class="text-h6">員工基本資訊</div>
        <div>
          <q-btn
            class="q-mr-sm btn border-none"
            color="primary"
            label="新增"
            @click="add"
          />
          <q-btn
            class="btn"
            color="white"
            text-color="black"
            label="刪除"
            @click="deleteEmployee"
          />
        </div>
      </div>
      <QTable
        :columns="state.columns"
        :rows="state.rows"
        ref="tableRef"
        @update:selected="onSelectedChangeFromTable"
      />
    </div>
  </q-page>
</template>

<script>
import { reactive, ref } from "vue";
import QTable from "src/components/QTable.vue";
import InputForm from "src/components/InputForm.vue";
import { api } from "../boot/axios";
import { useQuasar } from "quasar";

function convertToBirthdayFormat(dateTimeString) {
  const datePart = dateTimeString.split("T")[0];

  const dateObj = new Date(datePart);

  if (isNaN(dateObj.getTime())) {
    return "Invalid Date";
  }

  const year = dateObj.getFullYear();
  const month = (dateObj.getMonth() + 1).toString().padStart(2, "0");
  const day = dateObj.getDate().toString().padStart(2, "0");

  return `${year}/${month}/${day}`;
}

const { data } = await api.get("/members");
const rows = data?.members.map((item) => ({
  name: item.name,
  cellphone: item.cellphone,
  email: item.email,
  gender: item.gender,
  birthday: item.birthday,
}));

export default {
  name: "IndexPage",

  components: {
    QTable,
  },

  setup() {
    const $q = useQuasar();
    const selectList = ref([]);
    const onSelectedChangeFromTable = (data) => {
      // data passed from QTable is snapshot without updated
      // match state at both side
      const existingRow = state.rows.filter((row) =>
        data.some((s) => Object.keys(row).every((key) => row[key] === s[key]))
      );
      selectList.value = existingRow;
      // console.log("receive",data)
    };

    function add() {
      $q.dialog({
        title: "新增",
        component: InputForm,
        cancel: false,
        persistent: true,
      }).onOk((data) => {
        if (!data) return;

        const { name, email, gender, cellphone, birthday } = data;
        state.rows = [
          {
            name,
            cellphone,
            email,
            gender,
            birthday,
          },
          ...state.rows,
        ];
      });
    }

    function deleteEmployee() {
      if (selectList.value.length < 1) {
        $q.notify({ message: "Nothing is selected yet" });
        return;
      }
      $q.dialog({
        title: "刪除",
        message: `是否確定刪除這${selectList.value.length}筆資料`,
        cancel: true,
        persistent: true,
      })
        .onOk(() => {
          const selectNameList = selectList.value.map((s) => s.name);
          //
          const newRows = state.rows.filter(
            (item) => !selectNameList.includes(item.name)
          );

          state.rows = [...newRows];
        })
        .onCancel(() => {
          // console.log('>>>> Cancel')
        });
    }

    const state = reactive({
      columns: [
        {
          name: "name",
          label: "姓名",
          align: "left",
          field: "name",
          sortable: true,
        },
        {
          name: "cellphone",
          label: "手機",
          align: "left",
          field: "cellphone",
          sortable: true,
        },
        {
          name: "email",
          label: "信箱",
          align: "left",
          field: "email",
          sortable: true,
        },
        {
          name: "gender",
          label: "性別",
          align: "left",
          field: "gender",
          sortable: true,
        },
        {
          name: "birthday",
          label: "生日",
          align: "left",
          field: "birthday",
          format: (value) => convertToBirthdayFormat(value),
          sortable: true,
        },
      ],
      rows: rows,
    });

    return {
      state,
      onSelectedChangeFromTable,
      add,
      deleteEmployee,
    };
  },
};
</script>
<style lang="scss" scoped>
.block-item {
  min-width: 600px;
}
.btn {
  border: 2px solid lightgrey;
  border-radius: 4px;
}

.border-none {
  border: none;
}
</style>
