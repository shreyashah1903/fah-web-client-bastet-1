<template lang="pug">
FoldAnon(v-if="!showLoading")
div(v-if="!showLoading")
  Navbar
  router-view
div(v-else)
  .spinner-border.text-success(role="status")
    span.visually-hidden Loading...
  h3 Connecting to Folding@Home Client
</template>

<script>
import Navbar from './components/Navbar.vue'
import FoldAnon from './components/FoldAnon.vue'
import useWebSocket from './composables/useWebSocket'
import 'bootstrap/dist/css/bootstrap.min.css'
import { onBeforeUnmount } from "vue"

const ws_url = "ws://127.0.0.1:7396/api/websocket"

export default {
  name: 'App',
  components: {
    FoldAnon, Navbar
  },
  setup() {
    const { showLoading, updatePeerConnections, openWebSocket, close} = useWebSocket()
    openWebSocket(ws_url)
    updatePeerConnections()

    onBeforeUnmount(() => {
      close()
    })

    return { showLoading }
  }
}
</script>
