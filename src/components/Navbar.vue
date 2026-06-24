<template>
  <nav class="navbar" :class="temaClass">
    <div class="navbar-container">
      <button class="navbar-brand navbar-button" @click="irInicio">
        <span class="brand-icon">🌤️</span>

        <span class="brand-copy">
          <span class="brand-text">ClimaApp</span>
          <small>Tu clima, a un clic</small>
        </span>
      </button>

      <div class="navbar-links">
        <button
          class="nav-link nav-button"
          :class="{ 'nav-link-active': $route.path === '/' }"
          @click="irInicio"
        >
          <span>🏠</span>
          Inicio
        </button>

        <RouterLink
          v-if="isAuthenticated"
          to="/favoritos"
          class="nav-link"
          active-class="nav-link-active"
        >
          <span>❤️</span>
          Favoritos
          <span v-if="cantidadFavoritos > 0" class="fav-count">
            {{ cantidadFavoritos }}
          </span>
        </RouterLink>

        <RouterLink
          v-if="isAuthenticated"
          to="/perfil"
          class="nav-link"
          active-class="nav-link-active"
        >
          <span>👤</span>
          Perfil
        </RouterLink>
      </div>

      <div class="navbar-user">
        <template v-if="isAuthenticated">
          <div class="user-info">
            <div class="user-avatar">{{ iniciales }}</div>

            <div class="user-copy">
              <span class="user-label">Sesión activa</span>
              <span class="user-name">{{ usuarioActual.nombre }}</span>
            </div>
          </div>

          <button class="btn-logout" @click="cerrarSesion">
            Salir
          </button>
        </template>

        <template v-else>
          <RouterLink to="/login" class="btn-login">
            Iniciar sesión
          </RouterLink>

          <RouterLink to="/registro" class="btn-register">
            Registrarse
          </RouterLink>
        </template>
      </div>
    </div>
  </nav>
</template>

<script>
import { mapGetters } from 'vuex'

export default {
  name: 'Navbar',

  computed: {
    ...mapGetters('auth', [
      'isAuthenticated',
      'usuarioActual',
      'favoritos'
    ]),

    temaClass() {
      return this.usuarioActual?.preferencias?.tema === 'light'
        ? 'navbar-claro'
        : 'navbar-oscuro'
    },

    iniciales() {
      if (!this.usuarioActual) return ''
      return this.usuarioActual.nombre.substring(0, 2).toUpperCase()
    },

    cantidadFavoritos() {
      return this.favoritos?.length || 0
    }
  },

  methods: {
    irInicio() {
      this.$store.commit('weather/LIMPIAR_CLIMA')

      if (this.$route.path !== '/') {
        this.$router.push('/')
      }

      window.scrollTo({
        top: 0,
        behavior: 'smooth'
      })
    },

    async cerrarSesion() {
      await this.$store.dispatch('auth/logout')
      this.$store.commit('weather/LIMPIAR_CLIMA')
      this.$router.push('/login')
    }
  }
}
</script>

<style scoped>
.navbar {
  position: sticky;
  top: 0;
  z-index: 100;
  padding: 10px 16px;
  backdrop-filter: blur(18px);
  border-bottom: 1px solid rgba(148, 163, 184, 0.18);
}

.navbar-oscuro {
  background:
    linear-gradient(135deg, rgba(15, 23, 42, 0.92), rgba(30, 64, 175, 0.82));
}

.navbar-claro {
  background: rgba(248, 250, 252, 0.88);
}

.navbar-container {
  max-width: 1200px;
  margin: 0 auto;
  min-height: 66px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 22px;
}

.navbar-brand {
  display: flex;
  align-items: center;
  gap: 11px;
  text-decoration: none;
  flex-shrink: 0;
}

.navbar-button,
.nav-button {
  border: none;
  background: transparent;
  font-family: inherit;
  cursor: pointer;
}

.brand-icon {
  width: 44px;
  height: 44px;
  display: grid;
  place-items: center;
  border-radius: 16px;
  font-size: 24px;
  background: rgba(255, 255, 255, 0.14);
  box-shadow: inset 0 0 0 1px rgba(255, 255, 255, 0.16);
}

.brand-copy {
  display: grid;
  line-height: 1.05;
}

.brand-text {
  font-size: 19px;
  font-weight: 900;
  background: linear-gradient(135deg, #bfdbfe, #ffffff);
  background-clip: text;
  -webkit-background-clip: text;
  color: transparent;
  -webkit-text-fill-color: transparent;
}

.brand-copy small {
  margin-top: 3px;
  color: #bfdbfe;
  font-size: 0.68rem;
  font-weight: 800;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

.navbar-links {
  display: flex;
  align-items: center;
  gap: 8px;
  flex: 1;
  justify-content: center;
}

.nav-link {
  min-height: 40px;
  display: inline-flex;
  align-items: center;
  gap: 7px;
  padding: 9px 15px;
  border-radius: 999px;
  text-decoration: none;
  font-size: 14px;
  font-weight: 800;
  color: #dbeafe;
  transition: 0.25s ease;
  position: relative;
  white-space: nowrap;
}

.navbar-claro .nav-link {
  color: #1e293b;
}

.nav-link:hover {
  background: rgba(255, 255, 255, 0.14);
  color: white;
  transform: translateY(-2px);
}

.navbar-claro .nav-link:hover {
  background: #eff6ff;
  color: #2563eb;
}

.nav-link-active {
  background: rgba(96, 165, 250, 0.22);
  color: white !important;
  box-shadow: inset 0 0 0 1px rgba(147, 197, 253, 0.28);
}

.navbar-claro .nav-link-active {
  background: #dbeafe;
  color: #2563eb !important;
}

.fav-count {
  min-width: 20px;
  height: 20px;
  display: inline-grid;
  place-items: center;
  background: #ef4444;
  color: white;
  font-size: 11px;
  font-weight: 900;
  border-radius: 999px;
  padding: 0 6px;
}

.navbar-user {
  display: flex;
  align-items: center;
  gap: 10px;
  flex-shrink: 0;
}

.user-info {
  display: flex;
  align-items: center;
  gap: 9px;
  padding: 6px 10px 6px 6px;
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.12);
  border: 1px solid rgba(255, 255, 255, 0.14);
}

.navbar-claro .user-info {
  background: white;
  border-color: #dbeafe;
}

.user-avatar {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  background: linear-gradient(135deg, #3b82f6, #8b5cf6);
  display: grid;
  place-items: center;
  font-size: 13px;
  color: white;
  font-weight: 900;
}

.user-copy {
  display: grid;
  line-height: 1.05;
}

.user-label {
  color: #bfdbfe;
  font-size: 0.66rem;
  font-weight: 800;
  text-transform: uppercase;
}

.navbar-claro .user-label {
  color: #64748b;
}

.user-name {
  margin-top: 3px;
  font-size: 13px;
  font-weight: 900;
  color: white;
}

.navbar-claro .user-name {
  color: #0f172a;
}

.btn-logout,
.btn-login,
.btn-register {
  min-height: 40px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  border-radius: 999px;
  padding: 9px 16px;
  text-decoration: none;
  font-size: 13px;
  font-weight: 900;
  cursor: pointer;
  transition: 0.25s;
  font-family: inherit;
  white-space: nowrap;
}

.btn-login {
  border: 1px solid rgba(191, 219, 254, 0.5);
  color: #dbeafe;
  background: rgba(255, 255, 255, 0.08);
}

.btn-register {
  border: none;
  color: white;
  background: linear-gradient(135deg, #2563eb, #7c3aed);
}

.btn-logout {
  border: 1px solid rgba(248, 113, 113, 0.45);
  background: rgba(239, 68, 68, 0.12);
  color: #fecaca;
}

.navbar-claro .btn-login {
  color: #2563eb;
  background: white;
  border-color: #bfdbfe;
}

.btn-login:hover,
.btn-register:hover,
.btn-logout:hover {
  transform: translateY(-2px);
}

.btn-logout:hover {
  background: rgba(239, 68, 68, 0.2);
}

@media (max-width: 900px) {
  .navbar-container {
    flex-wrap: wrap;
    gap: 12px;
  }

  .navbar-brand {
    order: 1;
  }

  .navbar-user {
    order: 2;
    margin-left: auto;
  }

  .navbar-links {
    order: 3;
    width: 100%;
    justify-content: center;
    overflow-x: auto;
    padding-bottom: 2px;
  }
}

@media (max-width: 560px) {
  .navbar {
    padding: 8px 10px;
  }

  .brand-icon {
    width: 40px;
    height: 40px;
    border-radius: 14px;
  }

  .brand-text {
    font-size: 17px;
  }

  .brand-copy small,
  .user-copy {
    display: none;
  }

  .navbar-user {
    gap: 6px;
  }

  .nav-link {
    font-size: 12px;
    padding: 8px 11px;
  }

  .btn-login,
  .btn-register,
  .btn-logout {
    min-height: 36px;
    padding: 8px 11px;
    font-size: 12px;
  }
}
</style>