<template>
  <a-modal
    v-model:open="formState.visible"
    title="Student Detail"
    :okText="formState.type === 'info' ? 'Edit' : 'Save'"
    cancelText="Cancel"
    @ok="handleOK"
  >
    <a-form
      layout="vertical"
      :rules="rules"
      :model="formState"
      :disabled="formState.type === 'info'"
    >
      <div style="display: flex">
        <a-form-item has-feedback label="First Name" name="first_name">
          <a-input
            v-model:value="formState.first_name"
            placeholder="Perry"
            style="display: flex; flex: 1; margin-right: 10px"
          />
        </a-form-item>
        <a-form-item has-feedback label="Last Name" name="last_name">
          <a-input
            v-model:value="formState.last_name"
            placeholder="Platypus"
            style="display: flex; flex: 1; margin-left: 10px"
          />
        </a-form-item>
      </div>

      <a-form-item has-feedback label="Phone" name="phone">
        <a-input v-model:value="formState.phone" placeholder="84987654321" />
      </a-form-item>

      <a-form-item has-feedback label="Email" name="email">
        <a-input
          v-model:value="formState.email"
          placeholder="perry_x_spy@gmail.com"
        />
      </a-form-item>

      <a-form-item has-feedback label="Birthday" name="birthday">
        <a-date-picker
          v-model:value="formState.birthday"
          :allowClear="false"
          style="display: flex; color: black !important"
        />
      </a-form-item>

      <a-form-item has-feedback label="Gender" name="gender">
        <a-radio-group v-model:value="formState.gender">
          <a-radio :value="0">Female</a-radio>
          <a-radio :value="1">Male</a-radio>
        </a-radio-group>
      </a-form-item>

      <a-form-item has-feedback label="University" name="university">
        <a-input
          v-model:value="formState.university"
          placeholder="Hogwarts School of Witchcraft and Wizardry"
        />
      </a-form-item>
    </a-form>
  </a-modal>
</template>

<script>
import { API_URL } from "@/utils/constans";
import { message } from "ant-design-vue";
import axios from "axios";
import dayjs from "dayjs";
import { reactive } from "vue";

const DEFAULT_FORM_STATE = {
  type: null,
  visible: false,

  id: null,
  first_name: null,
  last_name: null,
  phone: null,
  email: null,
  birthday: null,
  gender: null,
  university: null,
};
const formState = reactive({ ...DEFAULT_FORM_STATE });

const resetFormState = () => {
  Object.keys(formState).forEach((key) => {
    formState[key] = DEFAULT_FORM_STATE[key];
  });
};

const validateForm = () => {
  return (
    formState.birthday != "" &&
    formState.last_name?.trim().length > 0 &&
    formState.first_name?.trim().length > 0 &&
    formState.university?.trim().length > 0 &&
    (formState.gender == 0 || formState.gender == 1) &&
    formState.phone?.match(
      /^([ +]*)([0-9-]{3,})([ .-/]*)([0-9-]{3,})([ .-/]*)([0-9-]{4,})$/
    ) &&
    formState.email?.match(/^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+.[a-zA-Z]{2,}$/)
  );
};

const fetchData = async () => {
  const student = { ...formState };
  student.birthday = student.birthday
    ? student.birthday.format("YYYY-MM-DD") + "T00:00:00Z"
    : null;
  var res = {},
    err = {};
  if (formState.id > 0) {
    await axios
      .put(`${API_URL}/api/v1/students/${formState.id}`, student)
      .then((_res) => (res = _res))
      .catch((_err) => (err = _err));
  } else {
    await axios
      .post(`${API_URL}/api/v1/students`, student)
      .then((_res) => (res = _res))
      .catch((_err) => (err = _err));
  }
  if (res?.data?.success) {
    message.success(res?.data?.message).then(() => location.reload());
  } else
    message.error(
      err?.response?.data?.message || err?.message || "SERVER ERROR"
    );
};

const commonResolve = () => Promise.resolve();
const commonReject = (msg) =>
  Promise.reject("Please input the valid " + msg + "!");

const rules = {
  first_name: [
    {
      required: true,
      trigger: "change",
      validator: async () =>
        formState.first_name.trim() === ""
          ? commonReject("First Name")
          : commonResolve(),
    },
  ],
  last_name: [
    {
      required: true,
      trigger: "change",
      validator: async () =>
        formState.last_name.trim() === ""
          ? commonReject("Last Name")
          : commonResolve(),
    },
  ],
  phone: [
    {
      required: true,
      trigger: "change",
      validator: async () =>
        formState.phone.trim() === "" ||
        !formState.phone.match(
          /^([ +]*)([0-9-]{3,})([ .-/]*)([0-9-]{3,})([ .-/]*)([0-9-]{4,})$/
        )
          ? commonReject("Phone")
          : commonResolve(),
    },
  ],
  email: [
    {
      required: true,
      trigger: "change",
      validator: async () =>
        formState.email.trim() === "" ||
        !formState.email.match(
          /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+.[a-zA-Z]{2,}$/
        )
          ? commonReject("Email")
          : commonResolve(),
    },
  ],
  birthday: [
    {
      required: true,
      trigger: "change",
      validator: async () =>
        !formState.birthday ? commonReject("Birthday") : commonResolve(),
    },
  ],
  gender: [
    {
      required: true,
      trigger: "change",
      validator: async () =>
        formState.gender === null ? commonReject("Gender") : commonResolve(),
    },
  ],
  university: [
    {
      required: true,
      trigger: "change",
      validator: async () =>
        formState.university.trim() === ""
          ? commonReject("University")
          : commonResolve(),
    },
  ],
};

const handleOK = () => {
  if (formState.type === "info") {
    formState.type = "edit";
    return;
  }
  if (validateForm()) fetchData();
  else message.error("Please fill the valid form");
};

export default {
  name: "StudentDetail",
  props: {
    model: {
      type: Object,
      required: true,
    },
    reloadData: {
      type: Function,
      required: true,
    },
  },
  data() {
    return {
      formState,
      rules,

      handleOK,
    };
  },
  watch: {
    model: {
      immediate: true,
      deep: true,
      handler(val) {
        resetFormState();
        if (["info", "edit", "create"].includes(val.type)) {
          formState.type = val.type;
          formState.visible = true;
        } else formState.visible = false;
        if (val?.data?.id > 0) {
          Object.keys(formState).forEach((key) => {
            if (key === "birthday") {
              formState[key] = val.data[key]
                ? dayjs(val.data[key], "YYYY-MM-DD")
                : null;
              return;
            }
            if (key === "type" || key === "visible") return;
            formState[key] = val.data[key];
          });
        }
      },
    },
  },
};
</script>

<style scoped></style>
