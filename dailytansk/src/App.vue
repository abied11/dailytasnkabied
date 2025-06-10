<script setup>
import { ref, computed, watchEffect } from 'vue';

// --- State Aplikasi (Data) ---
const tasks = ref([]); // Array untuk menyimpan semua tugas

// Ambil tugas dari localStorage saat aplikasi dimuat
// watchEffect akan berjalan sekali saat setup dan setiap kali dependensi berubah
watchEffect(() => {
  const storedTasks = localStorage.getItem('tasks');
  if (storedTasks) {
    tasks.value = JSON.parse(storedTasks);
  }
});

// Simpan tugas ke localStorage setiap kali 'tasks' berubah
watchEffect(() => {
  localStorage.setItem('tasks', JSON.stringify(tasks.value));
});

// State untuk form penambahan/pengeditan tugas
const newTaskName = ref('');
const newTaskDescription = ref('');
const newTaskPriority = ref('medium'); // Default medium
const editingTaskId = ref(null); // ID tugas yang sedang diedit

// --- Computed Properties (Data Turunan) ---

// 1. Conditional Rendering: Menampilkan pesan jika tidak ada tugas
const hasTasks = computed(() => tasks.value.length > 0);

// Untuk form, tombol, dll.
const isEditing = computed(() => editingTaskId.value !== null);

// --- Methods (Fungsi Logika) ---

// Menambah atau mengupdate tugas
const handleSubmitTask = () => {
  if (newTaskName.value.trim() === '') {
    alert('Nama tugas tidak boleh kosong!');
    return;
  }

  if (isEditing.value) {
    // Mode Edit
    const taskToUpdate = tasks.value.find(task => task.id === editingTaskId.value);
    if (taskToUpdate) {
      taskToUpdate.name = newTaskName.value.trim();
      taskToUpdate.description = newTaskDescription.value.trim();
      taskToUpdate.priority = newTaskPriority.value;
    }
    editingTaskId.value = null; // Keluar dari mode edit
  } else {
    // Mode Tambah Baru
    const newId = Date.now(); // ID unik sederhana
    tasks.value.push({
      id: newId,
      name: newTaskName.value.trim(),
      description: newTaskDescription.value.trim(),
      priority: newTaskPriority.value, // 'low', 'medium', 'high'
      isCompleted: false,
      createdAt: new Date().toLocaleDateString()
    });
  }

  // Reset form
  newTaskName.value = '';
  newTaskDescription.value = '';
  newTaskPriority.value = 'medium';
};

// Toggle status selesai tugas
const toggleComplete = (id) => {
  const task = tasks.value.find(t => t.id === id);
  if (task) {
    task.isCompleted = !task.isCompleted;
  }
};

// Mengatur form untuk mode pengeditan
const editTask = (id) => {
  const taskToEdit = tasks.value.find(task => task.id === id);
  if (taskToEdit) {
    editingTaskId.value = id;
    newTaskName.value = taskToEdit.name;
    newTaskDescription.value = taskToEdit.description;
    newTaskPriority.value = taskToEdit.priority;
  }
};

// Membatalkan pengeditan
const cancelEdit = () => {
  editingTaskId.value = null;
  newTaskName.value = '';
  newTaskDescription.value = '';
  newTaskPriority.value = 'medium';
};

// Menghapus tugas
const deleteTask = (id) => {
  if (confirm('Apakah Anda yakin ingin menghapus tugas ini?')) {
    tasks.value = tasks.value.filter(task => task.id !== id);
    // Jika tugas yang dihapus sedang diedit, batalkan pengeditan
    if (editingTaskId.value === id) {
      cancelEdit();
    }
  }
};

// Fungsi untuk mendapatkan kelas prioritas (untuk Attribute Bindings)
const getPriorityClass = (priority) => {
  switch (priority) {
    case 'low': return 'priority-low';
    case 'medium': return 'priority-medium';
    case 'high': return 'priority-high';
    default: return '';
  }
};
</script>

<template>
  <div class="container">
    <h1>My Daily Task Tracker</h1>

    <form @submit.prevent="handleSubmitTask" class="task-form">
      <h2>{{ isEditing ? 'Edit Tugas' : 'Tambah Tugas Baru' }}</h2>
      <div class="form-group">
        <label for="taskName">Nama Tugas:</label>
        <input type="text" id="taskName" v-model.trim="newTaskName" placeholder="Contoh: Belajar Vue.js" required>
      </div>
      <div class="form-group">
        <label for="taskDescription">Deskripsi (Opsional):</label>
        <textarea id="taskDescription" v-model.trim="newTaskDescription" placeholder="Detail lebih lanjut..."></textarea>
      </div>
      <div class="form-group">
        <label for="taskPriority">Prioritas:</label>
        <select id="taskPriority" v-model="newTaskPriority">
          <option value="low">Rendah</option>
          <option value="medium">Sedang</option>
          <option value="high">Tinggi</option>
        </select>
      </div>
      <div class="form-actions">
        <button type="submit">{{ isEditing ? 'Simpan Perubahan' : 'Tambah Tugas' }}</button>
        <button type="button" v-if="isEditing" @click="cancelEdit" class="btn-cancel">Batal</button>
      </div>
    </form>

    <hr>

    <h2>Daftar Tugas Anda</h2>

    <p v-if="!hasTasks" class="no-tasks-message">Belum ada tugas. Ayo tambahkan yang pertama!</p>

    <ul class="task-list" v-else>
      <li v-for="task in tasks" :key="task.id"
          class="task-item"
          :class="{ 'task-completed': task.isCompleted, [getPriorityClass(task.priority)]: true }">
        <div class="task-info">
          <input type="checkbox" :checked="task.isCompleted" @change="toggleComplete(task.id)" class="task-checkbox">
          <div>
            <h3 :class="{ 'strikethrough': task.isCompleted }">{{ task.name }}</h3>
            <p v-if="task.description">{{ task.description }}</p> <small>Prioritas: <span class="priority-text">{{ task.priority.charAt(0).toUpperCase() + task.priority.slice(1) }}</span> | Dibuat: {{ task.createdAt }}</small>
          </div>
        </div>
        <div class="task-actions">
          <button @click="editTask(task.id)" :disabled="isEditing && editingTaskId === task.id" class="btn-edit">Edit</button>
          <button @click="deleteTask(task.id)" class="btn-delete">Hapus</button>
          </div>
      </li>
    </ul>
  </div>
</template>

<style>
/* Basic Styling */
body {
  font-family: Arial, sans-serif;
  background-color: #f4f7f6;
  color: #333;
  margin: 0;
  padding: 20px;
  display: flex;
  justify-content: center;
  align-items: flex-start;
  min-height: 100vh;
}

.container {
  background-color: #fff;
  padding: 30px;
  border-radius: 8px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
  max-width: 800px;
  width: 100%;
}

h1, h2 {
  color: #2c3e50;
  text-align: center;
  margin-bottom: 25px;
}

hr {
  border: 0;
  border-top: 1px solid #eee;
  margin: 30px 0;
}

/* Form Styling */
.task-form {
  background-color: #f9f9f9;
  padding: 20px;
  border-radius: 6px;
  margin-bottom: 30px;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

.form-group input[type="text"],
.form-group textarea,
.form-group select {
  width: calc(100% - 20px); /* Adjust for padding */
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
}

.form-group textarea {
  resize: vertical;
  min-height: 80px;
}

.form-actions {
  display: flex;
  gap: 10px;
  justify-content: flex-end;
  margin-top: 20px;
}

button {
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 16px;
  transition: background-color 0.3s ease;
}

button[type="submit"] {
  background-color: #4CAF50;
  color: white;
}

button[type="submit"]:hover {
  background-color: #45a049;
}

.btn-cancel {
  background-color: #f44336;
  color: white;
}

.btn-cancel:hover {
  background-color: #da190b;
}

button:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}

/* Task List Styling */
.no-tasks-message {
  text-align: center;
  color: #777;
  font-style: italic;
  padding: 20px;
  border: 1px dashed #ccc;
  border-radius: 6px;
}

.task-list {
  list-style: none;
  padding: 0;
}

.task-item {
  background-color: #ffffff;
  border: 1px solid #e0e0e0;
  border-left: 5px solid #007bff; /* Default border color */
  padding: 15px 20px;
  margin-bottom: 10px;
  border-radius: 8px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.05);
  transition: all 0.3s ease;
}

.task-item:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.task-info {
  display: flex;
  align-items: flex-start;
  flex-grow: 1;
  gap: 15px;
}

.task-checkbox {
  transform: scale(1.5);
  margin-top: 5px;
  cursor: pointer;
}

.task-info h3 {
  margin: 0;
  color: #333;
}

.task-info p {
  margin: 5px 0 0;
  color: #555;
}

.task-info small {
  color: #888;
  font-size: 0.85em;
}

.task-actions {
  display: flex;
  gap: 8px;
  margin-left: 20px;
}

.btn-edit, .btn-delete {
  padding: 8px 12px;
  font-size: 14px;
}

.btn-edit {
  background-color: #2196F3;
  color: white;
}
.btn-edit:hover {
  background-color: #0b7dda;
}

.btn-delete {
  background-color: #f44336;
  color: white;
}
.btn-delete:hover {
  background-color: #da190b;
}

/* Attribute Bindings: Conditional Classes */
.task-completed {
  background-color: #e0f2f1; /* Light green for completed tasks */
  border-left-color: #28a745; /* Green border for completed */
  opacity: 0.8;
}

.task-completed .strikethrough {
  text-decoration: line-through;
  color: #777;
}

/* Priority Styling */
.priority-low {
  border-left-color: #4CAF50; /* Green */
}
.priority-low .priority-text {
  color: #4CAF50;
  font-weight: bold;
}

.priority-medium {
  border-left-color: #FFC107; /* Orange */
}
.priority-medium .priority-text {
  color: #FFC107;
  font-weight: bold;
}

.priority-high {
  border-left-color: #F44336; /* Red */
}
.priority-high .priority-text {
  color: #F44336;
  font-weight: bold;
}
</style>