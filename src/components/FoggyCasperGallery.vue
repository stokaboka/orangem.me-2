<template>
  <div class="foggy-casper-gallery__wrapper" v-stream:mousemove="mouseMove$">
    --{{mouseMoves}}--
    <div ref="card" v-for="item in galleryItems" :key="item.id" :style="item.style.card" :class="item.class" class="absolute column justify-start items-center q-pa-sm bg-white shadow-2 foggy-casper-gallery__card foggy-casper-gallery__card-anim">
      <img :src="item.image" :style="item.style.image" class="foggy-casper-gallery__img foggy-casper-gallery__img-anim">
      <span class="q-mt-sm">{{item.label}}</span>
    </div>
  </div>
</template>

<script>

import { Subject } from 'rxjs'
import { map } from 'rxjs/operators'

let interval = null

export default {
  name: 'FoggyCasperGallery',
  subscriptions () {
    this.mouseMove$ = new Subject()
    return {
      mouseMoves: this.mouseMove$.pipe(
        map(e => {
          const { clientX, clientY } = e.event
          return {
            x: clientX,
            y: clientY
          }
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
      galleryItems: []
    }
  },
  created () {
    this.$watch('mouseMoves', (val) => {
      this.$refs.card[0].style.setProperty('--mouse-x', `${val.x}px`)
      this.$refs.card[0].style.setProperty('--mouse-y', `${val.y}px`)
    })
  },
  mounted () {
    this.galleryItems = this.regenerateGalleryItems(this.items)

    // interval = setInterval(() => {
    //   this.galleryItems = this.regenerateGalleryItems(this.items)
    // }, 1000)
  },
  computed: {
    // galleryItems () {
    //   return this.items.map(e => {
    //     const top = Math.floor(Math.random() * (window.innerHeight - this.itemSize.height))
    //     const left = Math.floor(Math.random() * (window.innerWidth - this.itemSize.width))
    //     const zoom = Math.random()
    //     const width = Math.floor(zoom * this.itemSize.width)
    //     const height = Math.floor(zoom * this.itemSize.height)
    //     return {
    //       ...e,
    //       style: {
    //         card: {
    //           top: `${top}px`,
    //           left: `${left}px`
    //         },
    //         image: {
    //           height: `${height}px`,
    //           width: `${width}px`
    //         }
    //       }
    //     }
    //   }, this)
    // }
  },
  beforeDestroy () {
    if (interval) {
      clearInterval(interval)
    }
  },
  methods: {
    onMouseMove (event) {
      // console.log(event)
      const { movementX, movementY } = event
      const movement = { x: movementX, y: movementY }

      const { x, y } = event
      const mousePoint = { x, y }

      this.galleryItems = this.regenerateGalleryParallax(this.galleryItems, mousePoint, movement)
    },
    regenerateGalleryParallax (items, mousePoint, movement) {
      const { innerWidth, innerHeight } = window
      const halfWindowHeight = Math.floor(innerHeight / 2)
      const halfWindowWidth = Math.floor(innerWidth / 2)
      const centerPoint = { x: halfWindowWidth, y: halfWindowHeight }

      return items.map(e => {
        // console.log(e)
        const reduceKoef = 4
        const { zoom } = e.position
        const { height, width } = e.position.image
        let { left, top } = e.position.card
        left = left + (mousePoint.x - centerPoint.x) * zoom * Math.sign(left - centerPoint.x) / reduceKoef
        top = top + (mousePoint.y - centerPoint.y) * zoom * Math.sign(top - centerPoint.y) / reduceKoef
        // left = left + movement.x * zoom * Math.sign(left - centerPoint.x)
        // top = top + movement.y * zoom * Math.sign(top - centerPoint.y)
        if (left > 0 && (left + width) <= innerWidth && top > 0 && (top + height) <= innerHeight) {
          return {
            ...e,
            // position: {
            //   zoom,
            //   card: { top, left },
            //   image: { height, width }
            // },
            style: {
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
        } else return { ...e }
      }, this)
    },
    regenerateGalleryItems (items) {
      return items.map(e => {
        const top = Math.floor(Math.random() * (window.innerHeight - this.itemSize.height))
        const left = Math.floor(Math.random() * (window.innerWidth - this.itemSize.width))
        const zoom = Math.random()
        const width = Math.floor(zoom * this.itemSize.width)
        const height = Math.floor(zoom * this.itemSize.height)

        return {
          ...e,
          class: 'card-animation__up-down',
          initial: {
            zoom,
            card: { top, left },
            image: { height, width }
          },
          position: {
            zoom,
            card: { top, left },
            image: { height, width }
          },
          style: {
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
      }, this)
    }
  }
}
</script>

<style scoped>

  .foggy-casper-gallery__wrapper {
    height: 100vh;
    width: 100vw;
    overflow: hidden;
  }

  .foggy-casper-gallery__card {
    --mouse-x: 0px;
    --mouse-y: 0px;

    transform: translateX(calc(var(--mouse-x) * 0.2px));
  }

  .foggy-casper-gallery__card-anim {
    transition: left 1s ease, top 10ms ease;
  }

  .foggy-casper-gallery__img {
    margin: 0;
  }

  .foggy-casper-gallery__img-anim {
    transition: width 1s ease, height 10ms ease;
  }

  .card-animation__up-down {
    animation-duration: 3s;
    animation-name: move-up-down;
    animation-iteration-count: infinite;
    animation-timing-function: ease;
  }

  @keyframes move-up-down {
    from, to {
      transform: translateY(0px);
    }

    50% {
      transform: translateY(-5px);
    }
  }
</style>
