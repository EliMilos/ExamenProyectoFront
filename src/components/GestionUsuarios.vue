<template>
  <div class="container-fluid py-5">
    <div class="row justify-content-center">
      <div class="col-12 col-xl-10">
        <h2 class="text-center fw-bold mb-5 gradient-title">
          <i class="fas fa-users me-3"></i>Gestión de Usuarios
        </h2>

        <!-- Formulario de Nuevo Usuario -->
        <div class="card modern-card mb-5">
          <div class="card-header bg-gradient-primary">
            <h5 class="card-title text-white mb-0">
              <i class="fas fa-user-plus me-2"></i>Nuevo Usuario
            </h5>
          </div>
          <div class="card-body">
            <form @submit.prevent="crearUsuario">
              <div class="row g-4">
                <div class="col-md-6">
                  <div class="form-floating">
                    <input
                      v-model="nuevoUsuario.cedula"
                      type="text"
                      class="form-control modern-input"
                      id="cedula"
                      placeholder="Cédula"
                      required
                    />
                    <label for="cedula">
                      <i class="fas fa-id-card me-2"></i>Cédula
                    </label>
                  </div>
                </div>
                <div class="col-md-6">
                  <div class="form-floating">
                    <input
                      v-model="nuevoUsuario.email"
                      type="email"
                      class="form-control modern-input"
                      id="email"
                      placeholder="Correo"
                      required
                    />
                    <label for="email">
                      <i class="fas fa-envelope me-2"></i>Correo Electrónico
                    </label>
                  </div>
                </div>
                <div class="col-md-6">
                  <div class="form-floating">
                    <input
                      v-model="nuevoUsuario.password"
                      type="password"
                      class="form-control modern-input"
                      id="password"
                      placeholder="Contraseña"
                      required
                    />
                    <label for="password">
                      <i class="fas fa-lock me-2"></i>Contraseña
                    </label>
                  </div>
                </div>
                <div class="col-md-6">
                  <div class="form-floating">
                    <select
                      v-model="nuevoUsuario.role"
                      class="form-select modern-input"
                      id="role"
                      required
                    >
                      <option value="">Seleccionar Rol</option>
                      <option value="Admin">Administrador</option>
                      <option value="Usuario">Usuario</option>
                    </select>
                    <label for="role">
                      <i class="fas fa-user-tag me-2"></i>Rol
                    </label>
                  </div>
                </div>
                <div class="col-12 text-end">
                  <button
                    type="submit"
                    class="btn btn-success modern-btn"
                    :disabled="loading"
                  >
                    <i class="fas fa-plus me-2"></i>
                    <span v-if="loading">Creando...</span>
                    <span v-else>Crear Usuario</span>
                  </button>
                </div>
              </div>
            </form>
          </div>
        </div>

        <!-- Lista de Usuarios -->
        <div class="card modern-card">
          <div class="card-header bg-gradient-info">
            <h5 class="card-title text-white mb-0">
              <i class="fas fa-list me-2"></i>Lista de Usuarios
              <span class="badge bg-light text-dark ms-2">{{ usuarios.length }}</span>
            </h5>
          </div>
          <div class="card-body">
            <div class="table-responsive">
              <table class="table table-hover modern-table">
                <thead class="table-dark">
                  <tr>
                    <th class="text-center">#</th>
                    <th><i class="fas fa-id-card me-2"></i>Cédula</th>
                    <th><i class="fas fa-envelope me-2"></i>Correo</th>
                    <th><i class="fas fa-user-tag me-2"></i>Rol</th>
                    <th class="text-center">
                      <i class="fas fa-cogs me-2"></i>Acciones
                    </th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="(user, index) in usuarios" :key="user.id" class="table-row">
                    <td class="text-center fw-bold">{{ index + 1 }}</td>
                    <td><span class="user-info">{{ user.cedula }}</span></td>
                    <td><span class="user-email">{{ user.email }}</span></td>
                    <td>
                      <span
                        class="badge"
                        :class="user.role === 'Admin' ? 'bg-warning text-dark' : 'bg-info'"
                      >
                        <i
                          :class="user.role === 'Admin' ? 'fas fa-crown' : 'fas fa-user'"
                          class="me-1"
                        ></i>
                        {{ user.role }}
                      </span>
                    </td>
                    <td class="text-center">
                      <div class="action-buttons">
                        <button
                          class="btn-action btn-edit"
                          @click="editar(user)"
                          :disabled="loading"
                          title="Editar"
                        >
                          <i class="fas fa-edit"></i>
                          Editar
                        </button>
                        <button
                          class="btn-action btn-delete"
                          @click="confirmarEliminacion(user)"
                          :disabled="loading"
                          title="Eliminar"
                        >
                          <i class="fas fa-trash"></i>
                          Eliminar
                        </button>
                      </div>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Modal de Edición -->
    <div class="modal fade" ref="editarModal" tabindex="-1" aria-hidden="true">
      <div class="modal-dialog modal-lg">
        <div class="modal-content modern-modal">
          <form @submit.prevent="actualizarUsuario">
            <div class="modal-header bg-gradient-primary">
              <h5 class="modal-title text-white">
                <i class="fas fa-user-edit me-2"></i>Editar Usuario
              </h5>
              <button
                type="button"
                class="btn-close btn-close-white"
                @click="cerrarModal"
              ></button>
            </div>
            <div class="modal-body">
              <div class="row g-3">
                <div class="col-md-6">
                  <div class="form-floating">
                    <input
                      v-model="usuarioEdit.cedula"
                      class="form-control modern-input"
                      id="editCedula"
                      placeholder="Cédula"
                      required
                    />
                    <label for="editCedula">
                      <i class="fas fa-id-card me-2"></i>Cédula
                    </label>
                  </div>
                </div>
                <div class="col-md-6">
                  <div class="form-floating">
                    <input
                      v-model="usuarioEdit.email"
                      class="form-control modern-input"
                      id="editEmail"
                      type="email"
                      placeholder="Correo"
                      required
                    />
                    <label for="editEmail">
                      <i class="fas fa-envelope me-2"></i>Correo
                    </label>
                  </div>
                </div>
                <div class="col-md-6">
                  <div class="form-floating">
                    <select
                      v-model="usuarioEdit.role"
                      class="form-select modern-input"
                      id="editRole"
                      required
                    >
                      <option value="Admin">Administrador</option>
                      <option value="Usuario">Usuario</option>
                    </select>
                    <label for="editRole">
                      <i class="fas fa-user-tag me-2"></i>Rol
                    </label>
                  </div>
                </div>


<!-- Nueva contraseña -->


                <div class="col-md-6">
                <label class="form-label">Nueva Contraseña</label>
                <div class="input-group">
                  <input
                    :type="mostrarPasswordEdit ? 'text' : 'password'"
                    class="form-control modern-input"
                    v-model="usuarioEdit.password"
                    placeholder="Nueva contraseña"
                  />
                <button class="btn btn-outline-secondary" type="button" @click="mostrarPasswordEdit = !mostrarPasswordEdit">
              <i :class="mostrarPasswordEdit ? 'bi bi-eye-slash' : 'bi bi-eye'"></i>
            </button>
          </div>
        </div>



<!-- Confirmar contraseña -->


          <div class="col-md-6">
            <label class="form-label">Confirmar Contraseña</label>
          <div class="input-group">
            <input
              :type="mostrarConfirmPasswordEdit ? 'text' : 'password'"
              class="form-control modern-input"
              v-model="usuarioEdit.confirmPassword"
              placeholder="Confirmar contraseña"
            />
          <button class="btn btn-outline-secondary" type="button" @click="mostrarConfirmPasswordEdit = !mostrarConfirmPasswordEdit">
          <i :class="mostrarConfirmPasswordEdit ? 'bi bi-eye-slash' : 'bi bi-eye'"></i>
        </button>
      </div>
      </div>

              </div>
            </div>
            <div class="modal-footer">
              <button
                type="submit"
                class="btn btn-primary modern-btn"
                :disabled="loading"
              >
                <i class="fas fa-save me-2"></i>
                <span v-if="loading">Guardando...</span>
                <span v-else>Guardar Cambios</span>
              </button>
              <button
                type="button"
                class="btn btn-secondary modern-btn"
                @click="cerrarModal"
              >
                <i class="fas fa-times me-2"></i>Cancelar
              </button>
            </div>
          </form>
        </div>
      </div>
    </div>

    <!-- Modal de Confirmación de Eliminación -->
    <div class="modal fade" ref="confirmarModal" tabindex="-1" aria-hidden="true">
      <div class="modal-dialog">
        <div class="modal-content modern-modal">
          <div class="modal-header bg-gradient-danger">
            <h5 class="modal-title text-white">
              <i class="fas fa-exclamation-triangle me-2"></i>Confirmar Eliminación
            </h5>
            <button
              type="button"
              class="btn-close btn-close-white"
              @click="cerrarConfirmacion"
            ></button>
          </div>
          <div class="modal-body text-center">
            <div class="mb-3">
              <i class="fas fa-user-times fa-3x text-danger"></i>
            </div>
            <h6>¿Estás seguro de que deseas eliminar este usuario?</h6>
            <p class="text-muted mb-0" v-if="usuarioAEliminar">
              <strong>{{ usuarioAEliminar.email }}</strong>
            </p>
            <small class="text-warning">Esta acción no se puede deshacer.</small>
          </div>
          <div class="modal-footer">
            <button
              type="button"
              class="btn btn-danger modern-btn"
              @click="eliminarUsuario"
              :disabled="loading"
            >
              <i class="fas fa-trash me-2"></i>
              <span v-if="loading">Eliminando...</span>
              <span v-else>Eliminar</span>
            </button>
            <button
              type="button"
              class="btn btn-secondary modern-btn"
              @click="cerrarConfirmacion"
            >
              <i class="fas fa-times me-2"></i>Cancelar
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>


<script>
export default {
  data() {
    return {
      usuarios: [],
      nuevoUsuario: { cedula: "", email: "", password: "", role: "" },
      usuarioEdit: {
        id: "",
        cedula: "",
        email: "",
        role: "",
        concurrencyStamp: "",
        password: "",
        confirmPassword: ""
      },
      usuarioAEliminar: null,
      editarModalInstance: null,
      confirmarModalInstance: null,
      loading: false,


      /////////////////////////////////////
      mostrarPasswordEdit: false,
      mostrarConfirmPasswordEdit: false,

    };
  },
  mounted() {
    this.obtenerUsuarios();
    this.inicializarModales();
  },
  methods: {
    inicializarModales() {
      if (window.bootstrap) {
        this.editarModalInstance = new window.bootstrap.Modal(this.$refs.editarModal);
        this.confirmarModalInstance = new window.bootstrap.Modal(this.$refs.confirmarModal);
      }
    },

    async obtenerUsuarios() {
      try {
        this.loading = true;
        const token = localStorage.getItem("token");
        const res = await fetch("http://localhost:5265/api/Auth", {
          headers: { Authorization: `Bearer ${token}` },
        });

        if (!res.ok) throw new Error("Error al obtener usuarios");

        const data = await res.json();
        this.usuarios = data.map((u) => ({
          id: u.id,
          cedula: u.cedula,
          email: u.email,
          lockoutEnabled: u.lockoutEnabled,
          concurrencyStamp: u.concurrencyStamp,
          role: u.role,
        }));
      } catch (error) {
        console.error("Error:", error);
        this.mostrarNotificacion("Error al cargar usuarios", "error");
      } finally {
        this.loading = false;
      }
    },

    async editar(user) {
      try {
        this.loading = true;
        const token = localStorage.getItem("token");

        const res = await fetch(`http://localhost:5265/api/Auth/${user.id}`, {
          headers: { Authorization: `Bearer ${token}` },
        });

        const data = await res.json();

        if (!res.ok) {
          this.mostrarNotificacion(data.message || "Error al obtener usuario", "error");
          return;
        }

        this.usuarioEdit = {
          id: data.id,
          cedula: data.cedula,
          email: data.email,
          concurrencyStamp: data.concurrencyStamp,
          role: data.role || "Usuario",
          password: "",
          confirmPassword: ""
        };

        this.editarModalInstance?.show();
      } catch (error) {
        console.error("Error:", error);
        this.mostrarNotificacion("Error de conexión al obtener usuario", "error");
      } finally {
        this.loading = false;
      }
    },

    cerrarModal() {
      this.editarModalInstance?.hide();
      this.usuarioEdit = {
        id: "",
        cedula: "",
        email: "",
        role: "",
        concurrencyStamp: "",
        password: "",
        confirmPassword: ""
      };
    },

    async actualizarUsuario() {
      try {
        this.loading = true;
        const token = localStorage.getItem("token");

        const { password, confirmPassword } = this.usuarioEdit;

        if (password) {
          if (password !== confirmPassword)
            return this.mostrarNotificacion("Las contraseñas no coinciden.", "error");
          if (password.length < 6)
            return this.mostrarNotificacion("Debe tener al menos 6 caracteres.", "error");
          if (password.length > 10)
            return this.mostrarNotificacion("No debe exceder 10 caracteres.", "error");
          if (!/[A-Z]/.test(password))
            return this.mostrarNotificacion("Debe tener una letra mayúscula.", "error");
          if (!/[a-z]/.test(password))
            return this.mostrarNotificacion("Debe tener una letra minúscula.", "error");
          if (!/\d/.test(password))
            return this.mostrarNotificacion("Debe tener al menos un número.", "error");
          if (!/[\W_]/.test(password))
            return this.mostrarNotificacion("Debe tener un carácter especial.", "error");
        }

        const datosActualizacion = {
          userId: this.usuarioEdit.id,
          cedula: this.usuarioEdit.cedula,
          email: this.usuarioEdit.email,
          concurrencyStamp: this.usuarioEdit.concurrencyStamp,
          role: this.usuarioEdit.role,
          newPassword: password || null
        };

        const res = await fetch(`http://localhost:5265/api/Auth/${this.usuarioEdit.id}`, {
          method: "PUT",
          headers: {
            "Content-Type": "application/json",
            Authorization: `Bearer ${token}`,
          },
          body: JSON.stringify(datosActualizacion),
        });

        const response = await res.json();

        if (res.ok) {
          const index = this.usuarios.findIndex((u) => u.id === this.usuarioEdit.id);
          if (index !== -1) {
            this.usuarios[index].cedula = this.usuarioEdit.cedula;
            this.usuarios[index].email = this.usuarioEdit.email;
            this.usuarios[index].concurrencyStamp = response.concurrencyStamp;
            this.usuarios[index].role = this.usuarioEdit.role;
          }

          this.mostrarNotificacion(response.message || "Usuario actualizado", "success");
          this.cerrarModal();
        } else {
          this.mostrarNotificacion(response.message || "Error al actualizar", "error");
        }
      } catch (error) {
        console.error("Error:", error);
        this.mostrarNotificacion("Error de conexión", "error");
      } finally {
        this.loading = false;
      }
    },

    // Eliminar, Crear, Validar, Notificaciones (no se modifican)
    async crearUsuario() { /* sin cambios */ },
    validarUsuario() { /* sin cambios */ },
    confirmarEliminacion(user) { this.usuarioAEliminar = user; this.confirmarModalInstance?.show(); },
    cerrarConfirmacion() { this.confirmarModalInstance?.hide(); this.usuarioAEliminar = null; },
    async eliminarUsuario() { /* sin cambios */ },
    mostrarNotificacion(mensaje, tipo = "info") {
      const alertClass =
        tipo === "success"
          ? "alert-success"
          : tipo === "error"
          ? "alert-danger"
          : "alert-info";
      const notification = document.createElement("div");
      notification.className = `alert ${alertClass} alert-dismissible fade show position-fixed`;
      notification.style.cssText =
        "top: 20px; right: 20px; z-index: 9999; min-width: 300px;";
      notification.innerHTML = `${mensaje}<button type="button" class="btn-close" data-bs-dismiss="alert"></button>`;
      document.body.appendChild(notification);
      setTimeout(() => notification.remove(), 5000);
    },
  },
};
</script>


<style scoped>
/* Gradientes y colores modernos */
.gradient-title {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.bg-gradient-primary {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.bg-gradient-info {
  background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
}

.bg-gradient-danger {
  background: linear-gradient(135deg, #ff6b6b 0%, #ee5a24 100%);
}

/* Cards modernos */
.modern-card {
  border: none;
  border-radius: 20px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  transition: all 0.3s ease;
}

.modern-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.15);
}

.modern-card .card-header {
  border: none;
  padding: 1.5rem;
}

.modern-card .card-body {
  padding: 2rem;
}

/* Inputs modernos */
.modern-input {
  border-radius: 15px;
  border: 2px solid #e9ecef;
  transition: all 0.3s ease;
  padding: 0.75rem 1rem;
}

.modern-input:focus {
  border-color: #667eea;
  box-shadow: 0 0 0 0.2rem rgba(102, 126, 234, 0.25);
  transform: translateY(-2px);
}

.form-floating > .modern-input:focus ~ label,
.form-floating > .modern-input:not(:placeholder-shown) ~ label {
  color: #667eea;
}

/* Botones modernos */
.modern-btn {
  border-radius: 50px;
  padding: 0.75rem 2rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  transition: all 0.3s ease;
  border: none;
  position: relative;
  overflow: hidden;
}

.modern-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
}

.modern-btn:active {
  transform: translateY(0);
}

/* Estilos para los botones de acción específicos */
.action-buttons {
  display: flex;
  gap: 8px;
  justify-content: center;
  align-items: center;
}

.btn-action {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 6px 12px;
  border: none;
  border-radius: 6px;
  font-size: 12px;
  font-weight: 500;
  text-decoration: none;
  cursor: pointer;
  transition: all 0.2s ease;
  white-space: nowrap;
  min-width: 70px;
  justify-content: center;
}

.btn-action:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

/* Botón Editar - Estilo naranja/amarillo */
.btn-edit {
  background-color: #ff9500;
  color: white;
  border: 1px solid #ff9500;
}

.btn-edit:hover:not(:disabled) {
  background-color: #e8890b;
  border-color: #e8890b;
  transform: translateY(-1px);
  box-shadow: 0 2px 4px rgba(255, 149, 0, 0.3);
}

.btn-edit:active {
  transform: translateY(0);
  box-shadow: 0 1px 2px rgba(255, 149, 0, 0.3);
}

/* Botón Eliminar - Estilo rojo */
.btn-delete {
  background-color: #dc3545;
  color: white;
  border: 1px solid #dc3545;
}

.btn-delete:hover:not(:disabled) {
  background-color: #c82333;
  border-color: #c82333;
  transform: translateY(-1px);
  box-shadow: 0 2px 4px rgba(220, 53, 69, 0.3);
}

.btn-delete:active {
  transform: translateY(0);
  box-shadow: 0 1px 2px rgba(220, 53, 69, 0.3);
}

/* Iconos dentro de los botones */
.btn-action i {
  font-size: 11px;
}

/* Tabla moderna */
.modern-table {
  border-radius: 15px;
  overflow: hidden;
  border: none;
}

.modern-table .table-dark {
  background: linear-gradient(135deg, #2c3e50 0%, #34495e 100%);
}

.table-row {
  transition: all 0.3s ease;
}

.table-row:hover {
  background-color: rgba(102, 126, 234, 0.1);
  transform: scale(1.01);
}

.user-info {
  font-weight: 600;
  color: #2c3e50;
}

.user-email {
  color: #667eea;
  font-size: 0.9rem;
}

/* Modal moderno */
.modern-modal {
  border-radius: 20px;
  border: none;
  overflow: hidden;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.3);
}

.modern-modal .modal-header {
  border: none;
  padding: 1.5rem 2rem;
}

.modern-modal .modal-body {
  padding: 2rem;
}

.modern-modal .modal-footer {
  border: none;
  padding: 1.5rem 2rem;
  background-color: #f8f9fa;
}

/* Badges modernos */
.badge {
  font-size: 0.75rem;
  padding: 0.5rem 1rem;
  border-radius: 50px;
  font-weight: 600;
}

/* Animaciones */
@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.modern-card {
  animation: fadeInUp 0.6s ease;
}

/* Responsive */
@media (max-width: 768px) {
  .modern-card .card-body {
    padding: 1rem;
  }

  .modern-btn {
    padding: 0.5rem 1.5rem;
    font-size: 0.875rem;
  }

  .action-buttons {
    flex-direction: column;
    gap: 4px;
  }

  .btn-action {
    width: 100%;
    min-width: auto;
    font-size: 11px;
    padding: 5px 10px;
  }
}

/* Estados de carga */
.loading {
  opacity: 0.6;
  pointer-events: none;
}

/* Mejoras visuales adicionales */
.container-fluid {
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  min-height: 100vh;
}

.form-check-input:checked {
  background-color: #667eea;
  border-color: #667eea;
}

.text-warning {
  color: #f39c12 !important;
}

/* Efectos de hover para iconos */
.fas {
  transition: all 0.3s ease;
}

.modern-btn:hover .fas,
.btn-action:hover .fas {
  transform: scale(1.1);
}
</style>
