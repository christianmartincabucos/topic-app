
<template>
  <Topic :topics="topics" :persons="persons"/>
</template>
<script setup>
import { onMounted, nextTick,  ref } from 'vue';
import Topic from './components/Topic.vue'

const topics = ref({});
const persons = ref({});

onMounted(async () => {
  const response = await fetch('https://atillc.blob.core.windows.net/data-collector/icode/test-data/topics.json')
  const data = await response.json();
  topics.value = JSON.parse(localStorage.getItem('topics')) || data.topics;
  persons.value = JSON.parse(localStorage.getItem('persons')) || data.persons;
  console.log(topics.value);
  console.log(persons.value);
})
</script>

<style scoped>
</style>
