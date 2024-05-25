<template>
  <a-typography class="center padding">
    <a-typography-title>VDT24 STUDENTS</a-typography-title>
  </a-typography>

  <a-button
    type="primary"
    style="margin: 20px"
    @click="handleModel(null, 'create')"
    >Add student</a-button
  >

  <a-table
    :columns="columns"
    :data-source="studentData"
    :rowKey="(record) => record.id"
    :pagination="false"
  >
    <template #bodyCell="{ column, record }">
      <template v-if="column.key === 'action'">
        <span>
          <a @click="handleModel(record, 'info')">
            <InfoCircleTwoTone two-tone-color="mediumseagreen" />
          </a>
          <a-divider type="vertical" />
          <a @click="handleModel(record, 'edit')">
            <EditTwoTone two-tone-color="gold" style="color: mediumseagreen" />
          </a>
          <a-divider type="vertical" />
          <a-popconfirm
            title="Are you sure?"
            ok-text="Yes"
            cancel-text="No"
            @confirm="handleDelete(record.id)"
          >
            <a>
              <DeleteTwoTone two-tone-color="tomato" />
            </a>
          </a-popconfirm>
        </span>
      </template>
      <template v-else-if="column.key === 'gender'">
        <template v-if="record[column.key] === 0">
          <a-tag color="pink">Female</a-tag>
        </template>
        <template v-else-if="record[column.key] === 1">
          <a-tag color="blue">Male</a-tag>
        </template>
        <template v-else>
          <a-tag color="green">Other</a-tag>
        </template>
      </template>
      <template v-else>{{ record[column.key] }}</template>
    </template>
  </a-table>
  <a-pagination
    v-model:current="currentPagination"
    :total="totalStudent"
    v-model:pageSize="pageSize"
    show-size-changer
    @change="onPaginationChange"
    class="center padding"
  />
  <template>
    <student-detail v-bind:model="model" v-bind:reloadData="reloadData" />
  </template>
</template>

<script>
import { ref, computed } from "vue";
import StudentDetail from "./StudentDetail.vue";
import axios from "axios";
import { API_URL } from "@/utils/constans";
import {
  DeleteTwoTone,
  EditTwoTone,
  InfoCircleTwoTone,
} from "@ant-design/icons-vue";
import { message } from "ant-design-vue";

const columns = computed(() =>
  ["id", "name", "gender", "university", "action"].map((key) => {
    let col = {
      title: key.replace(/^\w/, (c) => c.toUpperCase()),
      dataIndex: key,
      key,
    };
    if (key === "name") {
      col.sorter = (a, b) =>
        a.name.toLowerCase().localeCompare(b.name.toLowerCase());
    }
    if (["id", "gender"].includes(key)) col.width = 100;
    if (["action"].includes(key)) col.width = 200;
    return col;
  })
);

const studentData = ref([]);
const currentPagination = ref(1);
const totalStudent = ref(0);
const pageSize = ref(10);
const model = ref({
  data: {},
  type: null,
});

const fetchData = (limit, offset) =>
  axios
    .get(`${API_URL}/api/v1/students?limit=${limit}&offset=${offset}`)
    .then((res) => {
      studentData.value = [];
      totalStudent.value = res?.data?.count;
      return res?.data?.data || [];
    })
    .then((students) => {
      students.map((st) => {
        studentData.value.push({
          ...st,
          key: st.id,
          name: st.first_name + " " + st.last_name,
        });
      });
    })
    .catch(() => {});

const reloadData = () =>
  fetchData(10, 0).then(() => {
    currentPagination.value = 1;
    pageSize.value = 10;
  });

const onPaginationChange = (page, pageSize) => {
  fetchData(pageSize, (page - 1) * pageSize);
};

const handleModel = (data, type) => {
  model.value.type = type;
  model.value.data = data;
};

const handleDelete = (id) => {
  axios
    .delete(`${API_URL}/api/v1/students/${id}`)
    .then((res) => {
      res?.data?.success
        ? message.success(res?.data?.message)
        : message.error(res?.data?.message || "SERVER ERROR");
    })
    .catch((err) => message.error(err?.message));
};

export default {
  components: { StudentDetail, DeleteTwoTone, EditTwoTone, InfoCircleTwoTone },
  name: "VdtList",
  data() {
    return {
      columns,
      studentData,
      currentPagination,
      totalStudent,
      pageSize,
      model,

      handleModel,
      handleDelete,
      onPaginationChange,
      reloadData,
    };
  },
  watch: {},
  mounted() {
    reloadData();
  },
};
</script>
