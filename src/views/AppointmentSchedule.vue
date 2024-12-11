<template>
  <div class="appointment-container">
    <h2>Gestión de Turnos</h2>

    <div class="turnos-reservados">
      <h3><i class="fas fa-list-alt"></i> Mis Turnos Reservados</h3>
      <ul v-if="turnosReservados.length">
        <li v-for="turno in turnosReservados" :key="turno.id">
          <span>{{ turno.fecha }} - {{ turno.hora }}</span>
          <button @click="cancelarTurno(turno.id)" aria-label="Cancelar turno">
            <i class="fas fa-times-circle"></i> Cancelar
          </button>
        </li>
      </ul>
      <p v-else>No tienes turnos reservados.</p>
    </div>

    <!-- Mensaje -->
    <div v-if="message" :class="['message', { success: isSuccessMessage, error: !isSuccessMessage }]">
      {{ message }}
    </div>

    <!-- Selección de Nutricionistas -->
    <div class="nutricionistas-section">
      <h3><i class="fas fa-user-md"></i> Seleccionar Nutricionista</h3>
      <select v-model="nutricionistaSeleccionado" @change="cargarTurnos" aria-label="Seleccionar nutricionista">
        <option disabled value="">Seleccione un nutricionista...</option>
        <option v-for="nutricionista in nutricionistas" :key="nutricionista.id" :value="nutricionista.id">
          {{ nutricionista.nombre }}
        </option>
      </select>
    </div>

    <!-- Turnos Disponibles -->
    <div class="turnos-container">
      <!-- Matutinos -->
      <div class="turnos-column">
        <h3><i class="fas fa-sun"></i> Turnos Matutinos</h3>
        <ul>
          <li v-for="turno in turnosMatutinos" :key="turno.id">
            <span>{{ turno.fecha }} - {{ turno.hora }}</span>
            <button @click="reservarTurno(turno.id)" aria-label="Reservar turno">
              <i class="fas fa-check-circle"></i> Reservar
            </button>
          </li>
        </ul>
        <p v-if="turnosMatutinos.length === 0">No hay turnos matutinos disponibles.</p>
      </div>

      <!-- Vespertinos -->
      <div class="turnos-column">
        <h3><i class="fas fa-moon"></i> Turnos Vespertinos</h3>
        <ul>
          <li v-for="turno in turnosVespertinos" :key="turno.id">
            <span>{{ turno.fecha }} - {{ turno.hora }}</span>
            <button @click="reservarTurno(turno.id)" aria-label="Reservar turno">
              <i class="fas fa-check-circle"></i> Reservar
            </button>
          </li>
        </ul>
        <p v-if="turnosVespertinos.length === 0">No hay turnos vespertinos disponibles.</p>
      </div>
    </div>
  </div>
</template>

<script>
import apiService from "../services/apiService";

export default {
  data() {
    return {
      nutricionistas: [], // Lista de nutricionistas
      nutricionistaSeleccionado: "", // Nutricionista seleccionado
      turnosDisponibles: [], // Todos los turnos disponibles
      turnosMatutinos: [], // Turnos matutinos
      turnosVespertinos: [], // Turnos vespertinos
      turnosReservados: [], // Turnos reservados por el usuario
      message: "", // Mensaje de estado
      isSuccessMessage: true, // Control de mensaje (éxito/error)
    };
  },
  async mounted() {
    await this.cargarNutricionistas(); // Cargar nutricionistas al montar
  },
  methods: {
    ordenarTurnos(turnos) {
      // Ordenar los turnos por fecha y hora en orden ascendente
      return turnos.sort((a, b) => {
        const fechaA = new Date(`${a.fecha}T${a.hora}`);
        const fechaB = new Date(`${b.fecha}T${b.hora}`);
        return fechaA - fechaB;
      });
    },
    async cargarNutricionistas() {
      try {
        const response = await apiService.get("/nutricionistas");
        this.nutricionistas = response.data;
      } catch (err) {
        this.message = "Error al cargar la lista de nutricionistas.";
        this.isSuccessMessage = false;
      }
    },
    async cargarTurnos() {
      if (!this.nutricionistaSeleccionado) {
        this.turnosDisponibles = [];
        this.turnosReservados = [];
        this.turnosMatutinos = [];
        this.turnosVespertinos = [];
        return;
      }
      try {
        const response = await apiService.get(`/turnos/nutricionista/${this.nutricionistaSeleccionado}`);

        // Separar turnos disponibles y reservados
        const { disponibles, reservados } = response.data;

        // Ordenar turnos
        this.turnosDisponibles = this.ordenarTurnos(disponibles);
        this.turnosReservados = this.ordenarTurnos(reservados);

        // Clasificar turnos disponibles en matutinos y vespertinos
        this.turnosMatutinos = this.turnosDisponibles.filter((turno) => {
          const hora = parseInt(turno.hora.split(":")[0]);
          return hora >= 8 && hora < 12;
        });

        this.turnosVespertinos = this.turnosDisponibles.filter((turno) => {
          const hora = parseInt(turno.hora.split(":")[0]);
          return hora >= 14 && hora < 18;
        });

        this.message = ""; // Limpiar mensajes
      } catch (err) {
        this.message = "Error al cargar los turnos.";
        this.isSuccessMessage = false;
      }
    },
    async reservarTurno(turnoId) {
      try {
        const response = await apiService.post("/turnos/reservar", { turno_id: turnoId });
        this.message = response.data.message;
        this.isSuccessMessage = true;

        // Refrescar turnos después de reservar
        await this.cargarTurnos();
      } catch (err) {
        this.message = err.response?.data?.message || "Error al reservar el turno.";
        this.isSuccessMessage = false;
      }
    },
    async cancelarTurno(turnoId) {
      try {
        const response = await apiService.delete(`/turnos/${turnoId}`);
        this.message = response.data.message;
        this.isSuccessMessage = true;

        // Refrescar turnos después de cancelar
        await this.cargarTurnos();
      } catch (err) {
        this.message = err.response?.data?.message || "Error al cancelar el turno.";
        this.isSuccessMessage = false;
      }
    },
  },
};
</script>

<style scoped>
.appointment-container {
  padding: 20px;
}

.appointment-container h2{
  display: flex;
  justify-content: center;
}
.nutricionistas-section {
  margin-bottom: 20px;
}

.turnos-container {
  display: flex;
  gap: 20px; /* Espaciado entre columnas */
  flex-wrap: wrap; /* Permite que las columnas se ajusten en pantallas pequeñas */
}

.turnos-column {
  flex: 1 1 45%; /* Ajuste flexible para columnas */
  padding: 15px;
  border: 1px solid #ccc;
  border-radius: 8px;
  background-color: #f9f9f9;
  margin-bottom: 20px;
}

.turnos-column h3 {
  margin-bottom: 10px;
  font-size: 18px;
}

.turnos-column ul {
  list-style: none;
  padding: 0;
}

.turnos-column li {
  margin-bottom: 10px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.turnos-column button {
  padding: 5px 10px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 5px;
}

.turnos-column button:hover {
  background-color: #0056b3;
}

.message {
  margin-top: 20px;
  padding: 10px;
  border-radius: 5px;
  text-align: center;
}

.message.success {
  background-color: #28a745;
  color: white;
}

.message.error {
  background-color: #d9534f;
  color: white;
}

/* Estilos responsivos */
@media (max-width: 768px) {
  .turnos-column {
    flex: 1 1 100%;
  }
}
</style>
