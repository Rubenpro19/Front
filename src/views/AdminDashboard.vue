<template>
  <div class="admin-dashboard">
    <h1>Panel de Administrador</h1>

    <!-- Botón para abrir el formulario de crear usuario -->
    <button class="btn btn-success" @click="mostrarFormularioCrearUsuario">
      <i class="fas fa-user-plus"></i> Crear Usuario
    </button>

    <!-- Formulario para crear un usuario -->
    <div v-if="showCreateForm" class="create-user-form">
      <h3>Crear Nuevo Usuario</h3>
      <form @submit.prevent="crearUsuario">
        <transition name="fade">
          <div v-if="showError" class="error-message">
            {{ errorMessage }}
          </div>
        </transition>
        <div class="form-group">
          <label for="nombre">Nombre</label>
          <input type="text" v-model="newUser.nombre" id="nombre" required />
        </div>
        <div class="form-group">
          <label for="email">Correo Electrónico</label>
          <input type="email" v-model="newUser.email" id="email" required />
        </div>
        <div class="form-group">
          <label for="password">Contraseña</label>
          <input type="password" v-model="newUser.password" id="password" required />
        </div>
        <div class="form-group">
          <label for="password_confirmation">Confirmar Contraseña</label>
          <input type="password" v-model="newUser.password_confirmation" id="password_confirmation" required />
        </div>
        <div class="form-group">
          <label for="rol_id">Rol:</label>
          <select v-model="newUser.rol_id" id="rol_id" required>
            <option value="2">Nutricionista</option>
            <option value="3">Paciente</option>
          </select>
        </div>
        <button type="submit"><i class="fas fa-check-circle"></i> Crear Usuario</button>
        <button @click="cancelarCrearUsuario" type="button"><i class="fas fa-times-circle"></i> Cancelar</button>
      </form>
    </div>

    <!-- Formulario para editar un usuario -->
    <div v-if="showEditForm" class="edit-user-form">
      <h3>Editar Usuario</h3>
      <form @submit.prevent="guardarEdicionUsuario">
        <div class="form-group">
          <label for="edit-nombre">Nombre</label>
          <input type="text" v-model="editedUser.nombre" id="edit-nombre" required />
        </div>
        <div class="form-group">
          <label for="edit-email">Correo Electrónico</label>
          <input type="email" v-model="editedUser.email" id="edit-email" required />
        </div>
        <button type="submit"><i class="fas fa-save"></i> Guardar Cambios</button>
        <button type="button" @click="cancelarEdicionUsuario"><i class="fas fa-times"></i> Cancelar</button>
      </form>
    </div>

    <!-- Lista de usuarios -->
    <div class="user-list">
      <h2>Gestión de Usuarios</h2>
      <table>
        <thead>
          <tr>
            <th>Nombre</th>
            <th>Correo Electrónico</th>
            <th>Rol</th>
            <th>Acciones</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="usuario in usuarios" :key="usuario.id">
            <td>{{ usuario.nombre }}</td>
            <td>{{ usuario.email }}</td>
            <td>{{ usuario.rol_id === 2 ? 'Nutricionista' : 'Paciente' }}</td>
            <td>
              <button class="btn btn-primary" @click="editarUsuario(usuario)">
                <i class="fas fa-edit"></i> Editar
              </button>
              <button class="btn btn-danger" @click="eliminarUsuario(usuario.id)" v-if="usuario.id !== adminId">
                <i class="fas fa-trash"></i> Eliminar
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import apiService from '../services/apiService';

export default {
  data() {
    return {
      usuarios: [],
      newUser: {
        nombre: '',
        email: '',
        password: '',
        password_confirmation: '',
        rol_id: 3,
      },
      editedUser: null,
      showCreateForm: false,
      showEditForm: false,
      showError: false,
      errorMessage: '',
      adminId: JSON.parse(sessionStorage.getItem('user')).id,
    };
  },
  created() {
    this.obtenerUsuarios();
  },
  methods: {
    async obtenerUsuarios() {
      try {
        const response = await apiService.get('/user');
        this.usuarios = response.data.filter(usuario => usuario.rol_id === 2 || usuario.rol_id === 3);
      } catch (error) {
        console.error('Error al obtener usuarios:', error);
      }
    },
    mostrarFormularioCrearUsuario() {
      this.showCreateForm = true;
    },
    cancelarCrearUsuario() {
      this.showCreateForm = false;
      this.newUser = { nombre: '', email: '', password: '', password_confirmation: '', rol_id: 3 };
      this.showError = false;
    },
    async crearUsuario() {
      if (this.newUser.password !== this.newUser.password_confirmation) {
        this.errorMessage = 'Las contraseñas no coinciden.';
        this.showError = true;
        return;
      }
      try {
        await apiService.post('/user', this.newUser);
        this.showCreateForm = false;
        this.obtenerUsuarios();
        this.newUser = { nombre: '', email: '', password: '', password_confirmation: '', rol_id: 3 };
      } catch (error) {
        this.errorMessage = error.response?.data.message || 'Error en la creación del usuario.';
        this.showError = true;
      }
    },
    async editarUsuario(usuario) {
      this.editedUser = { id: usuario.id, nombre: usuario.nombre, email: usuario.email }; // Solo los campos editables
      this.showEditForm = true;
    },
    async guardarEdicionUsuario() {
      try {
        const { id, nombre, email } = this.editedUser;
        await apiService.put(`/user/${id}`, { nombre, email });
        this.showEditForm = false;
        this.obtenerUsuarios();
      } catch (error) {
        console.error('Error al editar usuario:', error);
      }
    },
    cancelarEdicionUsuario() {
      this.showEditForm = false;
      this.editedUser = null;
    },
    async eliminarUsuario(id) {
      try {
        await apiService.delete(`/user/${id}`);
        this.obtenerUsuarios();
      } catch (error) {
        console.error('Error al eliminar usuario:', error);
      }
    },
  },
};
</script>

<style scoped>
.admin-dashboard {
  padding: 20px;
}

button {
  margin: 5px;
}

.create-user-form,
.edit-user-form {
  margin-top: 20px;
  padding: 20px;
  background-color: #f9f9f9;
  border: 1px solid #ddd;
  border-radius: 8px;
  max-width: 500px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  margin-left: auto;
  margin-right: auto;
}

.create-user-form h3,
.edit-user-form h3 {
  font-size: 20px;
  text-align: center;
  margin-bottom: 20px;
  color: #333;
}

.create-user-form .form-group,
.edit-user-form .form-group {
  margin-bottom: 15px;
}

.create-user-form label,
.edit-user-form label {
  display: block;
  font-weight: bold;
  margin-bottom: 5px;
  color: #555;
}

.create-user-form input,
.create-user-form select,
.edit-user-form input,
.edit-user-form select {
  width: 100%;
  padding: 10px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
}

.create-user-form input:focus,
.create-user-form select:focus,
.edit-user-form input:focus,
.edit-user-form select:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 5px rgba(0, 123, 255, 0.5);
}

.create-user-form button,
.edit-user-form button {
  width: 100%;
  padding: 10px;
  font-size: 16px;
  border: none;
  border-radius: 4px;
  background-color: rgb(21, 115, 71);
  color: white;
  cursor: pointer;
  font-weight: bold;
  transition: background-color 0.3s ease;
}

.create-user-form button:hover,
.edit-user-form button:hover {
  background-color: rgb(7, 85, 48);
}

.create-user-form button[type="button"],
.edit-user-form button[type="button"] {
  background-color: #6c757d;
}

.create-user-form button[type="button"]:hover,
.edit-user-form button[type="button"]:hover {
  background-color: #5a6268;
}

.error-message {
  color: red;
  margin-bottom: 15px;
  font-size: 14px;
}

.success-message {
  color: rgb(21, 115, 71);
  margin-bottom: 15px;
  font-size: 14px;
  text-align: center;
}

.user-list {
  margin-top: 30px;
}

.user-list table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
}

.user-list th, .user-list td {
  padding: 12px;
  border: 1px solid #ddd;
  text-align: left;
}

.user-list th {
  background-color: #f2f2f2;
}

.user-list td {
  background-color: #fafafa;
}

.user-list button {
  padding: 5px 10px;
  background-color: rgb(21, 115, 71);
  color: white;
  border: none;
  cursor: pointer;
  font-size: 14px;
}

.user-list button:hover {
  background-color: rgb(5, 80, 45);
}

/* Estilo responsive */
@media (max-width: 768px) {
  .create-user-form,
  .edit-user-form {
    width: 100%;
    padding: 10px;
  }

  .user-list table {
    font-size: 14px;
  }

  .user-list button {
    font-size: 12px;
  }
}
</style>
