<template>
  <div
    class="foggy-casper-gallery__wrapper"
    v-stream:mousemove="mouseMove$"
    @mousemove="onMouseMove"
  >
    <div
      ref="card"
      v-for="item in galleryItems"
      :key="item.id"
      @click="onCardClick(item)"
      :style="item.style.card"
      :class="item.class"
      class="absolute column justify-start items-center q-pa-sm bg-white shadow-2 foggy-casper-gallery__card foggy-casper-gallery__card-anim"
    >
      <img
        :src="item.image"
        :style="item.style.image"
        class="foggy-casper-gallery__img foggy-casper-gallery__img-anim"
      />
      <span v-if="item.position.zoom > 0.3" class="q-mt-sm">{{ item.label }}</span>
      {{ item.order }}
    </div>
  </div>
</template>

<script>
import { Subject } from 'rxjs'
import { map } from 'rxjs/operators'

let interval = null
// const debounceTimeoutMS = 0

const getPosition = (zoom, top, left, height, width) => {
  return {
    zoom,
    card: { top, left },
    image: { height, width }
  }
}

const getStyle = (top, left, height, width) => {
  return {
    card: {
      top: `${top}px`,
      left: `${left}px`
    },
    image: {
      height: `${height}px`,
      width: `${width}px`
    }
  }
}

const getRandomPosition = (size) => {
  const top = Math.floor(
    Math.random() * (window.innerHeight - size.height)
  )
  const left = Math.floor(
    Math.random() * (window.innerWidth - size.width)
  )

  return { top, left }
}

const getRandomSize = (size) => {
  const zoom = Math.random()
  const width = Math.floor(zoom * size.width)
  const height = Math.floor(zoom * size.height)

  return { zoom, width, height }
}

const getClassZoom = (zoom) => {
  return 'card-animation__up-down-' +
    (zoom <= 0.3 ? '5' : zoom <= 0.6 ? '10' : '15')
}

export default {
  name: 'FoggyCasperGallery',
  subscriptions () {
    this.mouseMove$ = new Subject()
    return {
      mouseMoves: this.mouseMove$.pipe(
        // debounceTime(debounceTimeoutMS),
        map(e => {
          const { clientX: x, clientY: y, movementX, movementY } = e.event
          return { x, y, movementX, movementY }
        })
      )
    }
  },
  props: {
    items: {
      type: Array,
      required: true
    },
    itemSize: {
      type: Object,
      required: true
    }
  },
  data () {
    return {
      galleryItems: [],
      parallax: {
        direction: -1
      },
      sortOrder: 1,
      mouseMoveHandler: 'on'
    }
  },
  created () {
    this.$watch('mouseMoves', val => {
      this.galleryItems = this.regenerateGalleryParallax(
        this.galleryItems,
        val
      )
    })
  },
  mounted () {
    this.galleryItems = this.regenerateGalleryItems(this.items)
  },
  computed: {},
  beforeDestroy () {
    if (interval) {
      clearInterval(interval)
    }
  },
  methods: {
    /**
     * TODO проверить order
     * @param card
     */
    onCardClick (card) {
      const cardOrder = card.order
      this.galleryItems = this.galleryItems.map(e => {
        if (e.id === card.id) {
          const zoom = 1
          const { top, left } = e
          const { height, width } = this.itemSize
          return {
            ...e,
            position: getPosition(zoom, top, left, height, width),
            style: getStyle(top, left, height, width),
            order: this.galleryItems.length - 1
          }
        } else
        if (e.order === cardOrder) {
          const { top, left } = e
          const { zoom, height, width } = getRandomSize(this.itemSize)
          return {
            ...e,
            position: getPosition(zoom, top, left, height, width),
            style: getStyle(top, left, height, width),
            order: cardOrder
          }
        } else {
          return { ...e }
        }
      }, this)
    },
    onMouseMove (e) {
      // if (this.mouseMoveHandler === 'on') {
      //   const { clientX: x, clientY: y } = e
      //   this.galleryItems = this.regenerateGalleryParallax(this.galleryItems, { x, y })
      // }
    },
    regenerateGalleryParallax (items, mousePoint) {
      const { innerWidth, innerHeight } = window
      const halfWindowHeight = Math.floor(innerHeight / 2)
      const halfWindowWidth = Math.floor(innerWidth / 2)
      const centerPoint = { x: halfWindowWidth, y: halfWindowHeight }

      return items.map(e => {
        const reduceKoef = 4
        const { zoom } = e.position
        const { height, width } = e.position.image
        let { left, top } = e.position.card
        left = Math.round(
          left +
            (this.parallax.direction * (mousePoint.x - centerPoint.x) * zoom) /
              reduceKoef
        )
        top = Math.round(
          top +
            (this.parallax.direction * (mousePoint.y - centerPoint.y) * zoom) /
              reduceKoef
        )
        return {
          ...e,
          style: getStyle(top, left, height, width)
        }
      }, this)
    },
    regenerateGalleryItems (items) {
      return items
        .map(e => {
          const { top, left } = getRandomPosition(this.itemSize)
          const { zoom, height, width } = getRandomSize(this.itemSize)

          return {
            ...e,
            class: getClassZoom(zoom),
            initial: {
              zoom,
              card: { top, left },
              image: { height, width }
            },
            position: getPosition(zoom, top, left, height, width),
            style: getStyle(top, left, height, width)
          }
        }, this)
        .sort(
          (a, b) => this.sortOrder * (b.position.zoom - a.position.zoom),
          this
        )
        .map((e, i) => {
          return {
            ...e,
            order: i
          }
        })
    }
  }
}
</script>

<style scoped>
.foggy-casper-gallery__wrapper {
  height: 100vh;
  width: 100vw;
  overflow: hidden;

  background-color: darkslateblue;
}

.foggy-casper-gallery__card {
  cursor: pointer;
}

.foggy-casper-gallery__card-anim {
  transition: left 1s ease, top 1s ease;
}

.foggy-casper-gallery__img {
  margin: 0;
}

.foggy-casper-gallery__img-anim {
  transition: width 1s ease, height 1s ease;
}

.card-animation__up-down-5 {
  animation: move-up-down-5 6s ease-in-out infinite;
}

.card-animation__up-down-10 {
  animation: move-up-down-10 3s ease-in-out infinite;
}

.card-animation__up-down-15 {
  animation: move-up-down-15 2s ease-in-out infinite;
}

@keyframes move-up-down-5 {
  from,
  to {
    transform: translateY(0px);
  }
  50% {
    transform: translateY(-5px);
  }
}

@keyframes move-up-down-10 {
  from,
  to {
    transform: translateY(0px);
  }
  50% {
    transform: translateY(-10px);
  }
}

@keyframes move-up-down-15 {
  from,
  to {
    transform: translateY(0px);
  }
  50% {
    transform: translateY(-15px);
  }
}
</style>
