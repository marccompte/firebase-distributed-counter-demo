<template>
  <q-page class="flex flex-center column justify-evenly">
    <q-btn color="primary" label="uploadCreds" @click="getCred" />
    <div class="error">{{ error }}</div>
    <div class="counter">Code counter: {{ counter }}</div>
    <div>Last code created in this session: {{ code }}</div>
  </q-page>

</template>

<script>
import { defineComponent, ref, onBeforeMount } from "vue"
import { useStore } from "vuex"
import { initializeApp } from "firebase/app"
import {
  collection,
  doc,
  getDoc,
  getDocs,
  getFirestore,
  addDoc,
  arrayUnion,
  setDoc,
  serverTimestamp,
  writeBatch
} from "firebase/firestore";

export default defineComponent({
  name: "IndexPage",
  setup () {
    const error = ref(null);
    const code = ref("None");
    const alphabet = ref("abcdef");
    const letter = ref("");
    const newDate = ref("");

    const $store = useStore()
    const numShards = 4
    const counter = ref(0)
    initializeApp({
      apiKey: '-',
      authDomain: '-',
      databaseURL: '-',
      projectId: '-',
      storageBucket: '-',
      messagingSenderId: '-',
      appId: '-'
    })
    const db = getFirestore()

    const createCode = () => {
      let numb = Math.round(Math.random() * (9999 - 1000) + 1000);
      letter.value = alphabet.value[Math.floor(Math.random() * 6)];
      code.value = letter.value + numb;
    };

    const createDate = () => {
      let date = new Date();
      let d = date.getDate();
      let mo = date.getMonth();
      let y = date.getFullYear();
      newDate.value = `${y}/${mo + 1}/${d}`;
    };

    const uploadCreds = async (email, number, client) => {
      try {
        createCode();
        createDate();

        const colRef = collection(db, "access", client, "creds");

        addDoc(colRef, {
          code: code.value,
          email: email,
          timeCreated: serverTimestamp(),
          dateCreated: newDate.value,
          hasBeenUsed: false,
          currentNumber: number,
          origNumber: number,
          usedBy: arrayUnion({ user: null, time: null }),
        }).then(response => {
          // If the new document was successfully created, we increment the counter
          if (typeof response !== "undefined") {
            incrementCounter("codes")
          }
        })
      } catch (err) {
        error.value = err.message;
        console.log(error.value);
      }
    };

    onBeforeMount(async () => {
      // get the current counter value from firebase
      const docRef = doc(db, "counters", "codes")
      // If the counter does not exist, create it.
      getDoc(docRef).then(response => {
        if (typeof response === "undefined") {
          createCounter(docRef, db)
        }
      })
      const shardCollection = collection(docRef, "shards")
      // Get the shards and the total count.
      const shards = await getDocs(shardCollection)
      // Update the reactive reference
      counter.value = shards.docs.reduce((previous, current) => {
        return previous + parseInt(current.data().count)
      }, 0)
    })

    const createCounter = (ref, db) => {
      const batch = writeBatch(db)
      // Initialize the counter document
      batch.set(ref, { numShards })
      // Initialize each shard with count=0
      for (let i = 0; i < numShards; i++) {
        const shardRef = doc(collection(ref, "shards"), i.toString())
        batch.set(shardRef, { count: 0 })
      }
      // Commit the write batch
      return batch.commit()
    }

    const incrementCounter = async (name) => {
      const docRef = doc(db, "counters", name)
      const shardCollection = collection(docRef, "shards")
      const shardId = Math.floor(Math.random() * numShards).toString()
      const shardRef = doc(shardCollection, shardId)
      // Get the current value of the shardId
      const shards = await getDocs(shardCollection)
      const shard = shards.docs.find((shard, index) => parseInt(index) === parseInt(shardId))
      // Update the distributed counter on Firebase
      setDoc(shardRef, { count: shard.data().count + 1 }).then(response => {
        // Updated successfully, so we update the counter locally
        counter.value = counter.value + 1
      }).catch(error => {
        console.log(error)
      })
    }
    return {
      code,
      counter,
      error,
      getCred: () => {
        uploadCreds("test@gmail.com", 1, "test_museum")
      }
    }
  }
})
</script>

<style scoped>
.counter {
  font-weight: bold;
  font-size: 4em;
}
.error {
  color: red;
}
</style>
