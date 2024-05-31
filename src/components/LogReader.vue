<template>
  <div>
    <h1>{{ props.msg }}</h1>
    <input v-model="searchQuery" placeholder="Search logs..." />
    <table>
      <thead>
      <tr>
        <th>Line Number</th>
        <th>Date</th>
        <th>Time</th>
        <th>Thread</th>
        <th>Level</th>
        <th>Logger</th>
        <th>Message</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="log in filteredLogContent" :key="log.lineNumber">
        <td>{{ log.lineNumber }}</td>
        <td>{{ log.date }}</td>
        <td>{{ log.time }}</td>
        <td>{{ log.thread }}</td>
        <td>{{ log.level }}</td>
        <td>{{ log.logger }}</td>
        <td>{{ log.message }}</td>
      </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'

interface Props {
  msg: string;
}

interface LogEntry {
  lineNumber: number;
  date: string;
  time: string;
  thread: string;
  level: string;
  logger: string;
  message: string;
}

const props = defineProps<Props>()
const parsedLogContent = ref<LogEntry[]>([])
const searchQuery = ref<string>('')

onMounted(async () => {
  try {
    const response = await fetch('/src/assets/myapp-2024-05-30.log.txt'); // Adjust the path if necessary
    if (!response.ok) {
      throw new Error('Network response was not ok');
    }
    const text = await response.text();
    parsedLogContent.value = parseLogFile(text);
  } catch (error) {
    console.error('Failed to read the log file:', error);
  }
})

function parseLogFile(text: string): LogEntry[] {
  const lines = text.split('\n').filter(line => line.trim() !== '');
  return lines.map((line, index) => {
    const datePart = line.substring(0, 10);
    const timePart = line.substring(11, 23);
    const threadMatch = line.match(/\[(.*?)\]/);
    const thread = threadMatch ? threadMatch[1] : '';
    const levelMatch = line.match(/(INFO|DEBUG|WARN|ERROR|TRACE)/);
    const level = levelMatch ? levelMatch[0] : '';
    const loggerMatch = line.match(/(\s[o|s]\.\w+\.\w+\.\w+\s)/);
    const logger = loggerMatch ? loggerMatch[0].trim() : '';
    const messageStart = line.indexOf(' - ', loggerMatch ? loggerMatch.index : 0) + 3;
    const message = line.substring(messageStart).trim();
    return {
      lineNumber: index + 1,
      date: datePart,
      time: timePart,
      thread,
      level,
      logger,
      message
    };
  });
}

const filteredLogContent = computed(() => {
  if (!searchQuery.value) {
    return parsedLogContent.value;
  }
  const query = searchQuery.value.toLowerCase();
  return parsedLogContent.value.filter(log =>
      log.lineNumber.toString().includes(query) ||
      log.date.toLowerCase().includes(query) ||
      log.time.toLowerCase().includes(query) ||
      log.thread.toLowerCase().includes(query) ||
      log.level.toLowerCase().includes(query) ||
      log.logger.toLowerCase().includes(query) ||
      log.message.toLowerCase().includes(query)
  );
});
</script>

<style scoped>
table {
  width: 100%;
  border-collapse: collapse;
}

th, td {
  border: 1px solid #ddd;
  padding: 8px;
}

th {
  background-color: #f2f2f2;
}

input {
  margin-bottom: 10px;
  padding: 5px;
  width: 100%;
  box-sizing: border-box;
}
</style>
