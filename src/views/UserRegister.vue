<template>
  <div class="registro-container">
    <h2>Crear Nueva Cuenta</h2>
    <form @submit.prevent="handleRegistro">
      <!-- Mostrar mensaje de error si existe algún problema -->
      <transition name="fade">
        <div v-if="showError" class="error-message">
          {{ errorMessage }}
        </div>
      </transition>

      <!-- Mostrar mensaje de éxito -->
      <transition name="fade">
        <div v-if="registroExitoso" class="success-message">
          ¡Cuenta registrada exitosamente! Redirigiendo al inicio de sesión...
        </div>
      </transition>
      <!-- Campos del formulario -->
      <div class="form-group">
        <label for="nombre">Nombre</label>
        <input type="text" v-model="nombre" id="nombre" required />
      </div>

      <div class="form-group">
        <label for="email">Correo Electrónico</label>
        <input type="email" v-model="email" id="email" required />
      </div>

      <div class="form-group">
        <label for="password">Contraseña</label>
        <input type="password" v-model="password" id="password" required />
      </div>

      <div class="form-group">
        <label for="password_confirmation">Confirmar Contraseña</label>
        <input type="password" v-model="password_confirmation" id="password_confirmation" required />
      </div>
       <!-- Botón para enviar el formulario -->
      <button type="submit">Registrarse</button>
       <!-- Enlace para redirigir al inicio de sesión si ya tiene una cuenta -->
      <div class="registro-footer">
        <h5>¿Ya tienes cuenta? <RouterLink to="/UserLogin">Inicia Sesión</RouterLink></h5>
      </div>
    </form>
  </div>
</template>

<script>
import apiService from '../services/apiService'; // Asegúrate de que esta ruta es correcta

export default {
  name: 'RegistroUsuario',
  data() {
    return {
      nombre: '',
      email: '',
      password: '',
      password_confirmation: '',
      showError: false,
      errorMessage: '',
      registroExitoso: false, // Nueva variable para manejar el estado de éxito
    };
  },
  methods: {
    async handleRegistro() {
      console.log('Iniciando el registro de usuario...');

      // Validación simple
      if (this.nombre === '' || this.email === '' || this.password === '' || this.password_confirmation === '') {
        this.errorMessage = 'Por favor, completa todos los campos.';
        this.showError = true;
        console.log('Validación fallida: faltan campos por completar.');
        return;
      }

      if (this.password !== this.password_confirmation) {
        this.errorMessage = 'Las contraseñas no coinciden.';
        this.showError = true;
        console.log('Validación fallida: las contraseñas no coinciden.');
        return;
      }

      try {
        console.log('Enviando datos al backend para crear el usuario...', {
          nombre: this.nombre,
          email: this.email,
          password: this.password,
        });

        // Realizamos la petición al backend para crear el usuario
        const response = await apiService.post('/user', {
          nombre: this.nombre,
          email: this.email,
          password: this.password,
        });

        console.log('Respuesta del backend recibida:', response);

        // Mostrar mensaje de éxito en vez de alert()
        this.registroExitoso = true;
        this.showError = false;

        // Limpiar el formulario y cualquier error
        this.limpiarFormulario();

        // Redirigir al login después de un pequeño retraso
        setTimeout(() => {
          console.log('Redirigiendo al login...');
          this.$router.push('/UserLogin');
        }, 2000); // Redirige después de 2 segundos

      } catch (error) {
        console.log('Error recibido durante el registro:', error);
        if (error.response && error.response.status === 422) {
          this.errorMessage = 'Error de validación: ' + Object.values(error.response.data).join(', ');
          console.log('Error de validación:', error.response.data);
        } else {
          this.errorMessage = 'Ocurrió un error en el registro. Intenta nuevamente.';
          console.log('Error general:', error);
        }
        this.showError = true;
      }
    },
    limpiarFormulario() {
      console.log('Limpiando el formulario...');
      this.nombre = '';
      this.email = '';
      this.password = '';
      this.password_confirmation = ''; // Limpiar el campo de confirmación de contraseña también
    },
  },
};
</script>

<style scoped>
/* Estilos del formulario de registro*/
.registro-container {
  max-width: 400px;
  margin: 100px auto;
  padding: 50px;
  border: 1px solid #ccc;
  border-radius: 10px;
  background-color: #f9f9f9;
  box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.1);
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 10px;
  font-weight: bold;
}

.form-group input {
  width: 100%;
  padding: 10px;
  border-radius: 4px;
  border: 1px solid #ccc;
}

button {
  width: 100%;
  padding: 10px;
  border: none;
  background-color: #28a745;
  color: white;
  font-size: 16px;
  cursor: pointer;
  border-radius: 4px;
}

.error-message {
  color: red;
  margin-bottom: 15px;
  font-size: 14px;
}

.success-message {
  color: green;
  margin-bottom: 15px;
  font-size: 14px;
  text-align: center;
}

.registro-footer {
  margin-top: 20px;
}

.registro-footer a {
  color: #007bff;
  cursor: pointer;
}

.registro-footer a:hover {
  text-decoration: underline;
}
</style>