<template>
  <div class="container">
    <h1>User Management</h1>

    <!-- 🔹 Formulaire Ajout -->
    <form @submit.prevent="addUser" class="form">
      <input v-model="newUser.name" placeholder="Name" required />
      <input v-model="newUser.email" placeholder="Email" required />
      <button type="submit">Add User</button>
    </form>

    <hr />

    <!-- 🔹 Tableau Users -->
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Email</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="user in users" :key="user._id">
          <!-- Mode édition -->
          <template v-if="editingId === user._id">
            <td><input v-model="editingUser.name" /></td>
            <td><input v-model="editingUser.email" /></td>
            <td>
              <button @click="updateUser(user._id)">Update</button>
              <button @click="cancelEdit">Cancel</button>
            </td>
          </template>

          <!-- Mode normal -->
          <template v-else>
            <td>{{ user.name }}</td>
            <td>{{ user.email }}</td>
            <td>
              <button @click="startEdit(user)">Edit</button>
              <button @click="deleteUser(user._id)">Delete</button>
            </td>
          </template>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

const API_URL = "https://thursday-production.up.railway.app/users";

const users = ref([]);
const newUser = ref({ name: "", email: "" });

const editingId = ref(null);
const editingUser = ref({});

// 🔹 GET users
async function fetchUsers() {
  try {
    const res = await fetch(API_URL);
    if (!res.ok) throw new Error(await res.text());
    users.value = await res.json();
  } catch (err) {
    console.error("Fetch users error:", err.message);
  }
}

// 🔹 ADD user
async function addUser() {
  try {
    const res = await fetch(API_URL, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(newUser.value)
    });
    if (!res.ok) throw new Error(await res.text());

    newUser.value = { name: "", email: "" };
    fetchUsers();
  } catch (err) {
    console.error("Add user error:", err.message);
  }
}

// 🔹 DELETE user
async function deleteUser(id) {
  try {
    if (!id) throw new Error("No ID for deletion");

    const res = await fetch(`${API_URL}/${id}`, { method: "DELETE" });
    if (!res.ok) throw new Error(await res.text());

    fetchUsers();
  } catch (err) {
    console.error("Delete user error:", err.message);
  }
}

// 🔹 Start edit
function startEdit(user) {
  if (!user._id) return console.error("User has no _id:", user);
  editingId.value = user._id;
  editingUser.value = { ...user };
}

// 🔹 Update user
async function updateUser(id) {
  try {
    if (!id) throw new Error("No ID for update");

    const res = await fetch(`${API_URL}/${id}`, {
      method: "PATCH",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(editingUser.value)
    });

    if (!res.ok) throw new Error(await res.text());

    editingId.value = null;
    editingUser.value = { name: "", email: "" };
    fetchUsers();
  } catch (err) {
    console.error("Update user error:", err.message);
  }
}

// 🔹 Cancel edit
function cancelEdit() {
  editingId.value = null;
}

onMounted(fetchUsers);
</script>

<style>
.container {
  max-width: 800px;
  margin: auto;
  padding: 20px;
  font-family: Arial;
}

form {
  margin-bottom: 20px;
}

input {
  padding: 6px;
  margin-right: 8px;
}

button {
  margin-right: 5px;
  padding: 5px 10px;
  cursor: pointer;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th,
td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}

th {
  background-color: #f4f4f4;
}
</style>
