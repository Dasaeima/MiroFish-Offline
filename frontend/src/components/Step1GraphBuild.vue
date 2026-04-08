<template>
  <div class="workbench-panel">
    <div class="scroll-container">
      <!-- Schritt 01: Ontologie -->
      <div class="step-card" :class="{ 'active': currentPhase === 0, 'completed': currentPhase > 0 }">
        <div class="card-header">
          <div class="step-info">
            <span class="step-num">01</span>
            <span class="step-title">Ontologie-Generierung</span>
          </div>
          <div class="step-status">
            <span v-if="currentPhase > 0" class="badge success">Abgeschlossen</span>
            <span v-else-if="currentPhase === 0" class="badge processing">Wird generiert</span>
            <span v-else class="badge pending">Wartend</span>
          </div>
        </div>
        <div class="card-content">
          <p class="api-note">POST /api/graph/ontology/generate</p>
          <p class="description">
            LLM analysiert Dokumenteninhalt und Simulationsanforderungen, extrahiert Realitätssamen und generiert automatisch geeignete Ontologiestrukturen
          </p>
          <div v-if="currentPhase === 0 && ontologyProgress" class="progress-section">
            <div class="spinner-sm"></div>
            <span>{{ ontologyProgress.message || 'Dokumente werden analysiert...' }}</span>
          </div>
          <div v-if="selectedOntologyItem" class="ontology-detail-overlay">
            <div class="detail-header">
               <div class="detail-title-group">
                  <span class="detail-type-badge">{{ selectedOntologyItem.itemType === 'entity' ? 'ENTITÄT' : 'RELATION' }}</span>
                  <span class="detail-name">{{ selectedOntologyItem.name }}</span>
               </div>
               <button class="close-btn" @click="selectedOntologyItem = null">×</button>
            </div>
            <div class="detail-body">
               <div class="detail-desc">{{ selectedOntologyItem.description }}</div>
               <div class="detail-section" v-if="selectedOntologyItem.attributes?.length">
                  <span class="section-label">ATTRIBUTE</span>
                  <div class="attr-list">
                     <div v-for="attr in selectedOntologyItem.attributes" :key="attr.name" class="attr-item">
                        <span class="attr-name">{{ attr.name }}</span>
                        <span class="attr-type">({{ attr.type }})</span>
                        <span class="attr-desc">{{ attr.description }}</span>
                     </div>
                  </div>
               </div>
               <div class="detail-section" v-if="selectedOntologyItem.examples?.length">
                  <span class="section-label">BEISPIELE</span>
                  <div class="example-list">
                     <span v-for="ex in selectedOntologyItem.examples" :key="ex" class="example-tag">{{ ex }}</span>
                  </div>
               </div>
               <div class="detail-section" v-if="selectedOntologyItem.source_targets?.length">
                  <span class="section-label">VERBINDUNGEN</span>
                  <div class="conn-list">
                     <div v-for="(conn, idx) in selectedOntologyItem.source_targets" :key="idx" class="conn-item">
                        <span class="conn-node">{{ conn.source }}</span>
                        <span class="conn-arrow">→</span>
                        <span class="conn-node">{{ conn.target }}</span>
                     </div>
                  </div>
               </div>
            </div>
          </div>
          <div v-if="projectData?.ontology?.entity_types" class="tags-container" :class="{ 'dimmed': selectedOntologyItem }">
            <span class="tag-label">GENERIERTE ENTITÄTSTYPEN</span>
            <div class="tags-list">
              <span v-for="entity in projectData.ontology.entity_types" :key="entity.name" class="entity-tag clickable" @click="selectOntologyItem(entity, 'entity')">{{ entity.name }}</span>
            </div>
          </div>
          <div v-if="projectData?.ontology?.edge_types" class="tags-container" :class="{ 'dimmed': selectedOntologyItem }">
            <span class="tag-label">GENERIERTE RELATIONSTYPEN</span>
            <div class="tags-list">
              <span v-for="rel in projectData.ontology.edge_types" :key="rel.name" class="entity-tag clickable" @click="selectOntologyItem(rel, 'relation')">{{ rel.name }}</span>
            </div>
          </div>
        </div>
      </div>

      <!-- Schritt 02: Graph-Aufbau -->
      <div class="step-card" :class="{ 'active': currentPhase === 1, 'completed': currentPhase > 1 }">
        <div class="card-header">
          <div class="step-info">
            <span class="step-num">02</span>
            <span class="step-title">GraphRAG-Aufbau</span>
          </div>
          <div class="step-status">
            <span v-if="currentPhase > 1" class="badge success">Abgeschlossen</span>
            <span v-else-if="currentPhase === 1" class="badge processing">{{ buildProgress?.progress || 0 }}%</span>
            <span v-else class="badge pending">Wartend</span>
          </div>
        </div>
        <div class="card-content">
          <p class="api-note">POST /api/graph/build</p>
          <p class="description">
            Basierend auf der generierten Ontologie werden Dokumente automatisch aufgeteilt und Neo4j aufgerufen, um Wissensgraphen aufzubauen, Entitäten und Beziehungen zu extrahieren und zeitliche Erinnerungen sowie Community-Zusammenfassungen zu erstellen
          </p>
          <div class="stats-grid">
            <div class="stat-card">
              <span class="stat-value">{{ graphStats.nodes }}</span>
              <span class="stat-label">Entitätsknoten</span>
            </div>
            <div class="stat-card">
              <span class="stat-value">{{ graphStats.edges }}</span>
              <span class="stat-label">Relationskanten</span>
            </div>
            <div class="stat-card">
              <span class="stat-value">{{ graphStats.types }}</span>
              <span class="stat-label">SCHEMA-Typen</span>
            </div>
          </div>
        </div>
      </div>

      <!-- Schritt 03: Abgeschlossen -->
      <div class="step-card" :class="{ 'active': currentPhase === 2, 'completed': currentPhase >= 2 }">
        <div class="card-header">
          <div class="step-info">
            <span class="step-num">03</span>
            <span class="step-title">Aufbau abgeschlossen</span>
          </div>
          <div class="step-status">
            <span v-if="currentPhase >= 2" class="badge accent">In Bearbeitung</span>
          </div>
        </div>
        <div class="card-content">
          <p class="api-note">POST /api/simulation/create</p>
          <p class="description">Graphaufbau ist abgeschlossen. Bitte fahre mit dem nächsten Schritt fort, um die Simulationsumgebung einzurichten</p>
          <button class="action-btn" :disabled="currentPhase < 2 || creatingSimulation" @click="handleEnterEnvSetup">
            <span v-if="creatingSimulation" class="spinner-sm"></span>
            {{ creatingSimulation ? 'Wird erstellt...' : 'Umgebungs-Setup betreten ➝' }}
          </button>
        </div>
      </div>
    </div>

    <!-- Systemlogs -->
    <div class="system-logs">
      <div class="log-header">
        <span class="log-title">SYSTEM-DASHBOARD</span>
        <span class="log-id">{{ projectData?.project_id || 'KEIN_PROJEKT' }}</span>
      </div>
      <div class="log-content" ref="logContent">
        <div class="log-line" v-for="(log, idx) in systemLogs" :key="idx">
          <span class="log-time">{{ log.time }}</span>
          <span class="log-msg">{{ log.msg }}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, ref, watch, nextTick } from 'vue'
import { useRouter } from 'vue-router'
import { createSimulation } from '../api/simulation'

const router = useRouter()

const props = defineProps({
  currentPhase: { type: Number, default: 0 },
  projectData: Object,
  ontologyProgress: Object,
  buildProgress: Object,
  graphData: Object,
  systemLogs: { type: Array, default: () => [] }
})

defineEmits(['next-step'])

const selectedOntologyItem = ref(null)
const logContent = ref(null)
const creatingSimulation = ref(false)

const handleEnterEnvSetup = async () => {
  if (!props.projectData?.project_id || !props.projectData?.graph_id) {
    console.error('Projekt- oder Graphinformationen fehlen')
    return
  }
  creatingSimulation.value = true
  try {
    const res = await createSimulation({
      project_id: props.projectData.project_id,
      graph_id: props.projectData.graph_id,
      enable_twitter: true,
      enable_reddit: true
    })
    if (res.success && res.data?.simulation_id) {
      router.push({ name: 'Simulation', params: { simulationId: res.data.simulation_id } })
    } else {
      console.error('Simulation konnte nicht erstellt werden:', res.error)
      alert('Fehler beim Erstellen der Simulation: ' + (res.error || 'Unbekannter Fehler'))
    }
  } catch (err) {
    console.error('Ausnahme beim Erstellen der Simulation:', err)
    alert('Ausnahme: ' + err.message)
  } finally {
    creatingSimulation.value = false
  }
}

const selectOntologyItem = (item, type) => {
  selectedOntologyItem.value = { ...item, itemType: type }
}

const graphStats = computed(() => ({
  nodes: props.graphData?.node_count ?? props.graphData?.nodes?.length ?? 0,
  edges: props.graphData?.edge_count ?? props.graphData?.edges?.length ?? 0,
  types: props.projectData?.ontology?.entity_types?.length ?? 0
}))

watch(() => props.systemLogs, async () => {
  await nextTick()
  if (logContent.value) logContent.value.scrollTop = logContent.value.scrollHeight
}, { deep: true })
</script>
