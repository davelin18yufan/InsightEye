<template>
  <q-dialog ref="dialogRef" @hide="onDialogHide">
    <q-card class="q-dialog-plugin">
      <div class="q-pa-md" style="max-width: 400px">
        <q-form class="q-gutter-sm">
          <q-input
            filled
            v-model="name"
            label="Employee name"
            hint="Full name"
            lazy-rules
            :rules="[(val) => (val && val.length > 2) || 'Invalid name']"
          />


          <q-input
            filled
            type="email"
            v-model="email"
            label="Email"
            hint="Email"
            lazy-rules
            :rules="[(val) => (val && val.length > 10) || 'Please type correctly',]"
          />

            <q-input
              filled
              type="number"
              v-model="cellphone"
              label="cellphone"
              hint="mobile number"
              lazy-rules
              :rules="[
                (val) =>
                  (val !== null && val !== '') || 'Please type phone number',
              ]"
            />

          <q-select
            filled
            v-model="gender"
            :options="['男', '女']"
            label="gender"
          />

          <q-input
            filled
            type="text"
            v-model="birthday"
            label="birthday"
            hint="YYYY-MM-DD"
            lazy-rules
            :rules="[validateDateInput]"
          />
        </q-form>
      </div>
      <q-card-actions align="right">
        <q-btn color="primary" label="OK" @click="onOKClick" />
        <q-btn color="primary" label="Cancel" @click="onDialogCancel" />
      </q-card-actions>
    </q-card>
  </q-dialog>
</template>
<script>
import { useDialogPluginComponent, useQuasar } from "quasar";
import { ref } from "vue";

export default {
  emits: ['ok', 'hide'],
  methods: {
    validateDateInput(value) {
      const regex = /^\d{4}-(0[1-9]|1[0-2])-(0[1-9]|[12][0-9]|3[01])$/;

      if (!regex.test(value)) {
        return 'Please enter validate formation(YYYY-MM-DD)';
      }
      const [year, month, day] = value.split('-').map(Number);

      // verify month (1 - 12)
      if (month < 1 || month > 12) {
        return 'Month should be between 1 ~ 12';
      }

      // verify date（1 - 31）
      if (day < 1 || day > 31) {
        return 'Date should be between 1 ~ 31';
      }

      return true;
    },
    hide() {
      this.$refs.dialog.hide()
    },
  },
  setup(prop, {emit}) {
    const name = ref(null);
    const cellphone = ref(null);
    const email = ref(null);
    const gender = ref(null);
    const birthday = ref(null);
    const { dialogRef, onDialogOK, onDialogCancel } =
      useDialogPluginComponent();
    const $q = useQuasar()

    return {
      name,
      cellphone,
      email,
      gender,
      birthday,
      dialogRef,
      onDialogHide: () => {
        name.value= null
        email.value = null
        gender.value= null
        cellphone.value= null
        birthday.value = null
      },
      onDialogCancel: () => {
        emit('hide')
      },
      onOKClick() {
        if (!(name.value && email.value && cellphone.value && gender.value && birthday.value)) {
          return $q.notify({
            message:"Please completely fill this form",
            icon: "announcement"
          })
        }

        onDialogOK({
          name: name.value,
          cellphone: cellphone.value,
          email: email.value,
          gender: gender.value,
          birthday: birthday.value,
        });
      },
      onCancelClick: onDialogCancel,
    };
  },
};
</script>
