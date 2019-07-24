<template>
  <q-page class="flex">
    <foggy-casper-gallery :items="gallery" :item-size="imageSize"></foggy-casper-gallery>
  </q-page>
</template>

<style>
</style>

<script>
/*
<img alt="Quasar logo" src="~assets/quasar-logo-full.svg">
 */

import FoggyCasperGallery from '../components/FoggyCasperGallery'
export default {
  name: 'PageIndex',
  components: { FoggyCasperGallery },
  data () {
    return {
      imageSize: {
        width: 360,
        height: 640
      }
    }
  },
  computed: {
    gallery () {
      return this.fillItems(5)
    }
  },
  mounted () {},
  methods: {
    fillItems (n) {
      return Array(n)
        .fill(0)
        .map((e, i) => this.itemGenerator(e, i), this)
    },
    itemGenerator (e, i) {
      return {
        id: i,
        label: `${i}`,
        image: this.imageGenerator(i)
      }
    },
    imageGenerator (i) {
      return `http://lorempixel.com/${this.imageSize.width}/${this.imageSize.height}/`
    }
  }
}
</script>
