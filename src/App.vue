<template>
  <div
    :class="[
      'min-h-screen flex items-center justify-center p-4',
      darkMode ? 'bg-gray-800' : 'bg-gradient-to-r from-blue-500 to-purple-500'
    ]"
  >
    <div
      :class="[
        'bg-white p-6 rounded-lg shadow-2xl w-full max-w-md',
        darkMode ? 'bg-gray-900 text-white' : 'text-gray-800'
      ]"
    >
      <h1 class="text-3xl font-bold text-center mb-6">Pomodoro Timer</h1>
      <div class="flex justify-between mb-4">
        <button
          @click="openSettings"
          class="p-2 rounded-lg shadow-lg transition transform hover:scale-105"
        >
          <svg
            width="24"
            height="24"
            xmlns="http://www.w3.org/2000/svg"
            fill-rule="evenodd"
            clip-rule="evenodd"
          >
            <path
              d="M12 8.666c-1.838 0-3.333 1.496-3.333 3.334s1.495 3.333 3.333 3.333 3.333-1.495 3.333-3.333-1.495-3.334-3.333-3.334m0 7.667c-2.39 0-4.333-1.943-4.333-4.333s1.943-4.334 4.333-4.334 4.333 1.944 4.333 4.334c0 2.39-1.943 4.333-4.333 4.333m-1.193 6.667h2.386c.379-1.104.668-2.451 2.107-3.05 1.496-.617 2.666.196 3.635.672l1.686-1.688c-.508-1.047-1.266-2.199-.669-3.641.567-1.369 1.739-1.663 3.048-2.099v-2.388c-1.235-.421-2.471-.708-3.047-2.098-.572-1.38.057-2.395.669-3.643l-1.687-1.686c-1.117.547-2.221 1.257-3.642.668-1.374-.571-1.656-1.734-2.1-3.047h-2.386c-.424 1.231-.704 2.468-2.099 3.046-.365.153-.718.226-1.077.226-.843 0-1.539-.392-2.566-.893l-1.687 1.686c.574 1.175 1.251 2.237.669 3.643-.571 1.375-1.734 1.654-3.047 2.098v2.388c1.226.418 2.468.705 3.047 2.098.581 1.403-.075 2.432-.669 3.643l1.687 1.687c1.45-.725 2.355-1.204 3.642-.669 1.378.572 1.655 1.738 2.1 3.047m3.094 1h-3.803c-.681-1.918-.785-2.713-1.773-3.123-1.005-.419-1.731.132-3.466.952l-2.689-2.689c.873-1.837 1.367-2.465.953-3.465-.412-.991-1.192-1.087-3.123-1.773v-3.804c1.906-.678 2.712-.782 3.123-1.773.411-.991-.071-1.613-.953-3.466l2.689-2.688c1.741.828 2.466 1.365 3.465.953.992-.412 1.082-1.185 1.775-3.124h3.802c.682 1.918.788 2.714 1.774 3.123 1.001.416 1.709-.119 3.467-.952l2.687 2.688c-.878 1.847-1.361 2.477-.952 3.465.411.992 1.192 1.087 3.123 1.774v3.805c-1.906.677-2.713.782-3.124 1.773-.403.975.044 1.561.954 3.464l-2.688 2.689c-1.728-.82-2.467-1.37-3.456-.955-.988.41-1.08 1.146-1.785 3.126"
            />
          </svg>
        </button>
        <button
          @click="toggleDarkMode"
          class="p-2 rounded-lg shadow-lg transition transform hover:scale-105"
        >
          {{ darkMode ? 'Light Mode' : 'Dark Mode' }}
        </button>
      </div>
      <div class="flex justify-between mb-4">
        <button @click="setMode('work')" :class="buttonClass('work')">Pomodoro</button>
        <button @click="setMode('shortBreak')" :class="buttonClass('shortBreak')">
          Short Break
        </button>
        <button @click="setMode('longBreak')" :class="buttonClass('longBreak')">Long Break</button>
      </div>
      <div class="text-center mb-6">
        <div class="text-6xl font-bold">{{ formattedTime }}</div>
        <div class="text-xl">{{ timerType }}</div>
      </div>
      <div class="flex justify-center space-x-4 mb-4">
        <button
          @click="startTimer"
          :disabled="isRunning"
          class="bg-blue-500 text-white px-4 py-2 rounded-lg shadow-lg transition transform hover:scale-105 disabled:opacity-50"
        >
          Start
        </button>
        <button
          @click="pauseTimer"
          :disabled="!isRunning"
          class="bg-yellow-500 text-white px-4 py-2 rounded-lg shadow-lg transition transform hover:scale-105 disabled:opacity-50"
        >
          Pause
        </button>
        <button
          @click="resetTimer"
          class="bg-red-500 text-white px-4 py-2 rounded-lg shadow-lg transition transform hover:scale-105"
        >
          Reset
        </button>
      </div>
      <div class="mb-4">
        <h2 class="text-xl font-bold mb-2">Task List</h2>
        <input
          v-model="newTask"
          @keyup.enter="addTask"
          placeholder="Add a new task"
          class="w-full p-2 border rounded-lg mb-2"
        />
        <select
          v-model="taskPriority"
          @change="setPriority"
          class="w-full p-2 border rounded-lg mb-2"
        >
          <option value="normal">Normal</option>
          <option value="high">High</option>
          <option value="low">Low</option>
        </select>
        <ul>
          <li
            v-for="(task, index) in sortedTasks"
            :key="index"
            class="flex justify-between items-center"
          >
            <span :class="task.priority">{{ task.text }}</span>
            <button @click="removeTask(index)" class="text-red-500">Remove</button>
          </li>
        </ul>
      </div>
      <div class="mb-4">
        <h2 class="text-xl font-bold mb-2">Task History</h2>
        <ul>
          <li
            v-for="(hist, index) in taskHistory"
            :key="index"
            class="flex justify-between items-center"
          >
            <span>{{ hist }}</span>
          </li>
        </ul>
      </div>
      <div class="mb-4">
        <h2 class="text-xl font-bold mb-2">Statistics</h2>
        <p>Total Sessions: {{ totalSessions }}</p>
        <p>Completed Sessions: {{ completedSessions }}</p>
      </div>
    </div>

    <div
      v-if="isSettingsOpen"
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50"
    >
      <div class="bg-white p-6 rounded-lg shadow-lg w-full max-w-md">
        <h2 class="text-2xl font-bold mb-4">Settings</h2>
        <label class="block mb-2">
          Work Duration (minutes)
          <input
            v-model.number="workDuration"
            type="number"
            min="1"
            class="w-full mt-1 p-2 border rounded-lg"
          />
        </label>
        <label class="block mb-2">
          Short Break Duration (minutes)
          <input
            v-model.number="shortBreakDuration"
            type="number"
            min="1"
            class="w-full mt-1 p-2 border rounded-lg"
          />
        </label>
        <label class="block mb-4">
          Long Break Duration (minutes)
          <input
            v-model.number="longBreakDuration"
            type="number"
            min="1"
            class="w-full mt-1 p-2 border rounded-lg"
          />
        </label>
        <div class="flex justify-end space-x-4">
          <button
            @click="saveSettings"
            class="px-4 py-2 bg-blue-500 text-white rounded-lg shadow-lg transition transform hover:scale-105"
          >
            Save
          </button>
          <button
            @click="closeSettings"
            class="px-4 py-2 bg-red-500 text-white rounded-lg shadow-lg transition transform hover:scale-105"
          >
            Cancel
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      workDuration: 25,
      shortBreakDuration: 5,
      longBreakDuration: 15,
      timer: null,
      timeLeft: 0,
      isRunning: false,
      timerType: 'work',
      isSettingsOpen: false,
      darkMode: false,
      tasks: [],
      newTask: '',
      taskPriority: 'normal',
      taskHistory: [],
      totalSessions: 0,
      completedSessions: 0
    }
  },
  computed: {
    formattedTime() {
      const minutes = Math.floor(this.timeLeft / 60)
      const seconds = this.timeLeft % 60
      return `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`
    },
    sortedTasks() {
      return this.tasks.slice().sort((a, b) => {
        const priorityOrder = { high: 1, normal: 2, low: 3 }
        return priorityOrder[a.priority] - priorityOrder[b.priority]
      })
    }
  },
  methods: {
    startTimer() {
      this.isRunning = true
      this.timeLeft = this[this.timerType + 'Duration'] * 60
      this.timer = setInterval(() => {
        if (this.timeLeft > 0) {
          this.timeLeft -= 1
        } else {
          this.isRunning = false
          clearInterval(this.timer)
          this.completedSessions++
          this.totalSessions++
          this.logSession()
          alert(
            `${this.timerType === 'work' ? 'Work' : this.timerType === 'shortBreak' ? 'Short Break' : 'Long Break'} session ended!`
          )
        }
      }, 1000)
    },
    pauseTimer() {
      this.isRunning = false
      clearInterval(this.timer)
    },
    resetTimer() {
      this.pauseTimer()
      this.timeLeft = this[this.timerType + 'Duration'] * 60
    },
    setMode(mode) {
      this.timerType = mode
      this.resetTimer()
    },
    openSettings() {
      this.isSettingsOpen = true
    },
    closeSettings() {
      this.isSettingsOpen = false
    },
    saveSettings() {
      this.closeSettings()
      this.resetTimer()
    },
    toggleDarkMode() {
      this.darkMode = !this.darkMode
    },
    buttonClass(mode) {
      return this.timerType === mode
        ? 'bg-blue-500 text-white px-4 py-2 rounded-lg shadow-lg'
        : 'bg-gray-300 text-black px-4 py-2 rounded-lg shadow-lg'
    },
    addTask() {
      if (this.newTask.trim() !== '') {
        this.tasks.push({ text: this.newTask.trim(), priority: this.taskPriority })
        this.newTask = ''
        this.taskPriority = 'normal'
      }
    },
    removeTask(index) {
      this.tasks.splice(index, 1)
    },
    logSession() {
      const sessionType =
        this.timerType === 'work'
          ? 'Work'
          : this.timerType === 'shortBreak'
            ? 'Short Break'
            : 'Long Break'
      this.taskHistory.push(
        `${sessionType} session completed at ${new Date().toLocaleTimeString()}`
      )
    },
    checkAutoDarkMode() {
      const hours = new Date().getUTCHours() + 7 // Waktu Indonesia Barat (WIB)
      if (hours >= 18 || hours < 6) {
        this.darkMode = true // Aktifkan mode gelap otomatis
      } else {
        this.darkMode = false // Nonaktifkan mode gelap otomatis
      }
    }
  },
  mounted() {
    this.resetTimer()
    this.checkAutoDarkMode() // Cek mode gelap otomatis saat aplikasi dimuat
    window.addEventListener('load', this.checkAutoDarkMode) // Cek mode gelap saat halaman dimuat
    setInterval(this.checkAutoDarkMode, 60000) // Periksa setiap menit
  },
  beforeUnmount() {
    window.removeEventListener('load', this.checkAutoDarkMode)
  }
}
</script>

<style>
/* Tambahkan gaya tambahan jika diperlukan */
.high {
  color: red;
  font-weight: bold;
}
.normal {
  color: black;
}
.low {
  color: green;
}
</style>
