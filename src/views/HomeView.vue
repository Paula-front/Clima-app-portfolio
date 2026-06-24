<template>
  <div class="home-page">
    <section class="hero">
      <div class="hero-panel">
        <div class="hero-copy">
          <p class="hero-kicker">🌤️ ClimaApp · Portafolio Final M8</p>

          <h1>Consulta el clima, pronóstico y alertas en tiempo real.</h1>

          <p class="hero-subtitle">
            Busca una ciudad, revisa el clima actual, estadísticas semanales y guarda tus lugares favoritos.
          </p>

          <div class="search-box">
            <span class="search-icon">🔍</span>

            <input
              v-model="busqueda"
              type="text"
              placeholder="Buscar ciudad: Santiago, Tokio, París..."
              @input="onInput"
              @keyup.enter="buscarCiudad"
            />

            <button
              @click="buscarCiudad"
              :disabled="cargandoBusqueda || !busqueda.trim()"
            >
              {{ cargandoBusqueda ? 'Buscando...' : 'Buscar' }}
            </button>
          </div>

          <div v-if="sugerencias.length" class="sugerencias">
            <button
              v-for="s in sugerencias"
              :key="s.lat + s.lon"
              @click="seleccionarSugerencia(s)"
            >
              📍 {{ s.nombre }}, {{ s.pais }}
              <small v-if="s.region">{{ s.region }}</small>
            </button>
          </div>

          <p v-if="errorBusqueda" class="error-msg">
            ⚠️ {{ errorBusqueda }}
          </p>
        </div>

        <div class="hero-summary">
          <div class="summary-card main">
            <span>🌡️</span>
            <strong>API real</strong>
            <small>Open-Meteo</small>
          </div>

          <div class="summary-card">
            <span>📊</span>
            <strong>Estadísticas</strong>
            <small>Semana completa</small>
          </div>

          <div class="summary-card">
            <span>⚠️</span>
            <strong>Alertas</strong>
            <small>Reglas simples</small>
          </div>
        </div>
      </div>
    </section>

    <section class="quick-actions">
      <button class="action-card" @click="seleccionarCiudad(ciudadesChile[4])">
        <span>🇨🇱</span>
        <div>
          <strong>Santiago</strong>
          <small>Consultar clima actual</small>
        </div>
      </button>

      <button class="action-card" @click="seleccionarCiudad(ciudadesChile[7])">
        <span>🌧️</span>
        <div>
          <strong>Valdivia</strong>
          <small>Ver pronóstico del sur</small>
        </div>
      </button>

      <button class="action-card" @click="seleccionarCiudad(ciudadesChile[11])">
        <span>🌴</span>
        <div>
          <strong>Rapa Nui</strong>
          <small>Explorar clima insular</small>
        </div>
      </button>

      <button class="action-card" @click="seleccionarCiudad(ciudadesMundo[2])">
        <span>🌏</span>
        <div>
          <strong>Tokio</strong>
          <small>Capital internacional</small>
        </div>
      </button>
    </section>

    <section
      v-if="isAuthenticated && favoritos.length"
      class="favorites-section"
    >
      <div class="section-header">
        <div>
          <p class="section-label">Acceso rápido</p>
          <h2>⭐ Tus ciudades favoritas</h2>
        </div>
      </div>

      <div class="favorites-list">
        <button
          v-for="favorito in favoritos"
          :key="favorito.nombre"
          class="favorite-city"
          @click="seleccionarCiudad(favorito)"
        >
          📍 {{ favorito.nombre }}
        </button>
      </div>
    </section>

    <section v-if="!climaActual && !loading" class="explore-section">
      <div class="section-header">
        <div>
          <p class="section-label">Explorar</p>
          <h2>Selecciona una ciudad para comenzar</h2>
        </div>
        <p>
          Puedes buscar cualquier ciudad o elegir una de las sugerencias.
        </p>
      </div>

      <div class="explore-grid">
        <article class="quick-card">
          <h3>🇨🇱 Ciudades de Chile</h3>

          <div class="chips">
            <button
              v-for="ciudad in ciudadesChile"
              :key="ciudad.nombre"
              @click="seleccionarCiudad(ciudad)"
            >
              {{ ciudad.nombre }}
            </button>
          </div>
        </article>

        <article class="quick-card">
          <h3>🌍 Capitales del mundo</h3>

          <div class="chips">
            <button
              v-for="ciudad in ciudadesMundo"
              :key="ciudad.nombre"
              @click="seleccionarCiudad(ciudad)"
            >
              {{ ciudad.nombre }}
            </button>
          </div>
        </article>
      </div>
    </section>

    <section v-if="loading" class="loading-box">
      <div class="spinner"></div>
      <p>Obteniendo clima de {{ ciudadActual?.nombre || 'la ciudad' }}...</p>
    </section>

    <section v-if="climaActual && !loading" class="result-layout">
      <div class="result-heading">
        <div>
          <p class="section-label">Resultado</p>
          <h2>{{ ciudadActual?.nombre }}, {{ ciudadActual?.pais }}</h2>
        </div>

        <button class="back-btn small" @click="limpiarClima">
          ← Buscar otra ciudad
        </button>
      </div>

      <div class="result-grid">
        <div class="result-left">
          <CityImage
            :ciudad="ciudadActual?.nombre"
            :pais="ciudadActual?.pais"
          />

          <div class="map-card">
            <div>
              <h3>📍 Ubicación</h3>
              <p>{{ ciudadActual?.nombre }}, {{ ciudadActual?.pais }}</p>
            </div>

            <div class="map-frame">
              <iframe
                :src="mapUrl"
                width="100%"
                height="230"
                style="border: 0"
                loading="lazy"
                referrerpolicy="no-referrer-when-downgrade"
              ></iframe>
            </div>

            <div class="coords">
              <p>Lat: {{ ciudadActual?.lat }}</p>
              <p>Lon: {{ ciudadActual?.lon }}</p>
            </div>
          </div>
        </div>

        <div class="result-right">
          <WeatherCard
            :ciudad="ciudadActual"
            :clima="climaActual"
            :unidad="unidad"
            @agregar-favorito="agregarAFavoritos"
          />

          <div class="section-card">
            <h3>📅 Pronóstico semanal</h3>

            <div class="forecast-grid">
              <ForecastCard
                v-for="dia in pronostico"
                :key="dia.fecha"
                :dia="dia"
                :unidad="unidad"
              />
            </div>
          </div>

          <WeekStats
            v-if="pronostico.length"
            :pronostico="pronostico"
            :unidad="unidad"
          />

          <div class="alerts">
            <div
              v-for="alerta in alertas"
              :key="alerta.tipo"
              class="alert warning"
            >
              ⚠️ {{ alerta.mensaje }}
            </div>

            <div v-if="!alertas.length" class="alert success">
              ✅ Sin alertas relevantes. Condiciones normales.
            </div>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
import { mapGetters, mapActions } from 'vuex'
import weatherService from '../services/weatherService.js'
import WeatherCard from '../components/WeatherCard.vue'
import ForecastCard from '../components/ForecastCard.vue'
import CityImage from '../components/CityImage.vue'
import WeekStats from '../components/WeekStats.vue'

export default {
  name: 'HomeView',

  components: {
    WeatherCard,
    ForecastCard,
    CityImage,
    WeekStats
  },

  data() {
    return {
      busqueda: '',
      sugerencias: [],
      errorBusqueda: '',
      cargandoBusqueda: false,
      ciudadesChile: weatherService.ciudadesChile,
      ciudadesMundo: weatherService.ciudadesMundo,
      alertas: []
    }
  },

  computed: {
    ...mapGetters('weather', [
      'climaActual',
      'ciudadActual',
      'pronostico',
      'error',
      'loading'
    ]),

    ...mapGetters('auth', [
      'isAuthenticated',
      'preferencias',
      'favoritos'
    ]),

    unidad() {
      return this.preferencias?.unidad || 'C'
    },

    mapUrl() {
      if (!this.ciudadActual?.lat || !this.ciudadActual?.lon) return ''

      const lat = Number(this.ciudadActual.lat)
      const lon = Number(this.ciudadActual.lon)

      return `https://www.openstreetmap.org/export/embed.html?bbox=${lon - 0.25},${lat - 0.25},${lon + 0.25},${lat + 0.25}&layer=mapnik&marker=${lat},${lon}`
    }
  },

  watch: {
    climaActual(nuevo) {
      if (nuevo) {
        this.alertas = weatherService.generarAlertas(nuevo, this.pronostico)
      }
    }
  },

  methods: {
    ...mapActions('weather', ['buscarClima']),
    ...mapActions('auth', ['agregarFavorito']),

    async onInput() {
      this.errorBusqueda = ''

      if (this.busqueda.trim().length < 2) {
        this.sugerencias = []
        return
      }

      try {
        this.sugerencias = await weatherService.buscarCiudad(this.busqueda)
      } catch {
        this.sugerencias = []
      }
    },

    async buscarCiudad() {
      if (!this.busqueda.trim()) return

      this.errorBusqueda = ''
      this.cargandoBusqueda = true

      try {
        const resultados = await weatherService.buscarCiudad(this.busqueda)

        if (resultados.length > 0) {
          await this.seleccionarSugerencia(resultados[0])
        } else {
          this.errorBusqueda = 'No se encontró la ciudad buscada.'
        }
      } catch (error) {
        this.errorBusqueda = error.message || 'No se pudo buscar la ciudad.'
      } finally {
        this.cargandoBusqueda = false
        this.sugerencias = []
      }
    },

    async seleccionarSugerencia(ciudad) {
      this.busqueda = ciudad.nombre
      this.sugerencias = []
      await this.buscarClima(ciudad)
    },

    async seleccionarCiudad(ciudad) {
      this.busqueda = ciudad.nombre
      await this.buscarClima(ciudad)
    },

    agregarAFavoritos(ciudad) {
      if (!this.isAuthenticated) {
        this.$router.push('/login')
        return
      }

      this.agregarFavorito(ciudad)
    },

    limpiarClima() {
      this.$store.commit('weather/LIMPIAR_CLIMA')
      this.busqueda = ''
      this.sugerencias = []
      this.errorBusqueda = ''
      this.alertas = []
    }
  }
}
</script>

<style scoped>
.home-page {
  min-height: 100vh;
  padding: 28px 24px 60px;
  background:
    radial-gradient(circle at top left, rgba(37, 99, 235, 0.18), transparent 28%),
    radial-gradient(circle at top right, rgba(14, 165, 233, 0.16), transparent 30%),
    linear-gradient(135deg, #f8fafc 0%, #eef2ff 48%, #e0f2fe 100%);
}

.hero {
  max-width: 1180px;
  margin: 0 auto 18px;
}

.hero-panel {
  display: grid;
  grid-template-columns: 1.35fr 0.65fr;
  gap: 24px;
  align-items: stretch;
  padding: 34px;
  border-radius: 32px;
  color: white;
  background:
    linear-gradient(135deg, rgba(15, 23, 42, 0.94), rgba(30, 64, 175, 0.82)),
    url('https://images.unsplash.com/photo-1534088568595-a066f410bcda?auto=format&fit=crop&w=1600&q=80');
  background-size: cover;
  background-position: center;
  box-shadow: 0 24px 70px rgba(30, 64, 175, 0.24);
}

.hero-copy {
  position: relative;
}

.hero-kicker,
.section-label {
  font-weight: 900;
  color: #93c5fd;
  text-transform: uppercase;
  letter-spacing: 0.12em;
  font-size: 0.76rem;
  margin-bottom: 10px;
}

.hero h1 {
  max-width: 760px;
  font-size: clamp(2.1rem, 4vw, 4rem);
  line-height: 1.03;
  margin-bottom: 16px;
}

.hero-subtitle {
  max-width: 760px;
  font-size: 1.05rem;
  color: rgba(255, 255, 255, 0.86);
  margin-bottom: 24px;
}

.search-box {
  position: relative;
  display: flex;
  align-items: center;
  gap: 12px;
  max-width: 760px;
  background: rgba(255, 255, 255, 0.16);
  border: 1px solid rgba(255, 255, 255, 0.24);
  backdrop-filter: blur(18px);
  border-radius: 18px;
  padding: 10px;
}

.search-icon {
  font-size: 1.3rem;
  padding-left: 8px;
}

.search-box input {
  flex: 1;
  background: transparent;
  border: none;
  outline: none;
  color: white;
  font-size: 1rem;
}

.search-box input::placeholder {
  color: rgba(255, 255, 255, 0.72);
}

.search-box button,
.back-btn {
  border: none;
  border-radius: 14px;
  padding: 13px 22px;
  font-weight: 900;
  cursor: pointer;
  color: white;
  background: linear-gradient(135deg, #2563eb, #7c3aed);
  transition: 0.25s;
}

.search-box button:disabled {
  opacity: 0.55;
  cursor: not-allowed;
}

.search-box button:hover:not(:disabled),
.back-btn:hover,
.action-card:hover,
.chips button:hover,
.favorite-city:hover {
  transform: translateY(-3px);
}

.sugerencias {
  position: absolute;
  z-index: 30;
  margin-top: 10px;
  width: min(760px, 100%);
  background: white;
  border-radius: 18px;
  overflow: hidden;
  box-shadow: 0 18px 45px rgba(15, 23, 42, 0.24);
}

.sugerencias button {
  width: 100%;
  padding: 14px 18px;
  border: none;
  background: white;
  text-align: left;
  cursor: pointer;
  color: #0f172a;
  font-weight: 700;
}

.sugerencias button:hover {
  background: #eff6ff;
}

.sugerencias small {
  display: block;
  color: #64748b;
  margin-top: 2px;
}

.error-msg {
  margin-top: 14px;
  color: #fecaca;
  font-weight: 800;
}

.hero-summary {
  display: grid;
  gap: 14px;
}

.summary-card {
  display: grid;
  align-content: center;
  gap: 4px;
  padding: 18px;
  border-radius: 24px;
  background: rgba(255, 255, 255, 0.14);
  border: 1px solid rgba(255, 255, 255, 0.18);
  backdrop-filter: blur(14px);
}

.summary-card.main {
  background: rgba(96, 165, 250, 0.22);
}

.summary-card span {
  font-size: 2rem;
}

.summary-card strong {
  font-size: 1.05rem;
}

.summary-card small {
  color: #dbeafe;
}

.quick-actions {
  max-width: 1180px;
  margin: 0 auto 22px;
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 14px;
}

.action-card {
  border: 1px solid rgba(148, 163, 184, 0.22);
  border-radius: 22px;
  padding: 18px;
  background: rgba(255, 255, 255, 0.9);
  box-shadow: 0 14px 34px rgba(15, 23, 42, 0.08);
  display: flex;
  align-items: center;
  gap: 14px;
  text-align: left;
  cursor: pointer;
  transition: 0.25s;
}

.action-card span {
  width: 48px;
  height: 48px;
  border-radius: 16px;
  display: grid;
  place-items: center;
  background: #eff6ff;
  font-size: 1.8rem;
}

.action-card strong {
  display: block;
  color: #0f172a;
  margin-bottom: 3px;
}

.action-card small {
  color: #64748b;
  font-weight: 700;
}

.favorites-section,
.explore-section,
.result-layout {
  max-width: 1180px;
  margin: 0 auto 24px;
}

.section-header,
.result-heading {
  display: flex;
  justify-content: space-between;
  gap: 18px;
  align-items: end;
  margin-bottom: 16px;
}

.section-header h2,
.result-heading h2 {
  color: #0f172a;
  font-size: clamp(1.55rem, 3vw, 2.2rem);
}

.section-header p:last-child {
  color: #64748b;
  font-weight: 700;
  max-width: 420px;
}

.favorites-list {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.favorite-city {
  border: none;
  padding: 11px 18px;
  border-radius: 999px;
  background: #eff6ff;
  color: #2563eb;
  font-weight: 800;
  cursor: pointer;
  transition: 0.25s;
}

.favorite-city:hover {
  background: #2563eb;
  color: white;
}

.explore-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 18px;
}

.quick-card,
.section-card,
.map-card {
  background: rgba(255, 255, 255, 0.86);
  border: 1px solid rgba(148, 163, 184, 0.24);
  border-radius: 26px;
  padding: 24px;
  box-shadow: 0 18px 45px rgba(15, 23, 42, 0.08);
  backdrop-filter: blur(16px);
}

.quick-card h3,
.section-card h3,
.map-card h3 {
  color: #0f172a;
  margin-bottom: 16px;
}

.chips {
  display: flex;
  flex-wrap: wrap;
  gap: 11px;
}

.chips button {
  border: 1px solid #cbd5e1;
  border-radius: 999px;
  padding: 11px 18px;
  background: white;
  color: #1e293b;
  font-weight: 800;
  cursor: pointer;
  transition: 0.25s;
}

.chips button:hover {
  background: #2563eb;
  border-color: #2563eb;
  color: white;
}

.loading-box {
  max-width: 1180px;
  margin: 40px auto;
  text-align: center;
  color: #1e293b;
  font-weight: 800;
}

.spinner {
  width: 58px;
  height: 58px;
  margin: 0 auto 16px;
  border-radius: 50%;
  border: 6px solid #bfdbfe;
  border-top-color: #2563eb;
  animation: spin 1s linear infinite;
}

.result-grid {
  display: grid;
  grid-template-columns: 0.9fr 1.1fr;
  gap: 24px;
  align-items: start;
}

.result-left,
.result-right {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.back-btn.small {
  width: auto;
  white-space: nowrap;
}

.map-card {
  display: grid;
  gap: 16px;
}

.map-card p {
  color: #475569;
}

.map-frame {
  overflow: hidden;
  border-radius: 22px;
  border: 1px solid #bfdbfe;
  background: #dbeafe;
}

.map-frame iframe {
  display: block;
}

.coords {
  display: flex;
  justify-content: space-between;
  gap: 10px;
  font-size: 0.9rem;
}

.forecast-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(118px, 1fr));
  gap: 12px;
}

.alerts {
  display: grid;
  gap: 10px;
}

.alert {
  padding: 14px 16px;
  border-radius: 16px;
  font-weight: 800;
}

.success {
  background: #dcfce7;
  color: #166534;
}

.warning {
  background: #fef3c7;
  color: #92400e;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

@media (max-width: 980px) {
  .hero-panel,
  .explore-grid,
  .result-grid {
    grid-template-columns: 1fr;
  }

  .hero-summary,
  .quick-actions {
    grid-template-columns: repeat(2, 1fr);
  }

  .section-header,
  .result-heading {
    align-items: flex-start;
    flex-direction: column;
  }
}

@media (max-width: 620px) {
  .home-page {
    padding: 20px 14px 40px;
  }

  .hero-panel {
    padding: 24px;
  }

  .search-box {
    flex-direction: column;
    align-items: stretch;
  }

  .search-icon {
    display: none;
  }

  .quick-actions,
  .hero-summary {
    grid-template-columns: 1fr;
  }

  .coords {
    flex-direction: column;
  }

  .back-btn.small {
    width: 100%;
  }
}
</style>