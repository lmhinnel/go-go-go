<template>
  <vdt-list />
</template>

<script>
import VdtList from "./components/VdtList.vue";

export default {
  name: "App",
  setup() {
    return {};
  },
  components: {
    VdtList,
  },
};
</script>

<style>
.center {
  display: flex;
  justify-content: center;
  align-items: center;
}
.padding {
  padding: 20px;
}
</style>
