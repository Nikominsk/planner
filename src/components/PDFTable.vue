<template>
  <div class="pdf-table" ref="planner">

    
  <table ref="lessonTable1">
      <thead>
        <tr>
          <th>Uhrzeit</th>
          <th v-for="day in daysOfWeek" :key="day"  :class="{ onlineColumn: isOfflineDay(day) }">
            <div class="th-entry-container">
              <div class="th-entry-date">11.02.</div>
              <div class="th-entry-content">
                {{ translateToGerman(day) }}
              </div>
              <button v-if="screenshotMode == false">HM</button>
            </div>
          
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(lessonsAtSpecificTime, timeOfDay, timeOfDayIndex) in schedulePlannerJson.schedule" :key="timeOfDay">
          <td class="time-content">{{ translateTimeIntoNumber(timeOfDay) }}</td>
          <Entry v-for="(dayOfWeek, dayOfWeekIndex) in daysOfWeek" :key="dayOfWeekIndex" 
            :entryData="lessonsAtSpecificTime[dayOfWeek]" 
            :hasSamePredecessor="hasSamePredecessor(lessonsAtSpecificTime[dayOfWeek], dayOfWeekIndex, timeOfDayIndex)"
            :numberOfEqualSuccessors="getNumberOfEqualSuccessors(lessonsAtSpecificTime[dayOfWeek], dayOfWeekIndex, timeOfDayIndex)"
            :isScreenshotMode="screenshotMode"
            @onChange="updateSchedule()"></Entry>
        </tr>
      </tbody>
    </table>

      <button @click="exportToPDF">Export to PDF</button>

  </div>
</template>

<script setup>
/* eslint-disable */
import html2canvas from 'html2canvas';
//import jsPDF from 'jspdf';
import 'jspdf-autotable';


import { ref, onMounted, watch, computed } from 'vue';
import SchedulePlanner from '@/assets/data/planner.json'

import Entry from "./Entry.vue";

import { jsPDF } from "jspdf";
import html2pdf from 'html2pdf.js';
import { saveAs } from 'file-saver';

import tableStyle from '@/assets/css/planner.css'

import axios from 'axios'

const emit = defineEmits(['onClose'])
let lessonTable1 = ref(null)
let planner = ref(null)

const eaTable = ref(null);

const schedulePlannerJson = ref({})

const daysOfWeek = ref([])
const hoursOfDay = ref([])

const screenshotMode = ref(false)

const items =  [
  { name: 'Item 1', link: 'https://example.com', linkText: 'Example Link 1' },
  { name: 'Item 2', link: 'https://example.org', linkText: 'Example Link 2' }
]

function exportToPDFJS2() {
  // Create a new jsPDF instance
  const pdf = new jsPDF();

  // Get the HTML content of the table
  const table = planner.value;
  const html = table.outerHTML;

  // Add the HTML content to the PDF
  pdf.fromHTML(html, 15, 15);

  // Save the PDF with a specific name
  pdf.save('table.pdf');
}

function exportToPDFJS1() {
  // Create a new jsPDF instance
  const pdf = new jsPDF();

  // Get the HTML content of the table
  const table = eaTable.value;
  const table2 = planner.value;

  var opt = {
    margin: 0,
    filename: 'myfile.pdf',
    image: { type: 'jpeg', quality: 0.98 },
    html2canvas: { scale: 3 },
    jsPDF: { unit: 'mm', format: 'a4', orientation: 'portrait' }
  };

  //html2pdf().from(table2).set(opt).save();
  // Use html2pdf to convert HTML to PDF
  html2pdf(table2, { jsPDF: pdf }).then((canvas) => {
    // Create a new jsPDF instance
    const pdf = new jsPDF('p', 'mm', 'a4');

    // Convert the canvas to a data URL
    const imgData = canvas.toDataURL('image/png');

    // Add the image to the PDF
    pdf.addImage(imgData, 'PNG', 10, 10);

    // Save the PDF with a specific name
    pdf.save('table.pdf');
  });

}

//https://stackoverflow.com/questions/52305534/changing-file-path-of-js-and-css-files-in-production-build


async function exportToPDF() {
/*
    const formData = new FormData();
    formData.append('html', 'my html text');

    fetch('http://localhost:3000/generate-pdf', {
      method: 'POST',
    })
    .then((response) => {

      if (!response.ok) {
        throw new Error(`HTTP error! Status: ${response.status}`);
      }


    })
    .catch((error) => {
      console.log('Fehler beim Versenden der E-Mail: ' + error);
    });*/
    
    await fetch('http://localhost:3000/generate-pdf', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        // Add any additional headers as needed
      },
      body: JSON.stringify({
        tableHTML: ""//await htmlContentWithStyle(tableHTML)
      })
    })
    .then(async (response) => {

      if (!response.ok) {
        throw new Error(`HTTP error! Status: ${response.status}`);
      }

      const pdfBuffer = await response.arrayBuffer();
      console.log(pdfBuffer);
      const blob = new Blob([pdfBuffer], { type: 'application/pdf' });

      saveAs(blob, "example.pdf");

    })
    .catch((error) => {
      console.log('Fehler beim Versenden der E-Mail: ' + error);
    });
}

onMounted(() => {

  getSchedule()

})

async function updateSchedule() {
    try {
    const response = await fetch('http://localhost:3000/update-schedule', { 
      method: 'POST',
      headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(schedulePlannerJson.value)
    })
    //const content = await response.json();
    if(response.status == 200) {
      console.log('update done')
    } else {
      console.log('error updating')
    }

    // 
    // console.log(schedulePlannerJson.value)

  } catch (error) {
    console.error('Fehler beim Abrufen der modifizierten URLs:', error);
  }
}

async function getSchedule() {
  try {
    const response = await fetch('http://localhost:3000/get-schedule')
    const scheduleData = await  response.json()
    daysOfWeek.value = scheduleData.days;
    hoursOfDay.value = scheduleData.hours;
    schedulePlannerJson.value = scheduleData.schedule;
    console.log(schedulePlannerJson.value)

   // 
   // console.log(schedulePlannerJson.value)

  } catch (error) {
    console.error('Fehler beim Abrufen der modifizierten URLs:', error);
  }

}

function translateToGerman(word) {
  if(word == "monday") {
    return "Montag";
  }
  if (word == "tuesday") {
    return "Dienstag";
  }
  if (word == "wednesday") {
    return "Mittwoch";
  }
  if (word == "thursday") {
    return "Donnerstag";
  }
  if (word == "friday") {
    return "Freitag";
  }
  if (word == "saturday") {
    return "Samstag";
  }
  if (word == "sunday") {
    return "Sonntag";
  }

}


function isOfflineDay(dayOfWeek) {
  return true;
  //return schedulePlannerJson.value.days[dayOfWeek].isOfflineDay;
}

function hasSamePredecessor(currentLesson, currentDayOfWeekIndex, currentTimeOfDayIndex) {
  if(currentTimeOfDayIndex-1 < 0) {
    return false;
  }

  //console.log(currentDayOfWeekIndex)

  const previousTimeOfDay = hoursOfDay.value[currentTimeOfDayIndex-1];

  const lessonsAtTimeOfDay = schedulePlannerJson.value.schedule[previousTimeOfDay];
  const currentDayOfWeek = daysOfWeek.value[currentDayOfWeekIndex]
  
  return lessonsAtTimeOfDay[currentDayOfWeek].name == currentLesson.name;
}

function getNumberOfEqualSuccessors(currentLesson, currentDayOfWeekIndex, currentTimeOfDayIndex) {

  let amountStreak = 0;
  const currentDayOfWeek = daysOfWeek.value[currentDayOfWeekIndex]
  for(var timeOfDayIndex = currentTimeOfDayIndex+1; timeOfDayIndex < hoursOfDay.value.length; timeOfDayIndex++) {
    const hourOfDay = hoursOfDay.value[timeOfDayIndex]
    const lessonsAtTimeOfDay = schedulePlannerJson.value.schedule[hourOfDay];
    if(lessonsAtTimeOfDay[currentDayOfWeek].name == currentLesson.name) {
      amountStreak++;
    } else {
      break;
    }
  }

  if (amountStreak > 0) {
    //console.log('current: ' + hoursOfDay[currentTimeOfDayIndex] + " " + currentDayOfWeek + " " + currentLesson.name)
  }
  return amountStreak;
}

function takeScreenshot2() {
  screenshotMode.value = true;
  setTimeout(() => {

    const table = planner.value; // Access the table element using the ref
    html2canvas(table).then((canvas) => {
      const screenshotUrl = canvas.toDataURL(); // Get the screenshot as a data URL
      const a = document.createElement('a');
      a.href = screenshotUrl;
      a.download = 'lesson_table.png'; // Set a file name for the screenshot
      a.click();
    });

    screenshotMode.value = false;
  }, 500);

}

function translateTimeIntoNumber(time) {
  switch(time) {
    case "eightOClock": return "08:00"
    case "nineOClock": return "09:00"
    case "tenOClock": return "10:00"
    case "elevenOClock": return "11:00"
  }
  return time
}

/*
function createPDF() {
  console.log('create pdf')
  // Create a new jsPDF instance
  const pdf = new jsPDF();

    // Set font style and size for the heading
    pdf.setFont('helvetica');
    pdf.setFontSize(16);

    // Add the heading (H1) to the PDF
    pdf.text('School Lesson Schedule', 10, 20);

  // Take a screenshot of the table using your existing function
  takeScreenshot().then((screenshotUrl) => {
    const pdfWidth = 210; // Standard PDF page width in mm
    const desiredWidth = (pdfWidth * 0.8); // 80% of the PDF width
    // Add the screenshot to the PDF
    //pdf.addImage(screenshotUrl, 'PNG', 0, 0, desiredWidth, desiredWidth * (60 / 90)); // Adjust height accordingly

    // Add formatted text
    const text = `
      This is a formatted text that goes before the table.
      You can add multiple lines and style it as needed.
    `;

    // Set font style and size for the text
    pdf.setFont('times');
    pdf.setFontSize(12);

    // Add the formatted text to the PDF
    pdf.text(text, 10, 80);

    // Generate a table with your data (you can customize the dimensions)
    pdf.autoTable({ html: lessonTable1.value, startY: 120 });

    // Save the PDF with a specific filename
    pdf.save('lesson_schedule.pdf');
  });
}

async function takeScreenshot() {
  const table = planner.value; // Access the table element using the ref
  const canvas = await html2canvas(table);
  return canvas.toDataURL('image/png');
}*/

</script>
  
<style>
  @import '../assets/css/planner.css';

  button {
    border: none;
    background-color: lightblue;
    border-radius: 10px;
    padding: 5px 10px;
    margin-top: 10px;
  }


</style>
  