<template>
  <div class="simulation-panel">
    <div class="sim-header">
      <span class="sim-title">SIMULATIONSMONITOR</span>
      <div class="sim-stats">
        <span class="stat">RUNDE <b>{{ currentRound }}</b></span>
        <span class="stat">AKTIONEN <b>{{ totalEvents }}</b></span>
        <span class="stat">Verstrichene Zeit <b>{{ elapsedTime }}</b></span>
      </div>
    </div>

    <div class="sim-content">
      <div class="events-panel">
        <div class="events-header">
          <span>EREIGNISSE GESAMT: {{ totalEvents }}</span>
        </div>
        <div class="events-list" ref="eventsContainer">
          <div v-if="events.length === 0" class="empty-events">
            Warten auf Agentenaktionen...
          </div>
          <div
            v-for="(event, idx) in events"
            :key="idx"
            class="event-item"
            :class="event.action_type?.toLowerCase()"
          >
            <span class="event-time">{{ event.sim_time || event.time }}</span>
            <span class="event-agent">{{ event.agent_name || event.agent_id }}</span>
            <span class="event-action">{{ event.action_type }}</span>
            <span v-if="event.target_agent" class="event-target">→ {{ event.target_agent }}</span>
            <span v-if="event.content" class="event-content">{{ event.content?.slice(0, 60) }}...</span>
            <span v-if="event.skipped" class="event-skipped">Aktion übersprungen</span>
          </div>
        </div>
      </div>

      <div class="side-panels">
        <!-- Trending -->
        <div class="side-panel">
          <div class="panel-title">TREND</div>
          <div class="trending-list">
            <div v-for="(topic, idx) in trendingTopics" :key="idx" class="trending-item">
              <span class="trend-rank">#{{ idx + 1 }}</span>
              <span class="trend-topic">{{ topic.name || topic }}</span>
              <span v-if="topic.count" class="trend-count">{{ topic.count }}</span>
            </div>
          </div>
        </div>

        <!-- Info-Platz -->
        <div class="side-panel">
          <div class="panel-title">Info-Platz</div>
          <div class="search-list">
            <div v-for="(search, idx) in recentSearches" :key="idx" class="search-item">
              <span class="search-label">Suchanfrage:</span>
              <span class="search-query">{{ search.query }}</span>
            </div>
          </div>
        </div>

        <!-- Themen-Community -->
        <div class="side-panel">
          <div class="panel-title">Themen-Community</div>
          <div class="community-list">
            <div v-for="(community, idx) in communities" :key="idx" class="community-item">
              <span class="community-name">{{ community.name || community }}</span>
            </div>
          </div>
        </div>

        <!-- Verfügbare Aktionen -->
        <div class="side-panel actions-panel">
          <div class="panel-title">Verfügbare Aktionen</div>
          <div class="actions-grid">
            <span v-for="action in availableActions" :key="action" class="action-badge" :class="action.toLowerCase()">{{ action }}</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Systemlogs -->
    <div class="system-logs">
      <div class="log-header">
        <span class="log-title">SYSTEM-DASHBOARD</span>
        <span class="log-id">{{ simulationId || 'KEIN_PROJEKT' }}</span>
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
import { ref, watch, nextTick, onMounted, onUnmounted, computed } from 'vue'
import { getSimulationStatus, getSimulationEvents } from '../api/simulation'

const props = defineProps({
  simulationId: String,
  graphData: Object,
  systemLogs: { type: Array, default: () => [] }
})

const emit = defineEmits(['go-back', 'next-step', 'add-log', 'update-status'])

const events = ref([])
const currentRound = ref(0)
const totalEvents = ref(0)
const elapsedTime = ref('00:00:00')
const trendingTopics = ref([])
const recentSearches = ref([])
const communities = ref([])
const eventsContainer = ref(null)
const logContent = ref(null)
let pollInterval = null

const availableActions = ['POST', 'LIKE', 'DISLIKE', 'REPOST', 'QUOTE', 'FOLLOW', 'MUTE', 'COMMENT', 'SEARCH', 'REFRESH', 'IDLE']

const fetchStatus = async () => {
  if (!props.simulationId) return
  try {
    const res = await getSimulationStatus(props.simulationId)
    if (res.success && res.data) {
      const d = res.data
      currentRound.value = d.current_round ?? currentRound.value
      totalEvents.value = d.total_events ?? totalEvents.value
      trendingTopics.value = d.trending_topics ?? trendingTopics.value
      communities.value = d.communities ?? communities.value
      if (d.status === 'completed') {
        clearInterval(pollInterval)
        emit('update-status', 'completed')
        emit('add-log', { time: new Date().toLocaleTimeString('de-DE'), msg: 'Simulation abgeschlossen' })
      }
    }
  } catch (e) { console.error(e) }
}

const fetchEvents = async () => {
  if (!props.simulationId) return
  try {
    const res = await getSimulationEvents(props.simulationId, events.value.length)
    if (res.success && res.data?.length) {
      events.value.push(...res.data)
      await nextTick()
      if (eventsContainer.value) eventsContainer.value.scrollTop = eventsContainer.value.scrollHeight
    }
  } catch (e) { console.error(e) }
}

onMounted(() => {
  fetchStatus()
  fetchEvents()
  pollInterval = setInterval(() => { fetchStatus(); fetchEvents() }, 3000)
})

onUnmounted(() => clearInterval(pollInterval))

watch(() => props.systemLogs, async () => {
  await nextTick()
  if (logContent.value) logContent.value.scrollTop = logContent.value.scrollHeight
}, { deep: true })
</script>
