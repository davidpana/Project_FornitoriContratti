<template>
  <header class="header">
    <div class="inner">
      <div class="brand">
        <div class="logo-wrap">
          <img src="../assets/logo.png" alt="ContractNotary" class="logo" />
        </div>
        <div class="titles">
          <h1 class="title">ContractNotary System</h1>
          <p class="subtitle">Notarizzazione sicura e tracciabilità dei contratti</p>
        </div>
      </div>

      <nav class="nav">
        <a href="#upload" class="nav-link">Carica PDF</a>
        <a href="#verifica" class="nav-link">Verifica</a>
        <span class="badge">v1.0</span>
      </nav>
    </div>
  </header>
</template>

<script setup>
// Header puramente presentazionale
</script>

<style scoped>
.header {
  background: linear-gradient(135deg,#0f172a 0%, #1e293b 50%, #4f46e5 100%);
  color: #fff;
  box-shadow: 0 8px 30px rgba(2,6,23,0.4);
}
.inner {
  max-width: 1100px;
  margin: 0 auto;
  padding: 22px 24px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
}
.brand { display: flex; align-items: center; gap: 20px; }
.logo-wrap {
  width: 200px;
  height: 95px;
  border-radius: 12px;
  background: #ffffff;
  border: 1px solid rgba(148, 163, 184, 0.65);
  box-shadow: 0 10px 28px rgba(15, 23, 42, 0.45), 0 0 0 3px rgba(255, 255, 255, 0.18);
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  flex: 0 0 200px;
}
.logo {
  width: 100%;
  height: 100%;
  border-radius: 0;
  object-fit: contain;
  background: #ffffff;
  transform: translateZ(0);
  transition: transform .25s ease;
}
.logo-wrap:hover .logo { transform: scale(1.03); }
.title {
  margin: 0; font-size: 1.6rem; font-weight: 700;
  background: linear-gradient(90deg,#fff 0%, #f3f4f6 100%);
  -webkit-background-clip: text; background-clip: text; color: transparent;
  letter-spacing: 0.2px;
}
.subtitle { margin: 2px 0 0; font-size: 0.92rem; opacity: 0.85; }
.nav { display: flex; align-items: center; gap: 14px; }
.nav-link { color: rgba(255,255,255,0.9); text-decoration: none; font-weight: 600; padding: 6px 10px; border-radius: 8px; transition: background .18s, transform .12s; }
.nav-link:hover { background: rgba(255,255,255,0.06); transform: translateY(-2px); }
.badge { background: rgba(255,255,255,0.08); padding: 6px 8px; border-radius: 999px; font-size: 0.78rem; font-weight: 600; color: #fff; }

@media (max-width: 700px) {
  .inner { flex-direction: column; align-items: flex-start; gap: 12px; }
  .brand { flex-direction: column; align-items: flex-start; }
  .logo-wrap { width: 180px; height: 86px; flex-basis: auto; }
  .nav { width: 100%; justify-content: flex-start; gap: 10px; }
  .title { font-size: 1.25rem; }
}
</style>
