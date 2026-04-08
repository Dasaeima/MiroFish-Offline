<template>
  <div class="history-section">
    <div class="history-header">
      <span class="header-icon">◆</span>
      <span class="header-title">Simulationsaufzeichnungen</span>
      <button class="refresh-btn" @click="loadHistory" :disabled="loading">
        <span :class="{ spinning: loading }">↻</span>
      </button>
    </div>

    <div v-if="loading" class="loading-state">
      <div class="spinner"></div>
      <span>Laden...</span>
    </div>

    <div v-else-if="projects.length === 0" class="empty-state">
      <span class="empty-icon">◈</span>
      <span>Keine Simulationsaufzeichnungen vorhanden</span>
    </div>

    <div v-else class="projects-grid">
      <div
        v-for="project in projects"
        :key="project.project_id"
        class="project-card"
        @click="openProject(project)"
      >
        <div class="project-header">
          <span class="project-icon">●</span>
          <span class="project-id">{{ project.project_id?.slice(0, 8) }}...</span>
          <span class="project-date">{{ formatDate(project.created_at) }}</span>
        </div>

        <div class="project-requirement">
          <span class="req-label">Simulationsanforderung</span>
          <p class="req-text">{{ project.simulation_requirement || 'Keine Anforderung' }}</p>
        </div>

        <div class="project-steps">
          <span class="step-badge" :class="{ done: project.graph_id }">Schritt 1</span>
          <span class="step-sep">›</span>
          <span class="step-badge" :class="{ done: project.simulation_id }">Schritt 2</span>
          <span class="step-sep">›</span>
          <span class="step-badge" :class="{ done: project.report_id }">Schritt 4</span>
        </div>

        <div class="project-actions">
          <button
            v-if="project.graph_id"
            class="action-link"
            @click.stop="resumeStep(project, 1)"
          >Graphkonstruktion</button>
          <button
            v-if="project.simulation_id"
            class="action-link"
            @click.stop="resumeStep(project, 2)"
          >Umgebungs-Setup</button>
          <button
            v-if="project.simulation_id"
            class="action-link"
            @click.stop="resumeStep(project, 3)"
          >Simulationswiedergabe</button>
          <button
            v-if="project.report_id"
            class="action-link"
            @click.stop="resumeStep(project, 4)"
          >Analysebericht</button>
        </div>

        <div class="project-files">
          <span class="files-label">Zugehörige Dateien</span>
          <div v-if="project.files && project.files.length > 0" class="files-list">
            <span
              v-for="file in project.files"
              :key="file"
              class="file-tag"
            >{{ file }}</span>
          </div>
          <span v-else class="no-files">Keine Dateien</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { getProjects } from '../api/projects'

const router = useRouter()
const projects = ref([])
const loading = ref(false)

const loadHistory = async () => {
  loading.value = true
  try {
    const res = await getProjects()
    if (res.success) {
      projects.value = res.data || []
    }
  } catch (e) {
    console.error('Fehler beim Laden des Verlaufs:', e)
  } finally {
    loading.value = false
  }
}

const formatDate = (dateStr) => {
  if (!dateStr) return ''
  return new Date(dateStr).toLocaleDateString('de-DE', { month: 'short', day: 'numeric', hour: '2-digit', minute: '2-digit' })
}

const openProject = (project) => {
  resumeStep(project, project.report_id ? 4 : project.simulation_id ? 3 : 1)
}

const resumeStep = (project, step) => {
  const query = {
    project_id: project.project_id,
    graph_id: project.graph_id,
    simulation_requirement: project.simulation_requirement,
  }
  if (step === 1) {
    router.push({ name: 'process', query })
  } else if (step === 2 || step === 3) {
    router.push({ name: 'Simulation', params: { simulationId: project.simulation_id }, query })
  } else if (step === 4) {
    router.push({ name: 'Report', params: { reportId: project.report_id, simulationId: project.simulation_id }, query })
  }
}

onMounted(loadHistory)
</script>
