<template>
  <main class="detalle-page tema-oscuro">

    <section class="detalle-hero">

      <RouterLink to="/" class="back-link">
        ← Volver al inicio
      </RouterLink>

      <div v-if="loading" class="estado estado-loading">
        <span class="spinner"></span>
        <p>Cargando detalle del clima...</p>
      </div>

      <div v-else-if="error" class="estado estado-error">
        <h2>⚠️ No se pudo cargar el detalle</h2>
        <p>{{ error }}</p>
        <RouterLink to="/" class="btn-volver">
          Volver al inicio
        </RouterLink>
      </div>

      <div v-else-if="ciudadActual && climaActual" class="detalle-contenido">

        <header class="detalle-header">
          <div>
            <p class="eyebrow">Detalle meteorológico</p>
            <h1>{{ ciudadActual.nombre }}</h1>
            <p class="pais">{{ ciudadActual.pais }}</p>
          </div>

          <div class="temperatura-principal">
            <span class="emoji">{{ obtenerEmoji(climaActual.codigo) }}</span>
            <strong>{{ climaActual.temperatura }}°C</strong>
            <small>{{ obtenerDescripcion(climaActual.codigo) }}</small>
          </div>
        </header>

        <section class="detalle-grid">
          <article class="info-card">
            <span>🌡️ Sensación térmica</span>
            <strong>{{ climaActual.sensacionTermica }}°C</strong>
          </article>

          <article class="info-card">
            <span>💧 Humedad</span>
            <strong>{{ climaActual.humedad }}%</strong>
          </article>

          <article class="info-card">
            <span>💨 Viento</span>
            <strong>{{ climaActual.viento }} km/h</strong>
          </article>

          <article class="info-card">
            <span>👁️ Visibilidad</span>
            <strong>{{ climaActual.visibilidad }} km</strong>
          </article>
        </section>

        <section v-if="alertas.length" class="alertas-section">
          <h2>Alertas meteorológicas</h2>

          <div class="alertas-lista">
            <article
              v-for="alerta in alertas"
              :key="alerta.tipo"
              class="alerta-card"
            >
              {{ alerta.mensaje }}
            </article>
          </div>
        </section>

        <section v-if="estadisticas" class="estadisticas-section">
          <h2>Estadísticas de la semana</h2>

          <div class="estadisticas-grid">
            <article class="stat-card">
              <span>Máxima</span>
              <strong>{{ estadisticas.tempMaxima }}°C</strong>
            </article>

            <article class="stat-card">
              <span>Mínima</span>
              <strong>{{ estadisticas.tempMinima }}°C</strong>
            </article>

            <article class="stat-card">
              <span>Promedio</span>
              <strong>{{ estadisticas.promedio }}°C</strong>
            </article>

            <article class="stat-card">
              <span>Días con lluvia</span>
              <strong>{{ estadisticas.diasLluvia }}</strong>
            </article>

            <article class="stat-card">
              <span>Días soleados</span>
              <strong>{{ estadisticas.diasSoleados }}</strong>
            </article>
          </div>
        </section>

        <section class="pronostico-section">
          <h2>Pronóstico próximos días</h2>

          <div class="pronostico-grid">
            <article
              v-for="dia in pronostico"
              :key="dia.fecha"
              class="forecast-card"
            >
              <span class="fecha">{{ formatearFecha(dia.fecha) }}</span>
              <span class="forecast-emoji">{{ obtenerEmoji(dia.codigo) }}</span>
              <strong>{{ dia.tempMax }}° / {{ dia.tempMin }}°</strong>
              <small>{{ obtenerDescripcion(dia.codigo) }}</small>
              <p>Precipitación: {{ dia.precipitacion }} mm</p>
            </article>
          </div>
        </section>

      </div>

      <div v-else class="estado estado-error">
        <h2>Ciudad no encontrada</h2>
        <p>Vuelve al inicio y selecciona una ciudad del listado.</p>
        <RouterLink to="/" class="btn-volver">
          Volver al inicio
        </RouterLink>
      </div>

    </section>

  </main>
</template>

<script>
import { mapGetters, mapActions } from 'vuex'
import weatherService from '../services/weatherService.js'

export default {
  name: 'DetalleView',

  props: {
    ciudad: {
      type: String,
      required: true
    }
  },

  computed: {
    ...mapGetters('weather', [
      'ciudadActual',
      'climaActual',
      'pronostico',
      'loading',
      'error'
    ]),

    todasLasCiudades() {
      return [
        ...weatherService.ciudadesChile,
        ...weatherService.ciudadesMundo
      ]
    },

    ciudadEncontrada() {
      const ciudadRuta = this.normalizarTexto(this.ciudad)

      return this.todasLasCiudades.find((item) => {
        return this.normalizarTexto(item.nombre) === ciudadRuta
      })
    },

    estadisticas() {
      return weatherService.calcularEstadisticas(this.pronostico)
    },

    alertas() {
      return weatherService.generarAlertas(this.climaActual, this.pronostico)
    }
  },

  async mounted() {
    if (this.ciudadEncontrada) {
      await this.buscarClima(this.ciudadEncontrada)
    }
  },

  methods: {
    ...mapActions('weather', ['buscarClima']),

    normalizarTexto(texto) {
      return String(texto || '')
        .normalize('NFD')
        .replace(/[\u0300-\u036f]/g, '')
        .trim()
        .toLowerCase()
    },

    obtenerEmoji(codigo) {
      return weatherService.obtenerEmoji(codigo)
    },

    obtenerDescripcion(codigo) {
      return weatherService.obtenerDescripcion(codigo)
    },

    formatearFecha(fecha) {
      return weatherService.formatearFecha(fecha)
    }
  }
}
</script>

<style scoped>
.detalle-page {
  min-height: 100vh;
  padding: 32px 20px;
  background:
    radial-gradient(circle at top left, rgba(96, 165, 250, 0.18), transparent 35%),
    radial-gradient(circle at bottom right, rgba(129, 140, 248, 0.16), transparent 40%),
    #020617;
  color: #e5e7eb;
}

.detalle-hero {
  width: min(1180px, 100%);
  margin: 0 auto;
}

.back-link {
  display: inline-flex;
  margin-bottom: 24px;
  color: #93c5fd;
  text-decoration: none;
  font-weight: 700;
}

.back-link:hover {
  color: #bfdbfe;
}

.estado {
  min-height: 420px;
  display: grid;
  place-items: center;
  text-align: center;
  border-radius: 28px;
  background: rgba(15, 23, 42, 0.78);
  border: 1px solid rgba(148, 163, 184, 0.18);
  padding: 32px;
}

.estado-loading {
  gap: 18px;
}

.spinner {
  width: 52px;
  height: 52px;
  border: 5px solid rgba(148, 163, 184, 0.3);
  border-top-color: #60a5fa;
  border-radius: 50%;
  animation: girar 0.9s linear infinite;
}

@keyframes girar {
  to {
    transform: rotate(360deg);
  }
}

.estado-error h2 {
  color: #fecaca;
  margin-bottom: 10px;
}

.btn-volver {
  display: inline-flex;
  margin-top: 18px;
  padding: 12px 18px;
  border-radius: 999px;
  background: linear-gradient(135deg, #2563eb, #7c3aed);
  color: white;
  text-decoration: none;
  font-weight: 800;
}

.detalle-contenido {
  display: grid;
  gap: 28px;
}

.detalle-header {
  display: flex;
  justify-content: space-between;
  gap: 24px;
  align-items: center;
  padding: 34px;
  border-radius: 32px;
  background: linear-gradient(135deg, rgba(15, 23, 42, 0.92), rgba(30, 41, 59, 0.82));
  border: 1px solid rgba(148, 163, 184, 0.22);
  box-shadow: 0 24px 80px rgba(0, 0, 0, 0.35);
}

.eyebrow {
  color: #93c5fd;
  font-weight: 800;
  text-transform: uppercase;
  letter-spacing: 0.14em;
  font-size: 0.78rem;
  margin-bottom: 8px;
}

.detalle-header h1 {
  font-size: clamp(2.2rem, 5vw, 4.5rem);
  margin: 0;
}

.pais {
  color: #cbd5e1;
  margin-top: 6px;
  font-size: 1.1rem;
}

.temperatura-principal {
  display: grid;
  justify-items: center;
  gap: 4px;
  min-width: 180px;
}

.emoji {
  font-size: 4rem;
}

.temperatura-principal strong {
  font-size: 3.5rem;
  color: #bfdbfe;
}

.temperatura-principal small {
  color: #cbd5e1;
  text-align: center;
}

.detalle-grid,
.estadisticas-grid,
.pronostico-grid {
  display: grid;
  gap: 16px;
}

.detalle-grid {
  grid-template-columns: repeat(4, 1fr);
}

.estadisticas-grid {
  grid-template-columns: repeat(5, 1fr);
}

.pronostico-grid {
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
}

.info-card,
.stat-card,
.forecast-card,
.alerta-card {
  padding: 20px;
  border-radius: 24px;
  background: rgba(15, 23, 42, 0.78);
  border: 1px solid rgba(148, 163, 184, 0.18);
}

.info-card span,
.stat-card span {
  display: block;
  color: #94a3b8;
  margin-bottom: 8px;
  font-weight: 700;
}

.info-card strong,
.stat-card strong {
  font-size: 1.65rem;
  color: #e0f2fe;
}

.alertas-section,
.estadisticas-section,
.pronostico-section {
  display: grid;
  gap: 16px;
}

.alertas-section h2,
.estadisticas-section h2,
.pronostico-section h2 {
  margin: 0;
  font-size: 1.6rem;
}

.alertas-lista {
  display: grid;
  gap: 12px;
}

.alerta-card {
  background: rgba(127, 29, 29, 0.38);
  border-color: rgba(248, 113, 113, 0.35);
  color: #fecaca;
  font-weight: 700;
}

.forecast-card {
  text-align: center;
  display: grid;
  gap: 8px;
}

.fecha {
  color: #93c5fd;
  font-weight: 800;
  text-transform: capitalize;
}

.forecast-emoji {
  font-size: 2.4rem;
}

.forecast-card strong {
  font-size: 1.35rem;
  color: #f8fafc;
}

.forecast-card small,
.forecast-card p {
  color: #cbd5e1;
  margin: 0;
}

@media (max-width: 900px) {
  .detalle-header {
    flex-direction: column;
    text-align: center;
  }

  .detalle-grid,
  .estadisticas-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 560px) {
  .detalle-page {
    padding: 20px 12px;
  }

  .detalle-header {
    padding: 24px;
  }

  .detalle-grid,
  .estadisticas-grid {
    grid-template-columns: 1fr;
  }

  .temperatura-principal strong {
    font-size: 2.8rem;
  }
}
</style>