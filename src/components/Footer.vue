<template>
  <footer class="app-footer">
    <div class="footer-container">
      <section class="footer-brand">
        <div class="footer-logo">
          <span>🌤️</span>
          <div>
            <h2>ClimaApp</h2>
            <p>Explora. Compara. Planifica.</p>
          </div>
        </div>

        <p class="footer-description">
          Aplicación de clima desarrollada con Vue 3, datos reales de Open-Meteo,
          estadísticas semanales y alertas meteorológicas.
        </p>
      </section>

      <section class="footer-column">
        <h3>Funcionalidades</h3>
        <ul>
          <li>Clima actual</li>
          <li>Pronóstico semanal</li>
          <li>Alertas meteorológicas</li>
          <li>Ciudades favoritas</li>
        </ul>
      </section>

      <section class="footer-column">
        <h3>Tecnologías</h3>
        <ul>
          <li>Vue 3</li>
          <li>Vuex</li>
          <li>Vue Router</li>
          <li>Vite</li>
          <li>Open-Meteo API</li>
        </ul>
      </section>
    </div>

    <div class="footer-bottom">
      <p>© 2026 Paula Pérez Valenzuela. Todos los derechos reservados</p>
      <p>
          Desarrollado con Vue 3 + Open-Meteo API</p>
    </div>
  </footer>
</template>

<script>
export default {
  name: 'AppFooter'
}
</script>

<style scoped>
.app-footer {
  background:
    linear-gradient(135deg, rgba(15, 23, 42, 0.96), rgba(30, 64, 175, 0.9));
  color: #e5e7eb;
  border-top: 1px solid rgba(191, 219, 254, 0.18);
  padding: 42px 24px 22px;
  overflow: hidden;
}

.footer-container {
  max-width: 1180px;
  margin: 0 auto;
  display: grid;
  grid-template-columns: 1.4fr 0.8fr 0.8fr;
  gap: 32px;
}

.footer-logo {
  display: flex;
  align-items: center;
  gap: 14px;
  margin-bottom: 16px;
}

.footer-logo span {
  width: 48px;
  height: 48px;
  flex: 0 0 48px;
  display: grid;
  place-items: center;
  border-radius: 18px;
  font-size: 26px;
  background: rgba(255, 255, 255, 0.14);
  box-shadow: inset 0 0 0 1px rgba(255, 255, 255, 0.14);
}

.footer-logo h2 {
  margin: 0;
  font-size: 1.45rem;
  font-weight: 900;
  color: white;
}

.footer-logo p {
  margin: 3px 0 0;
  color: #bfdbfe;
  font-weight: 800;
  font-size: 0.85rem;
}

.footer-description {
  max-width: 520px;
  color: #cbd5e1;
  line-height: 1.6;
  font-weight: 600;
}

.footer-column h3 {
  color: white;
  font-size: 1rem;
  margin-bottom: 14px;
}

.footer-column ul {
  list-style: none;
  padding: 0;
  margin: 0;
  display: grid;
  gap: 9px;
}

.footer-column li {
  color: #cbd5e1;
  font-weight: 700;
  font-size: 0.92rem;
}

.footer-column li::before {
  content: '•';
  color: #60a5fa;
  margin-right: 8px;
}

.footer-bottom {
  max-width: 1180px;
  margin: 28px auto 0;
  padding-top: 18px;
  border-top: 1px solid rgba(191, 219, 254, 0.16);
  color: #bfdbfe;
  font-size: 0.86rem;
  font-weight: 700;
  line-height: 1.6;
}

@media (max-width: 800px) {
  .app-footer {
    padding: 34px 18px 20px;
  }

  .footer-container {
    grid-template-columns: 1fr;
    gap: 26px;
  }

  .footer-description {
    max-width: 100%;
  }
}

@media (max-width: 480px) {
  .app-footer {
    padding: 32px 16px 20px;
  }

  .footer-logo {
    align-items: flex-start;
  }

  .footer-logo span {
    width: 42px;
    height: 42px;
    flex-basis: 42px;
    border-radius: 15px;
    font-size: 23px;
  }

  .footer-logo h2 {
    font-size: 1.25rem;
  }

  .footer-logo p,
  .footer-description,
  .footer-column li,
  .footer-bottom {
    font-size: 0.84rem;
  }

  .footer-bottom {
    text-align: left;
  }
}
</style>