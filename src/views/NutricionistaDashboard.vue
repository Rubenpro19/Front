<template>
  <div class="login-container">
    <h2 class="text-center mb-4 titulo">Citas Reservadas</h2>
    <div class="text-center mb-4">
      <button class="btn btn-success" @click="crearTurnos">Generar Todos los Turnos</button>
    </div>
    <!-- Notificación de éxito o error -->
    <div v-if="successMessage" class="success-message">
      {{ successMessage }}
    </div>
    <div v-if="errorMessage" class="error-message">
      {{ errorMessage }}
    </div>

    <!-- Tabla de Turnos -->
    <div v-if="turnos.length > 0" class="table-responsive">
      <table class="table table-bordered tabla">
        <thead class="thead-light">
          <tr>
            <th>#</th>
            <th>Paciente</th>
            <th>Fecha</th>
            <th>Hora</th>
            <th>Acciones</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(turno, index) in turnos" :key="turno.id">
            <td>{{ index + 1 }}</td>
            <td>{{ turno.paciente.nombre }}</td>
            <td>{{ turno.fecha }}</td>
            <td>{{ turno.hora }}</td>
            <td>
              <button
                class="btn btn-accion"
                @click="generarAtencion(turno.id)"
              >
                Generar Atención
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Mensaje si no hay turnos -->
    <div v-else class="text-center mensaje">
      <p>No hay turnos reservados por los pacientes.</p>
    </div>

    <!-- Lista de Atenciones -->
    <div v-if="atenciones.length > 0" class="mt-5">
      <h2 class="text-center mb-4 titulo">Atenciones Realizadas</h2>
      <div
        v-for="(atencion, index) in atenciones"
        :key="atencion.id"
        class="card atencion-card"
      >
        <div class="card-header header-atencion">
          <h5>
            Atención #{{ index + 1 }} - Paciente: {{ atencion.turno?.paciente?.nombre || 'N/A' }}
          </h5>
          <p>
            <strong>Fecha:</strong> {{ atencion.turno?.fecha || 'N/A' }} |
            <strong>Hora:</strong> {{ atencion.turno?.hora || 'N/A' }}
          </p>
          <button
            class="btn btn-detalle"
            @click="toggleDetalles(index)"
          >
            {{ atencion.mostrarDetalles ? 'Ocultar Detalles' : 'Ver Detalles' }}
          </button>
        </div>
        <div v-if="atencion.mostrarDetalles" class="card-body">
          <p><strong>Altura:</strong> {{ atencion.altura }} m</p>
          <p><strong>Peso:</strong> {{ atencion.peso }} kg</p>
          <p><strong>Cintura:</strong> {{ atencion.cintura }} cm</p>
          <p><strong>Cadera:</strong> {{ atencion.cadera }} cm</p>
          <p><strong>Circunferencia Muñeca:</strong> {{ atencion.circunferencia_muneca }}</p>
          <p><strong>Circunferencia Cuello:</strong> {{ atencion.circunferencia_cuello }}</p>
          <p><strong>Actividad Física:</strong> {{ atencion.actividad_fisica }}</p>
          <p><strong>IMC:</strong> {{ atencion.imc }}</p>
          <p><strong>TMB:</strong> {{ atencion.tmb }}</p>
          <p><strong>Cintura/Talla:</strong> {{ atencion.cintura_talla }}</p>
          <p><strong>Cintura/Cadera:</strong> {{ atencion.cintura_cadera }}</p>
          <p><strong>Porcentaje Grasa:</strong> {{ atencion.porcentaje_grasa }}</p>
          <p><strong>Complexión Hueso:</strong> {{ atencion.complexion_hueso }}</p>
          <p><strong>Observación:</strong> {{ atencion.observacion }}</p>
        </div>
      </div>
    </div>

    <!-- Mensaje si no hay atenciones -->
    <div v-else-if="atenciones.length === 0" class="text-center mt-5 mensaje">
      <p>No hay registros de atenciones disponibles.</p>
    </div>
  </div>
</template>

<script>
import apiService from "@/services/apiService";

export default {
  data() {
    return {
      turnos: [], // Lista de turnos reservados
      atenciones: [], // Lista de atenciones realizadas
      successMessage: "", // Mensaje de éxito
      errorMessage: "", // Mensaje de error
    };
  },
  methods: {
    async obtenerTurnosReservados() {
      try {
        const response = await apiService.obtenerTurnosReservados();
        this.turnos = response.data || [];
      } catch (error) {
        console.error("Error al obtener turnos reservados:", error);
      }
    },
    async obtenerAtenciones() {
      try {
        const response = await apiService.obtenerAtenciones();
        this.atenciones = response.data.atencion.map((atencion) => ({
          ...atencion,
          mostrarDetalles: false,
        }));
      } catch (error) {
        console.error("Error al obtener atenciones:", error);
      }
    },
    toggleDetalles(index) {
      if (this.atenciones[index]) {
        this.atenciones[index].mostrarDetalles = !this.atenciones[index].mostrarDetalles;
      }
    },
    async crearTurnos() {
      try {
        const response = await apiService.crearTurno();  // Llamamos a la función de apiService para crear los turnos
        console.log('Respuesta al crear turnos:', response.data);
        // Mostrar mensaje de éxito
        this.successMessage = "¡Turnos generados con éxito!";
        this.errorMessage = ""; // Limpiar mensajes de error
        this.obtenerTurnosReservados(); // Opcional, actualiza la lista de turnos después de crearlos
      } catch (error) {
        console.error('Error al generar los turnos:', error);
        // Mostrar mensaje de error
        this.errorMessage = "Hubo un problema al generar los turnos.";
        this.successMessage = ""; // Limpiar mensajes de éxito
      }
    },
  },
  mounted() {
    this.obtenerTurnosReservados(); // Cargar turnos al montar el componente
    this.obtenerAtenciones(); // Cargar atenciones realizadas al montar el componente
  },
};
</script>

<style scoped>
.login-container {
  padding: 2rem;
  background-color: #ffffff;
  border-radius: 8px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
  max-width: 800px;
  margin: 0 auto;
  font-family: 'Arial', sans-serif;
}

.titulo {
  color: #333333;
  font-weight: 600;
  font-size: 1.5rem;
}

.table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 1rem;
}

.table th {
  background-color: #f8f9fa;
  text-align: left;
  padding: 0.8rem;
  border-bottom: 1px solid #dddddd;
  color: #333;
}

.table td {
  padding: 0.8rem;
  border-bottom: 1px solid #dddddd;
  color: #555;
}

.btn {
  padding: 0.4rem 0.8rem;
  border-radius: 4px;
  border: none;
  cursor: pointer;
  font-size: 0.9rem;
}

.btn-accion {
  background-color: #007bff;
  color: white;
  transition: background-color 0.2s;
}

.btn-accion:hover {
  background-color: #0056b3;
}

.btn-detalle {
  background-color: #f8f9fa;
  color: #333;
  border: 1px solid #ddd;
}

.btn-detalle:hover {
  background-color: #e2e6ea;
}

.mensaje {
  color: #777;
  font-size: 1rem;
  margin-top: 1.5rem;
}

.card {
  margin-top: 1.5rem;
  border: 1px solid #e2e6ea;
  border-radius: 6px;
  overflow: hidden;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

.card-header {
  background-color: #f8f9fa;
  padding: 1rem;
  font-weight: bold;
  color: #333;
}

.card-body {
  padding: 1rem;
  color: #555;
}

/* Estilos para el mensaje de éxito */
.success-message {
  color: #28a745;
  background-color: #e9f7e9;
  padding: 1rem;
  border-radius: 4px;
  margin-bottom: 1rem;
}

/* Estilos para el mensaje de error */
.error-message {
  color: #dc3545;
  background-color: #f8d7da;
  padding: 1rem;
  border-radius: 4px;
  margin-bottom: 1rem;
}
</style>
