<template>
  <td :style="{ 'background-color': categoryBgColor, 'display': props.hasSamePredecessor ? 'none' : 'table-cell' }"
      :rowspan="numberOfEqualSuccessors+1">
      
      <div id="entry-container" v-if="props.entryData.name == ''"></div>

      <div class="entry-container" v-else-if="!isEditMode">
        <div class="name-text">{{ props.entryData.name }}</div>
        <div class="category-text">{{ props.entryData.category }}</div>
        <div class="location-text">location</div>
        <div class="optional-text">{{ props.entryData.optionalText }}</div>
        <div class="link-text"><a href="http://www.google.com">Click me</a></div>
        <div class="link-text"><a href="http://www.google.com">Click me</a></div>
      </div>
            
      <div class="entry-container" v-else-if="isEditMode">
        <input class="name-text" ref="subjectTitle" :value="props.entryData.name">
        <div class="category-text">{{ props.entryData.category }}</div>
        <div class="location-text">location</div>
        <div class="optional-text">{{ props.entryData.optionalText }}</div>
        <div class="link-text"><a href="http://www.google.com">Click me</a></div>
        <div class="link-text"><a href="http://www.google.com">Click me</a></div>
        <input type="button" value="save" @click="saveEdit()">
      </div>

      <button class="edit-button" v-if="isScreenshotMode != true">ST</button>
      
  </td>
</template>

<script setup>
/* eslint-disable */

import { ref, onMounted, watch } from 'vue';

const props = defineProps({
  entryData: Object,
  hasSamePredecessor: Boolean,
  numberOfEqualSuccessors: Number,
  isScreenshotMode: Boolean
})

const emit = defineEmits(['onChange'])


const categoryBgColor = ref("white")

const isEditMode = ref(true)

const subjectTitle = ref(null)

onMounted(() => {

  if(props.entryData.category == "categoryA") {
    categoryBgColor.value = "rgba(159, 209, 172, 0.6)";
  }
  
  if (props.entryData.category == "categoryB") {
    categoryBgColor.value = "rgba(159, 174, 209, 0.6)";
  }

  if (props.entryData.category == "categoryC") {
    categoryBgColor.value = "rgba(209, 178, 159, 0.6)";
  }

});

function saveEdit() {
  props.entryData.name = subjectTitle.value.value

  emit("onChange")
}


</script>
  
<style>





</style>
  