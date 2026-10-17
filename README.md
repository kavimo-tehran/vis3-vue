# Vis3 for Vue

Put the Kavimo Vis3 video player in a Vue app with a single component.

- Vue 2.6, Vue 2.7 and Vue 3
- Safe with server-side rendering (e.g. Nuxt)
- Cleans up after itself: when the component unmounts, the player stops downloading and is removed


## Installation

```bash
npm install @kavimo-tehran/vis3-vue
```

`vue` is a peer dependency: the component uses the Vue your app already has.


## Quick start

```vue
<script setup>
import Vis3 from '@kavimo-tehran/vis3-vue'
</script>

<template>
  <Vis3 domainName="stream.domain.com" ID="xxxxxxxxxxxx" />
</template>
```

- `domainName` is your Kavimo stream domain, without `https://`
- `ID` is the ID of the video


## Examples

### Vue 3 (or Vue 2.7): `<script setup>`

```vue
<script setup>
import Vis3 from '@kavimo-tehran/vis3-vue'

const handleLoad = (media) => {
  console.log('Player is ready', media.api.get.Title())
}
</script>

<template>
  <main>
    <h1>My course</h1>
    <Vis3 domainName="stream.domain.com" ID="xxxxxxxxxxxx" :onLoad="handleLoad" />
  </main>
</template>
```

### Vue 2: Options API

```vue
<template>
  <main>
    <h1>My course</h1>
    <Vis3 domainName="stream.domain.com" ID="xxxxxxxxxxxx" :onLoad="handleLoad" />
  </main>
</template>

<script>
import Vis3 from '@kavimo-tehran/vis3-vue'

export default {
  components: { Vis3 },
  methods: {
    handleLoad(media) {
      console.log('Player is ready', media.api.get.Title())
    }
  }
}
</script>
```

### Control the player with your own buttons

Keep `media.api` from `onLoad`, then call it whenever you need:

```vue
<script setup>
import Vis3 from '@kavimo-tehran/vis3-vue'

let player = null
const handleLoad = (media) => { player = media.api }
</script>

<template>
  <Vis3 domainName="stream.domain.com" ID="xxxxxxxxxxxx" :onLoad="handleLoad" />

  <button @click="player?.actions.Play()">Play</button>
  <button @click="player?.actions.Pause()">Pause</button>
  <button @click="player?.actions.Seek(60)">Jump to 1:00</button>
</template>
```

### Listen to playback events

```js
const handleLoad = (media) => {
  const api = media.api

  api.events.OnPlay(() => console.log('Playing'))
  api.events.OnPause(() => console.log('Paused'))
  api.events.OnEnded(() => console.log('Finished'))

  api.events.OnTimeupdate(() => {
    console.log('Now at second', api.get.CurrentTime())
  })
}
```

### Save the viewer's progress

```js
const handleLoad = (media) => {
  const api = media.api

  api.events.OnEnded(() => {
    fetch('/api/progress', {
      method: 'POST',
      body: JSON.stringify({ video: api.get.ID(), done: true }),
    })
  })
}
```

### Show another video

The props are read when the component mounts. Give it a `key`, so it rebuilds when the video changes:

```vue
<script setup>
import { ref } from 'vue'
import Vis3 from '@kavimo-tehran/vis3-vue'

const videoID = ref('xxxxxxxxxxxx')
</script>

<template>
  <Vis3 :key="videoID" :ID="videoID" domainName="stream.domain.com" />
  <button @click="videoID = 'yyyyyyyyyyyy'">Next lesson</button>
</template>
```

### Watermark (DRM text)

Show the viewer's details over the video, so a screen recording can be traced back:

```js
const handleLoad = (media) => {
  media.api.DRMText({
    text: ['user@example.com', '09123456789'],
    time: '2.5',        // seconds between moves (optional)
    color: '#FFFFFF',   // (optional)
    fontSize: '13px',   // px or % (optional)
    fontStyle: 'bold',  // normal, bold or italic (optional)
    fontName: 'arial',  // (optional)
    opacity: '0.4',     // (optional)
  })
}
```


## Props

| Prop         | Type     | Required | Description                                              |
| ------------ | -------- | -------- | -------------------------------------------------------- |
| `domainName` | String   | Yes      | Your stream domain, e.g. `stream.domain.com`             |
| `ID`         | String   | Yes      | The video ID                                             |
| `onLoad`     | Function | No       | Called once the player is ready, with the `media` object |


## The `media` object

`onLoad` receives `media`. Everything you can do with the player is on `media.api`.

### `api.get`: read information

| Method              | Returns                      |
| ------------------- | ---------------------------- |
| `get.ID()`          | The video ID                 |
| `get.Title()`       | The video title              |
| `get.Duration()`    | Length, in seconds           |
| `get.CurrentTime()` | Current position, in seconds |

### `api.is`: check the state

| Method            | Returns                    |
| ----------------- | -------------------------- |
| `is.Play()`       | `true` while playing       |
| `is.Pause()`      | `true` while paused        |
| `is.Mute()`       | `true` while muted         |
| `is.Fullscreen()` | `true` while in fullscreen |

### `api.actions`: control playback

| Method                     | Does                                           |
| -------------------------- | ---------------------------------------------- |
| `actions.Play()`           | Plays                                          |
| `actions.Pause()`          | Pauses                                         |
| `actions.Seek(seconds)`    | Jumps to a position                            |
| `actions.Forward(seconds)` | Goes forward (5 seconds if you leave it out)   |
| `actions.Rewind(seconds)`  | Goes back (5 seconds if you leave it out)      |
| `actions.Destroy()`        | Removes the player and stops all its downloads |

You don't need `Destroy()` with `<Vis3 />`: unmounting the component already does it.

### `api.events`: listen to playback

| Method                    | Your function runs when…                       |
| ------------------------- | ---------------------------------------------- |
| `events.OnPlay(fn)`       | Playback starts or resumes                     |
| `events.OnPause(fn)`      | Playback pauses                                |
| `events.OnEnded(fn)`      | The video reaches the end                      |
| `events.OnTimeupdate(fn)` | The position changes; read `get.CurrentTime()` |

### `api.DRMText(options)`: watermark

| Option      | Example                | Description                      |
| ----------- | ---------------------- | -------------------------------- |
| `text`      | `['user@example.com']` | Lines of text to show (required) |
| `time`      | `'2.5'`                | Seconds between moves            |
| `color`     | `'#FFFFFF'`            | Text color                       |
| `fontSize`  | `'13px'`               | Size in px or %                  |
| `fontStyle` | `'bold'`               | `normal`, `bold` or `italic`     |
| `fontName`  | `'arial'`              | Font family                      |
| `opacity`   | `'0.4'`                | From `0` to `1`                  |


## Good to know

- **Unmounting** removes the player (`beforeDestroy` in Vue 2, `beforeUnmount` in Vue 3): it stops downloading and leaves nothing behind. No cleanup code is needed.
- **`<keep-alive>` / `<KeepAlive>`**: a cached component is not unmounted, so the player is kept. While it is off the page it downloads nothing, and it carries on when it comes back.
- **Server-side rendering (e.g. Nuxt)**: the server renders an empty `<div>`, and the player is built in the browser.


## Troubleshooting

| Problem                          | Fix                                                                 |
| -------------------------------- | ------------------------------------------------------------------- |
| Nothing shows up                 | Check `domainName` (no `https://`, no trailing `/`) and `ID`        |
| Changing `ID` shows the old video | Add `:key="ID"` so the component rebuilds (see *Show another video*) |
| `onLoad` is never called         | Pass a function: `:onLoad="handleLoad"`, not `onLoad="handleLoad"`  |
