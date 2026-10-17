<template>
  <div :id="containerID"></div>
</template>

<script>
export default {
  name: 'Vis3',
  props: {
    onLoad: Function,
    domainName: String,
    ID: String
  },
  data() {
    return { containerID: '' }
  },
  mounted() {
    // Set after mount, not in data(): under SSR the server's id would differ from the
    // client's, hydration keeps the server one, and the embed would miss its div.
    this.containerID = Math.random().toString(36).substring(2)
    this.$nextTick(() => {
      const callback = `vis_callback_${this.containerID}`
      window[callback] = (result, vis) => {
        if (this.cancelled) {
          // Unmounted while the embed was still loading
          delete window[callback]
          return result.api.actions.Destroy && result.api.actions.Destroy()
        }
        this.api = result.api
        this.onLoad && this.onLoad(result, vis)
      }

      this.script = document.createElement('script')
      this.script.async = true
      this.script.src = `https://${this.domainName || ''}/${this.ID || ''}/embed?container=${this.containerID}&callback=${callback}`
      document.body.appendChild(this.script)
    })
  },
  methods: {
    unload() {
      this.cancelled = true
      this.script && this.script.remove()
      if (!this.api) return
      // Removing the script frees nothing: the player it built lives on in window.Vis
      this.api.actions.Destroy && this.api.actions.Destroy()
      delete window[`vis_callback_${this.containerID}`]
    }
  },
  // Vue 2 calls beforeDestroy, Vue 3 beforeUnmount
  beforeDestroy() { this.unload() },
  beforeUnmount() { this.unload() }
}
</script>
