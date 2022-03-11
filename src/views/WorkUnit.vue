<template lang="pug">
.view
  WorkUnitInfo(v-if="showWUInfo")
  .emptyContainer(v-else)
    .card.blue-card.d-flex.align-items-center.justify-content-center
      p Currently, there are no work units.
</template>

<script>
import { computed } from "vue";
import useWebSocket from "../composables/useWebSocket";
import WorkUnitInfo from "../components/WorkUnitInfo.vue";

export default {
  name: "WorkUnit",
  components: { WorkUnitInfo },
  setup() {
    const { units } = useWebSocket()

    const showWUInfo = computed(() => units.value.length > 0)

    return { showWUInfo };
  }
}
</script>

<style lang="stylus" scoped>
.emptyContainer
  height 80vh

.blue-card
  height 100%
  background-color lightblue

  @media screen and (max-width 768px)
    height 300px
</style>