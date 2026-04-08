<div align="center">

<img src="./static/image/mirofish-offline-banner.png" alt="MiroFish Offline" width="100%"/>

# MiroFish-Offline

**Vollständig lokaler Fork von [MiroFish](https://github.com/666ghj/MiroFish) — keine Cloud-APIs erforderlich. Deutsche Benutzeroberfläche.**

*Eine Multi-Agenten-Schwarmintelligenz-Engine, die öffentliche Meinung, Marktstimmung und soziale Dynamiken simuliert. Vollständig auf deiner eigenen Hardware.*

[![GitHub Stars](https://img.shields.io/github/stars/nikmcfly/MiroFish-Offline?style=flat-square&color=DAA520)](https://github.com/nikmcfly/MiroFish-Offline/stargazers)
[![GitHub Forks](https://img.shields.io/github/forks/nikmcfly/MiroFish-Offline?style=flat-square)](https://github.com/nikmcfly/MiroFish-Offline/network)
[![Docker](https://img.shields.io/badge/Docker-Build-2496ED?style=flat-square&logo=docker&logoColor=white)](https://hub.docker.com/)
[![License: AGPL-3.0](https://img.shields.io/badge/License-AGPL--3.0-blue?style=flat-square)](./LICENSE)

</div>

## Was ist das?

MiroFish ist eine Multi-Agenten-Simulationsmaschine: Lade ein beliebiges Dokument hoch (Pressemitteilung, Gesetzesentwurf, Finanzbericht) und es generiert Hunderte von KI-Agenten mit einzigartigen Persönlichkeiten, die die öffentliche Reaktion in sozialen Medien simulieren. Beiträge, Diskussionen, Meinungsveränderungen — Stunde für Stunde.

Das [originale MiroFish](https://github.com/666ghj/MiroFish) wurde für den chinesischen Markt entwickelt (chinesische Oberfläche, Zep Cloud für Wissensgraphen, DashScope API). Dieser Fork macht es **vollständig lokal und vollständig auf Deutsch**:

| Originales MiroFish | MiroFish-Offline |
|---|---|
| Chinesische Oberfläche | **Deutsche Oberfläche** (1.000+ übersetzte Texte) |
| Zep Cloud (Graph-Speicher) | **Neo4j Community Edition 5.15** |
| DashScope / OpenAI API (LLM) | **Ollama** (qwen2.5, llama3, etc.) |
| Zep Cloud Embeddings | **nomic-embed-text** via Ollama |
| Cloud-API-Keys erforderlich | **Keine Cloud-Abhängigkeiten** |

## Workflow

1. **Graphaufbau** — Extrahiert Entitäten (Personen, Unternehmen, Ereignisse) und Beziehungen aus deinem Dokument. Erstellt einen Wissensgraphen mit individuellem und gemeinschaftlichem Gedächtnis via Neo4j.
2. **Umgebungs-Setup** — Generiert Hunderte von Agenten-Personas, jede mit einzigartiger Persönlichkeit, Meinungsneigung, Reaktionsgeschwindigkeit, Einflussniveau und Gedächtnis vergangener Ereignisse.
3. **Simulation** — Agenten interagieren auf simulierten sozialen Plattformen: Posten, Antworten, Diskutieren, Meinungswandel. Das System verfolgt in Echtzeit die Stimmungsentwicklung, Themenausbreitung und Einflussdy­namiken.
4. **Bericht** — Ein ReportAgent analysiert die Simulationsumgebung, führt Interviews mit einer Fokusgruppe von Agenten, durchsucht den Wissensgraphen nach Belegen und erstellt eine strukturierte Analyse.
5. **Interaktion** — Chatte mit beliebigen Agenten aus der simulierten Welt. Frage sie, warum sie das gepostet haben, was sie gepostet haben. Volles Gedächtnis und Persönlichkeit bleiben erhalten.

## Screenshot

<div align="center">
<img src="./static/image/mirofish-offline-screenshot.jpg" alt="MiroFish Offline — Deutsche Oberfläche" width="100%"/>
</div>

## Schnellstart

### Voraussetzungen

- Docker & Docker Compose (empfohlen), **oder**
- Python 3.11+, Node.js 18+, Neo4j 5.15+, Ollama

### Option A: Docker (einfachste Methode)

```bash
git clone https://github.com/nikmcfly/MiroFish-Offline.git
cd MiroFish-Offline
cp .env.example .env

# Alle Dienste starten (Neo4j, Ollama, MiroFish)
docker compose up -d

# Benötigte Modelle in Ollama laden
docker exec mirofish-ollama ollama pull qwen2.5:32b
docker exec mirofish-ollama ollama pull nomic-embed-text
```

Öffne `http://localhost:3000` — das war's.

### Option B: Manuell

**1. Neo4j starten**

```bash
docker run -d --name neo4j \
  -p 7474:7474 -p 7687:7687 \
  -e NEO4J_AUTH=neo4j/mirofish \
  neo4j:5.15-community
```

**2. Ollama starten & Modelle laden**

```bash
ollama serve &
ollama pull qwen2.5:32b      # LLM (oder qwen2.5:14b für weniger VRAM)
ollama pull nomic-embed-text  # Embeddings (768d)
```

**3. Backend konfigurieren & starten**

```bash
cp .env.example .env
# .env bearbeiten, falls Neo4j/Ollama auf anderen Ports laufen

cd backend
pip install -r requirements.txt
python run.py
```

**4. Frontend starten**

```bash
cd frontend
npm install
npm run dev
```

Öffne `http://localhost:3000`.

## Konfiguration

Alle Einstellungen befinden sich in `.env` (Vorlage: `.env.example`):

```bash
# LLM — verweist auf lokales Ollama (OpenAI-kompatible API)
LLM_API_KEY=ollama
LLM_BASE_URL=http://localhost:11434/v1
LLM_MODEL_NAME=qwen2.5:32b

# Neo4j
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=mirofish

# Embeddings
EMBEDDING_MODEL=nomic-embed-text
EMBEDDING_BASE_URL=http://localhost:11434
```

Funktioniert mit jeder OpenAI-kompatiblen API — tausche Ollama gegen Claude, GPT oder einen anderen Anbieter aus, indem du `LLM_BASE_URL` und `LLM_API_KEY` änderst.

## Architektur

Dieser Fork führt eine saubere Abstraktionsschicht zwischen Anwendung und Graphdatenbank ein:

```
┌─────────────────────────────────────────┐
│              Flask API                   │
│  graph.py  simulation.py  report.py     │
└──────────────┬──────────────────────────┘
               │ app.extensions['neo4j_storage']
┌──────────────▼──────────────────────────┐
│           Dienst-Schicht                 │
│  EntityReader  GraphToolsService         │
│  GraphMemoryUpdater  ReportAgent         │
└──────────────┬──────────────────────────┘
               │ storage: GraphStorage
┌──────────────▼──────────────────────────┐
│         GraphStorage (abstrakt)          │
│              │                            │
│    ┌─────────▼─────────┐                │
│    │   Neo4jStorage     │                │
│    │  ┌───────────────┐ │                │
│    │  │ EmbeddingService│ ← Ollama       │
│    │  │ NERExtractor   │ ← Ollama LLM   │
│    │  │ SearchService  │ ← Hybrid-Suche │
│    │  └───────────────┘ │                │
│    └───────────────────┘                │
└─────────────────────────────────────────┘
               │
        ┌──────▼──────┐
        │  Neo4j CE   │
        │  5.15       │
        └─────────────┘
```

**Zentrale Design-Entscheidungen:**

- `GraphStorage` ist ein abstraktes Interface — tausche Neo4j gegen jede andere Graphdatenbank aus, indem du eine Klasse implementierst
- Dependency Injection via Flask `app.extensions` — keine globalen Singletons
- Hybridsuche: 0,7 × Vektorähnlichkeit + 0,3 × BM25-Schlüsselwortsuche
- Synchrone NER/RE-Extraktion via lokalem LLM (ersetzt Zeps asynchrone Episodes)
- Alle originalen Datenklassen und LLM-Tools (InsightForge, Panorama, Agent-Interviews) erhalten

## Hardwareanforderungen

| Komponente | Minimum | Empfohlen |
|---|---|---|
| RAM | 16 GB | 32 GB |
| VRAM (GPU) | 10 GB (14b-Modell) | 24 GB (32b-Modell) |
| Speicher | 20 GB | 50 GB |
| CPU | 4 Kerne | 8+ Kerne |

Der CPU-Only-Modus funktioniert, ist jedoch bei der LLM-Inferenz deutlich langsamer. Für schwächere Systeme empfiehlt sich `qwen2.5:14b` oder `qwen2.5:7b`.

## Anwendungsfälle

- **PR-Krisentest** — Simuliere die öffentliche Reaktion auf eine Pressemitteilung, bevor sie veröffentlicht wird
- **Handelssignal-Generierung** — Finanzielle Nachrichten einspeisen und simulierte Marktstimmung beobachten
- **Politikfolgenabschätzung** — Gesetzesentwürfe gegen simulierte öffentliche Reaktionen testen
- **Kreative Experimente** — Jemand hat einen klassischen chinesischen Roman mit fehlendem Ende eingespeist; die Agenten schrieben ein narrativ stimmiges Ende

## Lizenz

AGPL-3.0 — identisch mit dem originalen MiroFish-Projekt. Siehe [LICENSE](./LICENSE).

## Danksagungen & Quellenangabe

Dies ist ein modifizierter Fork von [MiroFish](https://github.com/666ghj/MiroFish) von [666ghj](https://github.com/666ghj), ursprünglich unterstützt durch die [Shanda Group](https://www.shanda.com/). Die Simulationsmaschine basiert auf [OASIS](https://github.com/camel-ai/oasis) vom CAMEL-AI-Team.

**Änderungen in diesem Fork:**
- Backend von Zep Cloud auf lokales Neo4j CE 5.15 + Ollama migriert
- Gesamte Oberfläche vom Chinesischen ins Deutsche übersetzt (20 Dateien, 1.000+ Texte)
- Alle Zep-Referenzen durch Neo4j ersetzt
- Umbenennung zu MiroFish Offline
