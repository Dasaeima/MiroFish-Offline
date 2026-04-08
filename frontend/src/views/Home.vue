<template>
  <div class="home-container">
    <!-- Top Navigation Bar -->
    <nav class="navbar" :style="s.navbar">
      <div class="nav-brand" :style="s.navBrand">MIROFISH OFFLINE</div>
      <div class="nav-links" :style="s.navLinks">
        <a href="https://github.com/nikmcfly/MiroFish-Offline" target="_blank" class="github-link" :style="s.githubLink">
          Unser Github besuchen <span>↗</span>
        </a>
      </div>
    </nav>

    <div class="main-content" :style="s.mainContent">
      <!-- Hero Section -->
      <section class="hero-section" :style="s.heroSection">
        <div class="hero-left" :style="s.heroLeft">
          <div class="tag-row" :style="s.tagRow">
            <span class="orange-tag" :style="s.orangeTag">Offline Multi-Agenten-Simulationsmotor</span>
            <span class="version-text" :style="s.versionText">/ v0.1-Vorschau</span>
          </div>

          <h1 class="main-title" :style="s.mainTitle">
            Beliebiges Dokument hochladen<br>
            <span class="gradient-text" :style="s.gradientText">Vorhersagen was als Nächstes passiert</span>
          </h1>

          <div class="hero-desc" :style="s.heroDesc">
            <p :style="s.heroDescP">
              Aus einem einzigen Dokument <span :style="s.highlightBold">MiroFish Offline</span> extrahiert Realitätssamen und baut eine Parallelwelt aus <span :style="s.highlightOrange">autonomen KI-Agenten</span> — die vollständig auf deinem Rechner läuft. Variablen injizieren, emergentes Verhalten beobachten und <span :style="s.highlightCode">"lokale Optima"</span> in komplexen sozialen Dynamiken finden.
            </p>
            <p class="slogan-text" :style="s.sloganText">
              Deine Daten verlassen nie deinen Rechner. Die Zukunft wird lokal simuliert<span :style="s.blinkingCursor">_</span>
            </p>
          </div>

          <div class="decoration-square" :style="s.decorationSquare"></div>
        </div>

        <div class="hero-right" :style="s.heroRight">
          <div class="logo-container" :style="s.logoContainer">
            <img src="../assets/logo/MiroFish_logo_left.jpeg" alt="MiroFish Logo" :style="s.heroLogo" />
          </div>
          <button :style="s.scrollDownBtn" @click="scrollToBottom">↓</button>
        </div>
      </section>

      <!-- Dashboard: Zweispalten-Layout -->
      <section class="dashboard-section" :style="s.dashboardSection">
        <!-- Linke Spalte: Status & Schritte -->
        <div class="left-panel" :style="s.leftPanel">
          <div class="panel-header" :style="s.panelHeader">
            <span :style="s.statusDot">■</span> Systemstatus
          </div>

          <h2 class="section-title" :style="s.sectionTitle">Bereit</h2>
          <p class="section-desc" :style="s.sectionDesc">
            Lokale Vorhersage-Engine auf Standby. Lade unstrukturierte Daten hoch, um eine Simulation zu starten.
          </p>

          <div class="metrics-row" :style="s.metricsRow">
            <div class="metric-card" :style="s.metricCard">
              <div class="metric-value" :style="s.metricValue">Kostenlos</div>
              <div class="metric-label" :style="s.metricLabel">Läuft auf deiner Hardware</div>
            </div>
            <div class="metric-card" :style="s.metricCard">
              <div class="metric-value" :style="s.metricValue">Privat</div>
              <div class="metric-label" :style="s.metricLabel">100% offline, keine Cloud</div>
            </div>
          </div>

          <div class="steps-container" :style="s.stepsContainer">
            <div class="steps-header" :style="s.stepsHeader">
               <span :style="s.diamondIcon">◇</span> Arbeitsablauf
            </div>
            <div :style="s.workflowList">
              <div v-for="(step, i) in steps" :key="i" :style="s.workflowItem">
                <span :style="s.stepNum">{{ step.num }}</span>
                <div :style="s.stepInfo">
                  <div :style="s.stepTitle">{{ step.title }}</div>
                  <div :style="s.stepDesc">{{ step.desc }}</div>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- Rechte Spalte: Interaktive Konsole -->
        <div class="right-panel" :style="s.rightPanel">
          <div class="console-box" :style="s.consoleBox">
            <div :style="s.consoleSection">
              <div class="console-header" :style="s.consoleHeader">
                <span>01 / Realitätssamen</span>
                <span>Unterstützt: PDF, MD, TXT</span>
              </div>
              <div
                :style="s.uploadZone"
                @dragover.prevent="handleDragOver"
                @dragleave.prevent="handleDragLeave"
                @drop.prevent="handleDrop"
                @click="triggerFileInput"
              >
                <input ref="fileInput" type="file" multiple accept=".pdf,.md,.txt" @change="handleFileSelect" style="display: none" :disabled="loading" />
                <div v-if="files.length === 0" :style="s.uploadPlaceholder">
                  <div :style="s.uploadIcon">↑</div>
                  <div :style="s.uploadTitle">Dateien hierher ziehen & ablegen</div>
                  <div :style="s.uploadHint">oder klicken zum Durchsuchen</div>
                </div>
                <div v-else :style="s.fileList">
                  <div v-for="(file, index) in files" :key="index" :style="s.fileItem">
                    <span>📄</span>
                    <span :style="s.fileName">{{ file.name }}</span>
                    <button @click.stop="removeFile(index)" :style="s.removeBtn">×</button>
                  </div>
                </div>
              </div>
            </div>

            <div :style="s.consoleDivider"><span :style="s.consoleDividerText">Parameter</span></div>

            <div :style="s.consoleSection">
              <div class="console-header" :style="s.consoleHeader">
                <span>>_ 02 / Simulations-Prompt</span>
              </div>
              <div :style="s.inputWrapper">
                <textarea v-model="formData.simulationRequirement" :style="s.codeInput" placeholder="// Beschreibe dein Simulations- oder Vorhersageziel in natürlicher Sprache" rows="6" :disabled="loading"></textarea>
                <div :style="s.modelBadge">Motor: Ollama + Neo4j (lokal)</div>
              </div>
            </div>

            <div :style="s.btnSection">
              <button :style="s.startEngineBtn" @click="startSimulation" :disabled="!canSubmit || loading">
                <span v-if="!loading">Motor starten</span>
                <span v-else>Initialisierung...</span>
                <span>→</span>
              </button>
            </div>
          </div>
        </div>
      </section>

      <HistoryDatabase />
    </div>
  </div>
</template>

<script setup>
import { ref, computed, reactive } from 'vue'
import { useRouter } from 'vue-router'
import HistoryDatabase from '../components/HistoryDatabase.vue'

const mono = 'JetBrains Mono, monospace'
const sans = 'Space Grotesk, Noto Sans SC, system-ui, sans-serif'

const s = reactive({
  navbar: { height: '60px', background: '#000', color: '#fff', display: 'flex', justifyContent: 'space-between', alignItems: 'center', padding: '0 40px' },
  navBrand: { fontFamily: mono, fontWeight: '800', letterSpacing: '1px', fontSize: '1.2rem' },
  navLinks: { display: 'flex', alignItems: 'center' },
  githubLink: { color: '#fff', textDecoration: 'none', fontFamily: mono, fontSize: '0.9rem', fontWeight: '500', display: 'flex', alignItems: 'center', gap: '8px' },
  mainContent: { maxWidth: '1400px', margin: '0 auto', padding: '60px 40px' },
  heroSection: { display: 'flex', justifyContent: 'space-between', marginBottom: '80px', position: 'relative' },
  heroLeft: { flex: '1', paddingRight: '60px' },
  tagRow: { display: 'flex', alignItems: 'center', gap: '15px', marginBottom: '25px', fontFamily: mono, fontSize: '0.8rem' },
  orangeTag: { background: '#FF4500', color: '#fff', padding: '4px 10px', fontWeight: '700', letterSpacing: '1px', fontSize: '0.75rem' },
  versionText: { color: '#999', fontWeight: '500', letterSpacing: '0.5px' },
  mainTitle: { fontSize: '4.5rem', lineHeight: '1.2', fontWeight: '500', margin: '0 0 40px 0', letterSpacing: '-2px', color: '#000' },
  gradientText: { background: 'linear-gradient(90deg, #000 0%, #444 100%)', WebkitBackgroundClip: 'text', WebkitTextFillColor: 'transparent', display: 'inline-block' },
  heroDesc: { fontSize: '1.05rem', lineHeight: '1.8', color: '#666', maxWidth: '640px', marginBottom: '50px', fontWeight: '400', textAlign: 'justify' },
  heroDescP: { marginBottom: '1.5rem' },
  highlightBold: { color: '#000', fontWeight: '700' },
  highlightOrange: { color: '#FF4500', fontWeight: '700', fontFamily: mono },
  highlightCode: { background: 'rgba(0,0,0,0.05)', padding: '2px 6px', borderRadius: '2px', fontFamily: mono, fontSize: '0.9em', color: '#000', fontWeight: '600' },
  sloganText: { fontSize: '1.2rem', fontWeight: '520', color: '#000', letterSpacing: '1px', borderLeft: '3px solid #FF4500', paddingLeft: '15px', marginTop: '20px' },
  blinkingCursor: { color: '#FF4500', fontWeight: '700' },
  decorationSquare: { width: '16px', height: '16px', background: '#FF4500' },
  heroRight: { flex: '0.8', display: 'flex', flexDirection: 'column', justifyContent: 'space-between', alignItems: 'flex-end' },
  logoContainer: { width: '100%', display: 'flex', justifyContent: 'flex-end', paddingRight: '40px' },
  heroLogo: { maxWidth: '500px', width: '100%' },
  scrollDownBtn: { width: '40px', height: '40px', border: '1px solid #E5E5E5', background: 'transparent', display: 'flex', alignItems: 'center', justifyContent: 'center', cursor: 'pointer', color: '#FF4500', fontSize: '1.2rem' },
  dashboardSection: { display: 'flex', gap: '60px', borderTop: '1px solid #E5E5E5', paddingTop: '60px', alignItems: 'flex-start' },
  leftPanel: { flex: '0.8', display: 'flex', flexDirection: 'column' },
  panelHeader: { fontFamily: mono, fontSize: '0.8rem', color: '#999', letterSpacing: '1px', marginBottom: '20px', display: 'flex', alignItems: 'center', gap: '8px' },
  statusDot: { color: '#00FF00' },
  sectionTitle: { fontSize: '2.5rem', fontWeight: '400', margin: '0 0 15px 0', letterSpacing: '-1px' },
  sectionDesc: { fontSize: '0.95rem', color: '#666', lineHeight: '1.7', marginBottom: '40px' },
  metricsRow: { display: 'flex', gap: '20px', marginBottom: '40px' },
  metricCard: { flex: '1', padding: '20px', border: '1px solid #E5E5E5', background: '#FAFAFA' },
  metricValue: { fontSize: '1.8rem', fontWeight: '700', marginBottom: '5px' },
  metricLabel: { fontSize: '0.8rem', color: '#999', fontFamily: mono },
  stepsContainer: { borderTop: '1px solid #E5E5E5', paddingTop: '30px' },
  stepsHeader: { fontFamily: mono, fontSize: '0.8rem', color: '#999', letterSpacing: '1px', marginBottom: '20px', display: 'flex', alignItems: 'center', gap: '8px' },
  diamondIcon: { color: '#FF4500' },
  workflowList: { display: 'flex', flexDirection: 'column', gap: '15px' },
  workflowItem: { display: 'flex', alignItems: 'flex-start', gap: '15px' },
  stepNum: { fontFamily: mono, fontSize: '0.8rem', color: '#999', minWidth: '30px', paddingTop: '2px' },
  stepInfo: { flex: '1' },
  stepTitle: { fontSize: '0.95rem', fontWeight: '600', marginBottom: '3px' },
  stepDesc: { fontSize: '0.85rem', color: '#999', fontFamily: mono },
  rightPanel: { flex: '1.2' },
  consoleBox: { border: '1px solid #E5E5E5', background: '#FAFAFA' },
  consoleSection: { padding: '25px' },
  consoleHeader: { fontFamily: mono, fontSize: '0.8rem', color: '#999', letterSpacing: '1px', marginBottom: '15px', display: 'flex', justifyContent: 'space-between' },
  uploadZone: { border: '1px dashed #CCC', padding: '30px', cursor: 'pointer', minHeight: '120px', display: 'flex', alignItems: 'center', justifyContent: 'center', transition: 'border-color 0.2s' },
  uploadPlaceholder: { textAlign: 'center' },
  uploadIcon: { fontSize: '2rem', color: '#CCC', marginBottom: '10px' },
  uploadTitle: { fontSize: '0.9rem', color: '#666', marginBottom: '5px' },
  uploadHint: { fontSize: '0.8rem', color: '#999', fontFamily: mono },
  fileList: { width: '100%', display: 'flex', flexDirection: 'column', gap: '8px' },
  fileItem: { display: 'flex', alignItems: 'center', gap: '10px', padding: '8px', background: '#fff', border: '1px solid #E5E5E5' },
  fileName: { flex: '1', fontSize: '0.85rem', fontFamily: mono, overflow: 'hidden', textOverflow: 'ellipsis', whiteSpace: 'nowrap' },
  removeBtn: { background: 'none', border: 'none', cursor: 'pointer', color: '#999', fontSize: '1.2rem', padding: '0 5px', lineHeight: '1' },
  consoleDivider: { borderTop: '1px solid #E5E5E5', margin: '0', padding: '10px 25px', background: '#F5F5F5' },
  consoleDividerText: { fontFamily: mono, fontSize: '0.75rem', color: '#999', letterSpacing: '1px' },
  inputWrapper: { position: 'relative' },
  codeInput: { width: '100%', background: '#fff', border: '1px solid #E5E5E5', padding: '15px', fontFamily: mono, fontSize: '0.85rem', color: '#333', resize: 'vertical', outline: 'none', lineHeight: '1.6' },
  modelBadge: { position: 'absolute', bottom: '10px', right: '10px', fontFamily: mono, fontSize: '0.7rem', color: '#999', background: '#F5F5F5', padding: '3px 8px' },
  btnSection: { padding: '20px 25px', borderTop: '1px solid #E5E5E5' },
  startEngineBtn: { width: '100%', padding: '15px 30px', background: '#000', color: '#fff', border: 'none', cursor: 'pointer', fontFamily: mono, fontSize: '0.9rem', fontWeight: '700', letterSpacing: '1px', display: 'flex', justifyContent: 'space-between', alignItems: 'center', transition: 'background 0.2s' },
})

const router = useRouter()
const fileInput = ref(null)
const files = ref([])
const loading = ref(false)
const formData = reactive({ simulationRequirement: '' })

const steps = [
  { num: '01', title: 'Wissensgraph', desc: 'Dokumentenverarbeitung → Ontologie-Extraktion → Neo4j Wissensgraph' },
  { num: '02', title: 'Umgebungs-Setup', desc: 'Agenten-Persona-Erzeugung + Simulationswelt-Konfiguration' },
  { num: '03', title: 'Simulationslauf', desc: 'Multi-Agenten-Sozialdynamik-Simulation' },
  { num: '04', title: 'Analysebericht', desc: 'KI-generierter Analysebericht' },
  { num: '05', title: 'Tiefe Interaktion', desc: 'Chat mit Berichts-Agent, Interview mit virtuellen Personen' },
]

const canSubmit = computed(() => files.value.length > 0 && formData.simulationRequirement.trim())

function scrollToBottom() {
  window.scrollTo({ top: document.body.scrollHeight, behavior: 'smooth' })
}

function triggerFileInput() {
  if (!loading.value) fileInput.value?.click()
}

function handleDragOver(e) {
  e.currentTarget.style.borderColor = '#FF4500'
}

function handleDragLeave(e) {
  e.currentTarget.style.borderColor = '#CCC'
}

function handleDrop(e) {
  e.currentTarget.style.borderColor = '#CCC'
  const droppedFiles = Array.from(e.dataTransfer.files).filter(f =>
    ['.pdf', '.md', '.txt'].some(ext => f.name.toLowerCase().endsWith(ext))
  )
  files.value = [...files.value, ...droppedFiles]
}

function handleFileSelect(e) {
  const selected = Array.from(e.target.files)
  files.value = [...files.value, ...selected]
  e.target.value = ''
}

function removeFile(index) {
  files.value.splice(index, 1)
}

async function startSimulation() {
  if (!canSubmit.value || loading.value) return
  loading.value = true
  try {
    const formDataObj = new FormData()
    files.value.forEach(file => formDataObj.append('files', file))
    formDataObj.append('simulation_requirement', formData.simulationRequirement)

    const response = await fetch('/api/projects/create', {
      method: 'POST',
      body: formDataObj,
    })

    if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`)
    const result = await response.json()

    router.push({
      name: 'process',
      query: {
        project_id: result.project_id,
        graph_id: result.graph_id,
        simulation_requirement: formData.simulationRequirement,
      }
    })
  } catch (error) {
    console.error('Fehler beim Starten der Simulation:', error)
    alert(`Fehler: ${error.message}`)
  } finally {
    loading.value = false
  }
}
</script>
