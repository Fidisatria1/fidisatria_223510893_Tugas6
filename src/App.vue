<template>
  <div>
    <form @submit.prevent="save">
      <input type="text" v-model="form.name" placeholder="Name" /><br>
      <input type="text" v-model="form.species" placeholder="Species" /><br>
      <input type="text" v-model="form.color" placeholder="Color" /><br>
      <button type="submit" class="btn-primary">Save</button>
    </form>
    <ul>
      <li v-for="animal in animals" :key="animal.id">
        <h3>{{ animal.name }}</h3>
        <p>Species: {{ animal.species }}</p>
        <p>Color: {{ animal.color }}</p>
        <button @click="edit(animal)" class="btn-edit">Edit</button>
        <button @click="remove(animal.id)" class="btn-delete">Delete</button>
      </li>
    </ul>
    <button @click="load" class="btn-load">Load</button>
    <p class="text-h6">Welcome to Animal Data View</p>
  </div>
</template>

<script>
import { ref, reactive, onMounted } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: null,
      name: '',
      species: '',
      color: ''
    });

    const animals = ref([]);

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/animals');
        animals.value = response.data;
      } catch (error) {
        console.error('Error loading animals:', error);
      }
    }

    async function save() {
      try {
        const url = form.id ? `http://localhost:3000/animals/${form.id}` : 'http://localhost:3000/animals';
        const method = form.id ? "put" : "post";
        const response = await axios[method](url, form);

        if (method === 'put') {
          const index = animals.value.findIndex(animal => animal.id === response.data.id);
          if (index !== -1) {
            animals.value[index] = response.data;
          }
        } else {
          animals.value.push(response.data);
        }

        form.id = null;
        form.name = '';
        form.species = '';
        form.color = '';
      } catch (error) {
        console.error('Error saving animal:', error);
      }
    }

    async function remove(id) {
      try {
        await axios.delete(`http://localhost:3000/animals/${id}`);
        animals.value = animals.value.filter(animal => animal.id !== id);
      } catch (error) {
        console.error('Error deleting animal:', error);
      }
    }

    function edit(animal) {
      form.id = animal.id;
      form.name = animal.name;
      form.species = animal.species;
      form.color = animal.color;
    }

    onMounted(load);

    return { form, animals, save, edit, remove, load };
  }
}
</script>

<style>
.btn-primary {
  background-color: blue;
  color: white;
  border: none;
  padding: 10px 20px;
  cursor: pointer;
  border-radius: 5px;
  margin-bottom: 10px;
}

.btn-primary:hover {
  background-color: darkblue;
}

.btn-edit, .btn-delete, .btn-load {
  background-color: lightblue;
  color: blue;
  border: none;
  padding: 5px 10px;
  cursor: pointer;
  border-radius: 3px;
  margin-right: 5px;
}

.btn-edit:hover, .btn-delete:hover, .btn-load:hover {
  background-color: blue;
  color: white;
}
</style>
