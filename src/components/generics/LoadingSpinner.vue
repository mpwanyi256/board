<template>
  <div class="loading">
    <v-progress-circular indeterminate color="primary"></v-progress-circular>
  </div>
</template>
<script lang="ts">
import { defineComponent } from "vue";

export default defineComponent({
  name: "LoadingSpinner-component",
});
</script>
<style scoped lang="scss">
.loading {
  height: 100%;
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
