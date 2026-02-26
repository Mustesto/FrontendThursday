<template>
  <div class="container">
    <h1>Users CRUD</h1>

    <!-- Formulaire pour créer un user -->
    <form @submit.prevent="addUser">
      <input v-model="newUser.name" placeholder="Name" required />
      <input v-model="newUser.email" placeholder="Email" required />
      <button type="submit">Add User</button>
    </form>

    <hr />

    <!-- Liste des users -->
    <ul>
      <li v-for="user in users" :key="user._id">
        {{ user.name }} - {{ user.email }}
        <button @click="deleteUser(user._id)">Delete</button>
        <button @click="editUser(user)">Edit</button>
      </li>
    </ul>

    <!-- Modifier un user -->
    <div v-if="editingUser">
      <h3>Edit User</h3>
      <input v-model="editingUser.name" placeholder="Name" />
      <input v-model="editingUser.email" placeholder="Email" />
      <button @click="updateUser">Update</button>
      <button @click="cancelEdit">Cancel</button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

// 🔹 Ton backend Railway
const API_URL = "https://thursday-production.up.railway.app/users";

const users = ref([]);
const newUser = ref({ name: "", email: "" });
const editingUser = ref(null);

// GET /users
async function fetchUsers() {
  const res = await fetch(API_URL);
  users.value = await res.json();
}

// POST /users
async function addUser() {
  await fetch(API_URL, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(newUser.value)
  });
  newUser.value = { name: "", email: "" };
  fetchUsers();
}

// DELETE /users/:id
async function deleteUser(id) {
  await fetch(`${API_URL}/${id}`, { method: "DELETE" });
  fetchUsers();
}

// Edit user
function editUser(user) {
  editingUser.value = { ...user };
}

// PATCH /users/:id
async function updateUser() {
  await fetch(`${API_URL}/${editingUser.value._id}`, {
    method: "PATCH",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      name: editingUser.value.name,
      email: editingUser.value.email
    })
  });
  editingUser.value = null;
  fetchUsers();
}

function cancelEdit() {
  editingUser.value = null;
}

// Charger la liste au démarrage
onMounted(fetchUsers);
</script>

<style>
.container { max-width: 600px; margin: auto; padding: 1rem; font-family: sans-serif; }
input { margin: 0.3rem; padding: 0.3rem; }
button { margin-left: 0.3rem; }
</style>
