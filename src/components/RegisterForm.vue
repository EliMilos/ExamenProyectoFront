<template>
  <div class="register-page">
    <!-- Modal de éxito -->
    <div class="modal fade" id="successModal" tabindex="-1" aria-labelledby="successModalLabel" aria-hidden="true">
      <div class="modal-dialog modal-dialog-centered">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title" id="successModalLabel">¡Registro exitoso!</h5>
          </div>
          <div class="modal-body">
            Tu cuenta ha sido creada correctamente.
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-primary w-100" @click="successModal.hide()">
              Cerrar
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Modal de error para errores de servidor -->
    <div class="modal fade" id="errorModal" tabindex="-1" aria-labelledby="errorModalLabel" aria-hidden="true">
      <div class="modal-dialog modal-dialog-centered">
        <div class="modal-content border-danger">
          <div class="modal-header bg-danger text-white">
            <h5 class="modal-title" id="errorModalLabel">Error</h5>
            <button type="button" class="btn-close btn-close-white" data-bs-dismiss="modal" aria-label="Cerrar"></button>
          </div>
          <div class="modal-body">
            {{ errorMessage }}
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-danger w-100" data-bs-dismiss="modal">Cerrar</button>
          </div>
        </div>
      </div>
    </div>

    <!-- Formulario de registro -->
    <div class="overlay d-flex justify-content-center align-items-center min-vh-100">
      <div class="card shadow-lg border-0" style="width: 100%; max-width: 400px;">
        <div class="card-body p-4">
          <div class="text-center mb-4">
            <img src="https://cdn-icons-png.flaticon.com/512/3135/3135715.png" alt="Register Icon" width="60" class="mb-2" />
            <h2 class="mb-0 fw-bold">Crear Cuenta</h2>
            <p class="text-muted mb-0" style="font-size: 0.95rem;">Regístrate para continuar</p>
          </div>
          <form @submit.prevent="register" autocomplete="off" novalidate>
            <div class="mb-3">
              <label for="email" class="form-label">Correo electrónico</label>
              <input
                v-model="email"
                type="email"
                class="form-control"
                id="email"
                placeholder="Ingresa tu correo"
                required
              />
              <div v-if="emailInvalid" class="text-danger small mt-1">
                El correo no tiene un formato válido.
              </div>
            </div>

            <div class="mb-3">
              <label for="password" class="form-label">Contraseña</label>
              <input
                v-model="password"
                type="password"
                class="form-control"
                id="password"
                placeholder="Crea una contraseña"
                required
              />
              <div v-if="passwordInvalid" class="text-danger small mt-1">
                La contraseña debe tener mínimo 8 caracteres, incluyendo mayúsculas, minúsculas, números y símbolos.
              </div>
            </div>

            <div class="mb-3">
              <label for="confirmPassword" class="form-label">Confirmar contraseña</label>
              <input
                v-model="confirmPassword"
                type="password"
                class="form-control"
                id="confirmPassword"
                placeholder="Repite tu contraseña"
                required
              />
              <div v-if="passwordMismatch" class="text-danger small mt-1">
                Las contraseñas no coinciden.
              </div>
            </div>

            <div class="mb-3">
              <label for="cedula" class="form-label">Cédula</label>
              <input
                v-model="cedula"
                type="text"
                class="form-control"
                id="cedula"
                placeholder="Ingresa tu cédula"
                maxlength="10"
                required
              />
              <div v-if="cedulaInvalid" class="text-danger small mt-1">
                La cédula debe tener exactamente 10 dígitos numéricos.
              </div>
            </div>

            <button type="submit" class="btn btn-primary w-100 fw-bold">Registrarse</button>
          </form>

          <div class="text-center mt-3">
            <router-link to="/login" class="small text-decoration-none text-secondary">
              ¿Ya tienes cuenta? Inicia sesión
            </router-link>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { nextTick } from 'vue';

export default {
  name: 'RegisterForm',
  data() {
    return {
      email: '',
      password: '',
      confirmPassword: '',
      cedula: '',
      successModal: null,
      errorModal: null,
      errorMessage: '',
      passwordMismatch: false,
      emailInvalid: false,
      passwordInvalid: false,
      cedulaInvalid: false
    };
  },
  mounted() {
    nextTick(() => {
      this.successModal = new window.bootstrap.Modal(document.getElementById('successModal'));
      this.errorModal = new window.bootstrap.Modal(document.getElementById('errorModal'));
    });
  },
  methods: {
    validateEmail(email) {
      const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      return re.test(email);
    },
    validatePassword(password) {
      const re = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[\W_]).{8,}$/;
      return re.test(password);
    },
    validateCedula(cedula) {
      const re = /^\d{10}$/;
      return re.test(cedula);
    },
    async register() {
      // Reset errores
      this.passwordMismatch = false;
      this.emailInvalid = false;
      this.passwordInvalid = false;
      this.cedulaInvalid = false;
      this.errorMessage = '';

      // Validaciones frontend
      if (!this.validateEmail(this.email)) {
        this.emailInvalid = true;
        return;
      }

      if (!this.validatePassword(this.password)) {
        this.passwordInvalid = true;
        return;
      }

      if (this.password !== this.confirmPassword) {
        this.passwordMismatch = true;
        return;
      }

      if (!this.validateCedula(this.cedula)) {
        this.cedulaInvalid = true;
        return;
      }

      // Si pasa validaciones locales, hacer petición backend
      try {
        const response = await fetch('http://localhost:5265/api/auth/register', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            email: this.email.toLowerCase(),
            password: this.password,
            cedula: this.cedula
          })
        });

        const message = await response.text();

        if (response.ok) {
          this.successModal.show();
          // limpiar formulario
          this.email = '';
          this.password = '';
          this.confirmPassword = '';
          this.cedula = '';
        } else {
          this.errorMessage = message;
          this.errorModal.show();
        }
      } catch (error) {
        this.errorMessage = 'Error de conexión con el servidor';
        this.errorModal.show();
      }
    }
  }
};
</script>


<style scoped>
.register-page {
  background-image: url('https://images.unsplash.com/photo-1494172961521-33799ddd43a5?auto=format&fit=crop&w=1600&q=80');
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  min-height: 100vh;
}

.overlay {
  background-color: rgba(255, 255, 255, 0.9);
  display: flex;
  justify-content: center;
  align-items: flex-start;
  min-height: 100vh;
  padding-top: 3rem;
}

.card {
  border-radius: 1rem;
  margin-bottom: 3rem;
}
</style>
