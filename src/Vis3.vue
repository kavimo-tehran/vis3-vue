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
    // Generated here rather than in a lifecycle hook so the div already carries the id
    // on the first render: the embed script resolves its container by id when it runs.
    return {
      containerID: Math.random().toString(36).substring(2)
    }
  },
  mounted() {
    // mounted, not created: in created the template has not been rendered yet, so the
    // container div the embed script goes looking for is not in the document.
    this.visApi = null
    this.visUnmounted = false

    const callbackName = `vis_callback_${this.containerID}`

    // Registered whether or not the caller passed onLoad: this callback is the only
    // handle we ever get on the player, and tearing it down later depends on having it.
    window[callbackName] = (result, vis) => {
      if (this.visUnmounted) {
        // Unmounted while the embed was still loading. The player has just been built
        // into a container that is no longer on the page, so take it straight back down.
        result && result.api && result.api.actions.Destroy && result.api.actions.Destroy()
        return
      }

      this.visApi = result && result.api

      if (typeof this.onLoad === 'function') {
        this.onLoad(result, vis)
      }
    }

    const script = document.createElement('script')
    script.async = true
    script.src = `https://${this.domainName || ''}/${this.ID || ''}/embed?container=${this.containerID}&callback=${callbackName}`

    document.body.appendChild(script)

    this.cleanup = () => {
      this.visUnmounted = true

      // Removing the script tag frees nothing — it has already run, and the player it
      // built lives on in window.Vis. Without Destroy() hls.js keeps filling its buffer,
      // so the fragments of a video the viewer has navigated away from keep downloading.
      this.visApi && this.visApi.actions.Destroy && this.visApi.actions.Destroy()
      this.visApi = null

      script.remove()
      delete window[callbackName]
    }
  },
  // Vue 2 calls beforeDestroy, Vue 3 calls beforeUnmount. Declaring both lets one
  // component work on either major version — the name the running Vue does not know is
  // simply ignored, and cleanup() is written to be safe if it somehow ran twice.
  beforeDestroy() {
    this.cleanup && this.cleanup()
  },
  beforeUnmount() {
    this.cleanup && this.cleanup()
  }
}
</script>
