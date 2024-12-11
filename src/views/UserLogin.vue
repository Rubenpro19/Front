<template>
  <div class="login-container">
    <h2>Iniciar Sesión</h2>
    <form @submit.prevent="handleLogin">
      <!-- Mensaje de error -->
      <transition name="fade">
        <div v-if="showError" class="error-message">
          <i class="fas fa-exclamation-circle icon-green"></i> {{ errorMessage }}
        </div>
      </transition>

      <!-- Mostrar mensaje de éxito -->
      <transition name="fade">
        <div v-if="showSuccess" class="success-message">
          <i class="fas fa-check-circle icon-green"></i> Inicio de sesión exitoso. Redirigiendo...
        </div>
      </transition>

      <!-- Campo de correo electrónico -->
      <div class="form-group">
        <label for="email">Correo Electrónico</label>
        <div class="input-with-icon">
          <i class="fas fa-envelope icon-green"></i>
          <input type="email" v-model="email" id="email" required />
        </div>
      </div>

      <!-- Campo de contraseña -->
      <div class="form-group">
        <label for="password">Contraseña</label>
        <div class="input-with-icon">
          <i class="fas fa-lock icon-green"></i>
          <input type="password" v-model="password" id="password" required />
        </div>
      </div>

      <!-- Botón para enviar el formulario -->
      <button type="submit"><i class="fas fa-sign-in-alt"></i> Iniciar Sesión</button>

      <!-- Enlace para registrarse si no se tiene una cuenta -->
      <div class="login-footer">
        <h5>No tienes cuenta? <RouterLink to="/UserRegister">Crear Cuenta</RouterLink></h5>
      </div>
    </form>
  </div>
</template>

<script>
import apiService from '../services/apiService'; // Importar el servicio de la API

export default {
  data() {
    return {
      email: '', // Email del usuario
      password: '', // Contraseña del usuario
      showError: false, // Muestra el error si existe
      errorMessage: '', // Mensaje de error
      showSuccess: false, // Muestra el mensaje de éxito
    };
  },
  methods: {
    async handleLogin() {
      try {
        // Realizamos la solicitud de inicio de sesión al backend
        const response = await apiService.post('/login', {
          email: this.email,
          password: this.password,
        });

        // Obtenemos el token y el rol del usuario desde la respuesta
        const token = response.data.token;
        const user = response.data.user;
        const rolId = user.rol_id;

        // Guardamos el token y el rol del usuario en sessionStorage
        sessionStorage.setItem('authToken', token);
        sessionStorage.setItem('user', JSON.stringify(user));

        // Mostrar mensaje de éxito visual
        this.showSuccess = true;
        this.showError = false;

        // Redirigir según el rol del usuario
        setTimeout(() => {
          if (rolId === 1) {
            // Administrador
            this.$router.push('/AdminDashboard');
          } else if (rolId === 2) {
            // Nutricionista
            this.$router.push('/NutricionistaDashboard');
          } else {
            // Paciente (por defecto)
            this.$router.push('/UserDashboard');
          }
        }, 1500); // Espera 1.5 segundos antes de redirigir

      } catch (error) {
        // Manejar errores de inicio de sesión
        this.showError = true;
        this.showSuccess = false;
        this.errorMessage = 'Error al iniciar sesión. Verifica tus credenciales.';
        console.error('Error al iniciar sesión:', error);
      }
    },
  },
};
</script>

<style scoped>
.login-container {
  max-width: 400px;
  margin: 100px auto;
  padding: 50px;
  border: 1px solid #ccc;
  border-radius: 10px;
  background-color: #f9f9f9;
  box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.1);
  animation: fadeIn 1s ease-in-out;
}

.login-container h2{
  display: flex;
  justify-content: center;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 10px;
  font-weight: bold;
}

.input-with-icon {
  position: relative;
}

.input-with-icon i {
  position: absolute;
  left: 10px;
  top: 50%;
  transform: translateY(-50%);
  color: #28a745; /* Color verde */
}

.form-group input {
  width: 100%;
  padding: 10px 10px 10px 30px; /* Agrega espacio para el ícono */
  border-radius: 4px;
  border: 1px solid #ccc;
  box-shadow: 0px 2px 5px rgba(0, 0, 0, 0.1);
  transition: box-shadow 0.3s ease;
}

.form-group input:focus {
  box-shadow: 0px 2px 10px rgba(0, 123, 255, 0.5);
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
  transition: background-color 0.3s ease;
}

button:hover {
  background-color: #218838;
}

button i {
  margin-right: 8px;
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

.login-footer {
  margin-top: 20px;
  display: flex;
  justify-content: space-between;
}

.login-footer a {
  color: green;
  cursor: pointer;
}

.login-footer a:hover {
  text-decoration: underline;
}

.login-footer h5{
  font-size: 1em;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}

.fade-enter, 
.fade-leave-to {
  opacity: 0;
}

/* Añadir clase para los iconos verdes */
.icon-green {
  color: #28a745; /* Color verde */
}
</style>
