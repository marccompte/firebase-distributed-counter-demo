<template>
  <q-item
    clickable
    tag="a"
    target="_blank"
    :href="link"
  >
    <q-item-section
      v-if="icon"
      avatar
    >
      <q-icon :name="icon" />
    </q-item-section>

    <q-item-section>
      <q-item-label>{{ title }}</q-item-label>
      <q-item-label caption>{{ caption }}</q-item-label>
    </q-item-section>
  </q-item>
</template>

<script>
// import { defineComponent, onMounted } from 'vue'
import { defineComponent } from 'vue'
// import { useStore } from 'vuex'
// import { initializeApp } from 'firebase/app'
// import { getFirestore, doc, writeBatch, collection, getDocs, setDoc } from 'firebase/firestore'
// import { getFirestore, doc, collection, getDocs, setDoc } from 'firebase/firestore'

export default defineComponent({
  name: 'EssentialLink',
  props: {
    title: {
      type: String,
      required: true
    },

    caption: {
      type: String,
      default: ''
    },

    link: {
      type: String,
      default: '#'
    },

    icon: {
      type: String,
      default: ''
    }
  },
  setup () {
    return {}
  }
})
</script>
