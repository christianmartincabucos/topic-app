<script setup>
import { ref, toRefs } from 'vue'
import { FwbButton, FwbTooltip, FwbModal } from 'flowbite-vue'

const props = defineProps({
  topics: Array,
  persons: Array,
})

const { topics, persons } = toRefs(props);
const count = ref(0)
const textComment = ref('');
const textTopic = ref('');
const showModalEdit = ref(false);
const showModalDelete = ref(false);
const topic = ref([]);
const formatDate = (date) => {
  return new Date(date).toUTCString();
}

topics.value ?? localStorage.setItem('topics', JSON.stringify(topics.value));
persons.value ?? localStorage.setItem('persons', JSON.stringify(persons.value));

const submitTopic = () => {
  if (textTopic.value.trim() !== '') {
    topics.value.filter(data => {
      if (data.guid === topic.value.guid) {
        data.name = textTopic.value;
      }
    })
    localStorage.setItem('topics', JSON.stringify(topics.value));
    showModalEdit.value = false;
  }
}
const topicLength = (length) => {
  if (length) {
    return `${length} ${length > 1 ? ' comments':' comment'}`;
  }
  return 'comment';
}

const editTopic = (data) => {
  showModalEdit.value = true;
  topic.value = data
  textTopic.value = topic.value.name;
}

const closeModal = () => {
  showModalEdit.value = false
}
const showDelete = (data) => {
  topic.value = data
  showModalDelete.value = true;
}

const deleteTopic = () => {
  topics.value.filter(data => {
    if (data.guid === topic.value.guid) {
      topics.value.splice(data, 1);
    }
  })
  localStorage.setItem('topics', JSON.stringify(topics.value));
  showModalDelete.value = false;
}

const addComment = (topicId) => {
  if (textComment.value) {
    topics.value.filter(topic => {
      if (topic.guid === topicId) {
        topic.comments.push({
          by: 'gt',
          comment: textComment.value,
          date: new Date().toUTCString()
        })
        textComment.value = '';
      }
    });
    localStorage.setItem('topics', JSON.stringify(topics.value));
  }
}
const deleteComment = (topicId, commentIndex) => {
  if( confirm('Are you sure you want to delete this comment') === true ) {
    topics.value.filter(topic => {
      if (topic.guid == topicId) {
        topic.comments.splice(commentIndex, 1);
      }
    });
    localStorage.setItem('topics', JSON.stringify(topics.value));
  }
}
</script>

<template>

  <div class="max-w-5xl px-4 pb-5 sm:px-6 lg:mx-auto py-10">
    <h1 class="text-5xl font-medium text-gray-600">Browse Topics</h1>
    <div class="mt-3 overflow-hidden bg-white rounded-xl transition duration-300 shadow-md text-slate-500 shadow-slate-200"
      v-for="(topic, index) in topics" :key="topic.guid+index">
      <div class="p-6">
        <header class="flex justify-between gap-4 mb-4">
          <div>
            <h3 class="text-xl font-semibold text-slate-700 first-letter:uppercase">{{ topic.name }}</h3>
          </div>
          <fwb-tooltip trigger="click" theme="light" placement="left">
            <template #trigger>
              <button>
                <svg class="w-6 h-6 text-gray-800 dark:text-white" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                  <path stroke="currentColor" stroke-linecap="round" stroke-width="2" d="M12 6h.01M12 12h.01M12 18h.01"/>
                </svg>
              </button>
            </template>
            <template #content>
              <button class="text-xs hover:text-blue-500" type="button" @click.prevent="editTopic(topic)">
                Edit
              </button><br>
              <button class="text-xs hover:text-blue-500" type="button" @click.prevent="showDelete(topic)">
                Delete 
              </button>
            </template>
          </fwb-tooltip>
        </header>
      </div>
    
      <section class="w-full bg-white divide-y rounded shadow-md divide-slate-200 shadow-slate-200">
        <details class="p-4 group">
          <summary class="[&::-webkit-details-marker]:hidden relative pr-8 font-medium list-none cursor-pointer text-slate-700 focus-visible:outline-none transition-colors duration-300 group-hover:text-slate-900 ">
            <span class="text-gray-500">{{ topicLength(topic.comments.length) }}</span>
            <svg xmlns="http://www.w3.org/2000/svg" class="absolute right-0 w-6 h-6 transition duration-300 top-1 stroke-slate-700 shrink-0 group-open:rotate-180" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.5" aria-labelledby="title-ac13 desc-ac13">
              <path stroke-linecap="round" stroke-linejoin="round" d="M9 5 5 1 1 5"/>
            </svg>

          </summary>
          <ul aria-label="User feed" role="feed" class="relative flex flex-col gap-12 py-5 pl-8 before:absolute before:top-0 before:left-8 before:h-full before:border before:-translate-x-1/2 before:border-slate-200 before:border-dashed after:absolute after:top-6 after:left-8 after:bottom-6 after:border after:-translate-x-1/2 after:border-slate-200 ">
            <li role="article" class="relative pl-8" v-for="(comm, i) in topic.comments" :key="comm.by+i">
              <div class="flex justify-start gap-4">
                <a href="#" class="absolute z-10 inline-flex items-center justify-center w-8 h-8 rounded-full -left-4 ring-2 bg-gray-300 ring-gray-300">
                  <h4 width="48" height="48" class="max-w-full rounded-full">{{ comm.by }}</h4>
                </a>
                <h4 class="text-slate-500">{{ comm.comment }}</h4>
              </div>
              <button @click="deleteComment(topic.guid, i)" type="button" class="pr-4 hover:text-black text-xs font-normal text-slate-400">Delete</button>
              <small><span class="text-xs font-normal text-slate-400"> {{ formatDate(comm.date) }}</span></small>
            </li>
          </ul>
          <div class="flex justify-between relative pl-10">
            <textarea v-model="textComment" type="text" placeholder="Write your comment" rows="2" class="relative w-full px-4 py-2 text-sm transition-all border rounded-xl outline-none focus-visible:outline-none peer border-slate-200 text-slate-500 autofill:bg-white invalid:border-pink-500 invalid:text-pink-500 focus:border-gray-500 focus:outline-none invalid:focus:border-pink-500 disabled:cursor-not-allowed disabled:bg-slate-50 disabled:text-slate-400"></textarea>
            <button @click="addComment(topic.guid)" class="inline-flex items-center self-center justify-center h-12 gap-2 px-6 text-sm font-medium tracking-wide transition duration-300 rounded-full focus-visible:outline-none justify-self-center whitespace-nowrap disabled:shadow-none">
              <span class="relative only:-mx-6">
                <span class="sr-only">Button description</span>
                <svg class="w-12 h-12 text-gray-500 dark:text-white" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" viewBox="0 0 24 24">
                  <path fill-rule="evenodd" d="M10.271 5.575C8.967 4.501 7 5.43 7 7.12v9.762c0 1.69 1.967 2.618 3.271 1.544l5.927-4.881a2 2 0 0 0 0-3.088l-5.927-4.88Z" clip-rule="evenodd"/>
                </svg>
              </span>
            </button>
        </div>
        </details>
      </section>
    </div>
  </div>
  <fwb-modal v-if="showModalEdit" @close="closeModal">
    <template #header>
      <div class="flex items-center text-lg">
        Edit Topic
      </div>
    </template>
    <template #body>
      <div class="relative">
        <textarea type="text" v-model="textTopic" placeholder="Write your message" rows="3" class="relative w-full px-4 py-2 text-sm placeholder-transparent transition-all border rounded outline-none focus-visible:outline-none peer border-slate-200 text-slate-500 autofill:bg-white invalid:border-pink-500 invalid:text-pink-500 focus:border-gray-300 focus:outline-none invalid:focus:border-pink-500 disabled:cursor-not-allowed disabled:bg-slate-50 disabled:text-slate-400"></textarea>
        <label for="id-01" class="cursor-text peer-focus:cursor-default absolute left-2 -top-2 z-[1] px-2 text-xs text-slate-400 transition-all before:absolute before:top-0 before:left-0 before:z-[-1] before:block before:h-full before:w-full before:bg-white before:transition-all peer-placeholder-shown:top-2.5 peer-placeholder-shown:text-sm peer-required:after:text-pink-500 peer-required:after:content-['\00a0*'] peer-invalid:text-pink-500 peer-focus:-top-2 peer-focus:text-xs peer-focus:text-emerald-500 peer-invalid:peer-focus:text-pink-500 peer-disabled:cursor-not-allowed peer-disabled:text-slate-400 peer-disabled:before:bg-transparent">
          Update your topic
        </label>
      </div>
      <button @click="submitTopic" class="inline-flex items-center justify-center h-8 gap-2 px-4 text-xs font-medium tracking-wide text-white transition duration-300 rounded-full focus-visible:outline-none whitespace-nowrap bg-gray-500 hover:bg-gray-600 focus:bg-gray-700 disabled:cursor-not-allowed disabled:border-gray-300 disabled:bg-gray-300 disabled:shadow-none">
        <span>Submit</span>
      </button>
    </template>
  </fwb-modal>
  <div v-if="showModalDelete" class="fixed top-0 left-0 z-20 flex items-center justify-center w-screen h-screen bg-slate-300/20 backdrop-blur-sm" aria-labelledby="header-5a content-5a" aria-modal="true" tabindex="-1" role="dialog">
    <!-- Modal -->
    <div class="flex max-h-[90vh] max-w-xs flex-col gap-6 overflow-hidden rounded bg-white p-6 text-slate-500 shadow-xl shadow-slate-700/10text-center" id="modal" role="document">
      <!-- Modal header -->
      <header id="header-5a" class="flex flex-col items-center gap-4">
        <svg xmlns="http://www.w3.org/2000/svg" class="w-8 h-8 stroke-pink-500" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.5" role="graphics-symbol" aria-labelledby="title-21 desc-21">
          <title id="title-21">Icon title</title>
          <desc id="desc-21">
            A more detailed description of the icon
          </desc>
          <path stroke-linecap="round" stroke-linejoin="round" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
        </svg>
        <h3 class="flex-1 text-xl font-medium text-slate-700">Delete Topic?</h3>
      </header>
      <!-- Modal body -->
      <div id="content-5a" class="flex-1 overflow-auto">
        <p>After deleting the topic, recovery will not be possible</p>
      </div>
      <!-- Modal actions -->
      <div class="flex justify-start gap-2">
        <button @click="deleteTopic(index)" class="inline-flex items-center justify-center flex-1 h-10 gap-2 px-5 text-sm font-medium tracking-wide text-white transition duration-300 rounded whitespace-nowrap bg-gray-500 hover:bg-gray-600 focus:bg-gray-700 focus-visible:outline-none disabled:cursor-not-allowed disabled:border-gray-300 disabled:bg-gray-300 disabled:shadow-none">
          <span>Yes, I'm sure</span>
        </button>
        <button @click="() => showModalDelete = false" class="inline-flex items-center justify-center flex-1 h-10 gap-2 px-5 text-sm font-medium tracking-wide transition duration-300 rounded justify-self-center whitespace-nowrap text-gray-500 hover:bg-gray-100 hover:text-gray-600 focus:bg-gray-200 focus:text-gray-700 focus-visible:outline-none disabled:cursor-not-allowed disabled:text-gray-300 disabled:shadow-none disabled:hover:bg-transparent">
          <span>Maybe not</span>
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
</style>
