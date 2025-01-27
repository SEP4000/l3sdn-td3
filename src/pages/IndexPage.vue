<template>
  <div class="q-pa-md">
    <q-form v-model="formValid">
      <q-input v-model="newTask.name" label="Nom de la tâche" filled dense :rules="[val => !!val || 'Ce champ est requis']"></q-input>
      <q-input v-model="newTask.details" label="Détails de la tâche" filled dense :rules="[val => !!val || 'Ce champ est requis']"></q-input>
      <q-toggle v-model="newTask.completed" label="Terminée" dense></q-toggle> <br>
      <q-btn class="q-mt-md" color="primary" label="Ajouter" :disable="!isInputValid" @click="addTask"></q-btn> 
    </q-form>
    <br>
    <q-input v-model="searchQuery" label="Rechercher une tâche" filled dense @input="searchTasks"></q-input>
    <q-list bordered class="q-mt-md">
      <q-linear-progress stripe size="10px" :value="completedTasksPercentage" color="primary" class="q-mb-md"></q-linear-progress>
      <div class="text-completed-tasks">{{ completedTasksCount }}/{{ tasks.length }} tâches complétées</div>
      <q-item v-for="(task, index) in filteredTasks" :key="index">
        <q-item-section>
          <q-checkbox v-model="task.completed" color="positive" class="q-checkbox-text"> Tâche {{ index + 1 }} terminée </q-checkbox>
        </q-item-section>
        <q-item-section>
          <div>
            <q-item-label>{{ task.name }}</q-item-label>
            <q-badge v-if="task.completed" color="green" label="Terminée"></q-badge>
            <q-badge v-else color="red" label="Non terminée"></q-badge>
          </div>
          <q-banner class="bg-primary text-white" color="primary" label="Détails" clickable> Détails </q-banner>
            <q-card>
              <q-card-section>
                {{ task.details }}
              </q-card-section>
            </q-card>
        </q-item-section>
        <q-item-section side>
          <q-btn color="deep-orange" glossy icon="delete" @click="showDeleteDialog(index)"></q-btn>
        </q-item-section>
      </q-item>
    </q-list>
    <q-dialog v-model="showDeleteConfirmation" persistent>
      <q-card>
        <q-card-section>
          Êtes-vous sûr de vouloir supprimer cette tâche ?
        </q-card-section>
        <q-card-actions align="right">
          <q-btn color="primary" label="Annuler" @click="cancelDelete"></q-btn>
          <q-btn color="negative" label="Supprimer" @click="deleteConfirmed"></q-btn>
        </q-card-actions>
      </q-card>
    </q-dialog>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { debounce } from 'lodash'

const newTask = ref({ name: '', details: '', completed: false }) 
const tasks = ref([
  { name: 'Tâche 1', details: 'Détails de la tâche 1', completed: true },
  { name: 'Tâche 2', details: 'Détails de la tâche 2', completed: false },
  { name: 'Tâche 3', details: 'Détails de la tâche 3', completed: false }
])

const formValid = ref(false)
const showDeleteConfirmation = ref(false)
let deleteIndex = null
const searchQuery = ref('')

const addTask = () => {
  if (newTask.value.name.trim() !== '' || newTask.value.details.trim() !== '') {
    tasks.value.push({
      name: newTask.value.name,
      details: newTask.value.details,
      completed: newTask.value.completed, 
    })
    newTask.value = { name: '', details: '', completed: false } 
    formValid.value = false
  }
}

const showDeleteDialog = (index) => {
  showDeleteConfirmation.value = true
  deleteIndex = index
}

const cancelDelete = () => {
  showDeleteConfirmation.value = false
  deleteIndex = null
}

const deleteConfirmed = () => {
  if (deleteIndex !== null) {
    tasks.value.splice(deleteIndex, 1)
    deleteIndex = null
    showDeleteConfirmation.value = false
  }
}

const completedTasksCount = computed(() => {
  return tasks.value.filter(task => task.completed).length
})

const completedTasksPercentage = computed(() => {
  return (completedTasksCount.value / tasks.value.length)
})

const isInputValid = computed(() => {
  return newTask.value.name.trim() !== '' && newTask.value.details.trim() !== ''
})

const filteredTasks = computed(() => {
  return tasks.value.filter(task => {
    return task.name.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
           task.details.toLowerCase().includes(searchQuery.value.toLowerCase())
  })
})

const searchTasks = debounce(() => {
  // Function for searching tasks
}, 300)
</script>

<style>
.q-item-section-side {
  display: flex;
  align-items: center;
}

.text-completed-tasks {
  text-align: center;
  margin-bottom: 10px;
}

.q-checkbox-text {
  margin-left: 5px;
}
</style>