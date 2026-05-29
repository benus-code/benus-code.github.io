# 🤖 AI-Powered Automation for an Educational Consulting Agency
### n8n · Ollama (LLaMA 3 + BGE) · RAG · Supabase · Telegram · Docker

> **EN** — A self-hosted, AI-driven automation system that handles incoming requests through Telegram, understands them with a **local LLM**, and performs targeted actions: answering FAQs, registering students into Google Sheets, and booking appointments in Google Calendar — all grounded in a **RAG (Retrieval-Augmented Generation)** knowledge base running on a personal VPS.
>
> **FR** — Système d'automatisation auto-hébergé piloté par IA qui traite les demandes entrantes via Telegram, les comprend grâce à un **LLM local**, et exécute des actions ciblées : réponses aux FAQ, enregistrement d'étudiants dans Google Sheets, et prise de rendez-vous dans Google Calendar — le tout ancré sur une base de connaissances **RAG**, hébergée sur un VPS personnel.

> 🎓 This project was developed and documented as a **Master's practice report at TUSUR** (Tomsk State University of Control Systems and Radioelectronics), Dept. EMIS — group 544-M.

---

## 🇬🇧 English

### Overview
Educational consulting agencies face a high volume of repetitive, low-value requests: answering FAQs (scholarships, visas, housing), collecting administrative data, and scheduling appointments. This project automates those processes with an intelligent, autonomous system that runs on modest hardware, without depending on third-party AI clouds.

### Architecture
The whole stack is container-based, orchestrated with **Docker Compose**, with all services communicating over an internal Docker network.

| Component | Role |
|-----------|------|
| **n8n** | Central orchestrator — visual, low-code workflows triggered by Telegram messages or HTTP webhooks |
| **Ollama** | Local AI inference engine (REST API) hosting two models |
| **LLaMA 3** | Natural-language understanding & answer generation |
| **BGE-base** (BAAI General Embedding) | Document vectorization for semantic search |
| **Supabase** | PostgreSQL + vector store for the RAG knowledge base |
| **Traefik** | Reverse proxy with automatic HTTPS (Let's Encrypt) |
| **Telegram Bot API** | Conversational user interface |
| **Google APIs** (Sheets / Calendar) | Data storage & appointment booking via OAuth2 |
| **DuckDNS** | Free dynamic DNS for a personal domain |

### The RAG pipeline
1. **Chunking** — PDF/TXT documents are split into logical sections (300–500 tokens).
2. **Vectorization** — each chunk is embedded with BGE via a local Ollama call.
3. **Storage** — vectors are stored in Supabase with metadata.
4. **Retrieval** — on a user query, cosine similarity selects the 3–5 closest chunks.
5. **Generation** — query + retrieved context are sent to LLaMA 3, which generates a grounded answer.

The system prompt strictly constrains the AI to answer **only** from the internal documents, drastically reducing hallucinations.

### Workflows implemented
- **Automatic AI reply** — captures the Telegram message, classifies intent, builds a structured prompt, returns a grounded answer (with rephrasing for ambiguous questions and session/history awareness).
- **Student registration** — conversational form: collects name, contact/email, passport number, validates each field, then writes to Google Sheets.
- **Smart appointment booking** — interprets natural language ("Friday afternoon"), checks Google Calendar availability, creates the event, sends a confirmation summary.
- **RAG knowledge ingestion** — admin drops a document on Google Drive → auto-chunked → embedded locally with BGE → inserted into the Supabase vector table.

### Results
All critical workflows were validated through realistic manual test scenarios: FAQ answering grounded on uploaded documents, full student registration into Sheets, natural-language appointment booking, and live RAG enrichment.

### Limitations (honestly documented)
- **Limited VPS performance** — modest RAM/CPU caused slow AI response times, especially during initial model loading. This is the core reason the fully-local instance was not kept in production.
- **Complex setup** — orchestrating Docker, Supabase, Ollama and Traefik requires advanced skills and scattered documentation.
- **Model stability** — some models are memory-hungry and can crash under load.
- **Embedding latency** — vectorization is slow on large document volumes.

### Solutions applied
- A **cloud demo instance** (n8n.io + OpenAI embeddings/GPT-4) was deployed for smooth presentations, replicating the logic without changing the architecture.
- **Document optimization** — controlled chunk sizes (300–500 tokens) to speed up vectorization and improve relevance.
- **Prompt calibration** — reinforced instructions for more structured answers and better handling of ambiguous cases.

### Key takeaway
This project demonstrates that an intelligent, reliable, self-hosted automation system with local AI is feasible even on constrained infrastructure — while laying a modular foundation for future extensions (web UI, admin dashboard, multilingual support).

---

## 🇫🇷 Français

### Présentation
Les agences de conseil en éducation font face à un fort volume de demandes répétitives et à faible valeur : réponses aux FAQ (bourses, visas, logement), collecte de données administratives, prise de rendez-vous. Ce projet automatise ces processus avec un système intelligent et autonome, fonctionnant sur des ressources modestes, sans dépendre de clouds IA tiers.

### Architecture
Toute la stack est conteneurisée, orchestrée avec **Docker Compose**, les services communiquant sur un réseau Docker interne.

| Composant | Rôle |
|-----------|------|
| **n8n** | Orchestrateur central — workflows visuels low-code déclenchés par Telegram ou webhooks HTTP |
| **Ollama** | Moteur d'inférence IA local (API REST) hébergeant deux modèles |
| **LLaMA 3** | Compréhension du langage & génération de réponses |
| **BGE-base** | Vectorisation des documents pour la recherche sémantique |
| **Supabase** | PostgreSQL + base vectorielle pour la base de connaissances RAG |
| **Traefik** | Reverse proxy avec HTTPS automatique (Let's Encrypt) |
| **Telegram Bot API** | Interface conversationnelle |
| **Google APIs** (Sheets / Calendar) | Stockage & prise de rendez-vous via OAuth2 |
| **DuckDNS** | DNS dynamique gratuit pour un domaine personnel |

### Le pipeline RAG
1. **Découpage** — documents PDF/TXT découpés en sections logiques (300–500 tokens).
2. **Vectorisation** — chaque fragment vectorisé via BGE en local (Ollama).
3. **Stockage** — vecteurs stockés dans Supabase avec métadonnées.
4. **Récupération** — sur requête, la similarité cosinus sélectionne les 3–5 fragments les plus proches.
5. **Génération** — requête + contexte envoyés à LLaMA 3 qui génère une réponse ancrée.

Le prompt système contraint l'IA à répondre **uniquement** à partir des documents internes, réduisant fortement les hallucinations.

### Workflows réalisés
- **Réponse IA automatique** — capture le message Telegram, classe l'intention, construit un prompt structuré, renvoie une réponse ancrée (reformulation des questions ambiguës, gestion de l'historique de session).
- **Enregistrement des étudiants** — formulaire conversationnel : nom, contact/email, numéro de passeport, validation de chaque champ, puis écriture dans Google Sheets.
- **Prise de rendez-vous intelligente** — interprète le langage naturel (« vendredi après-midi »), vérifie la disponibilité dans Google Calendar, crée l'événement, envoie une confirmation.
- **Ingestion RAG** — l'admin dépose un document sur Google Drive → découpage auto → vectorisation locale (BGE) → insertion dans la table vectorielle Supabase.

### Résultats
Tous les workflows critiques ont été validés via des scénarios de tests manuels réalistes : réponse FAQ ancrée sur documents, enregistrement complet dans Sheets, prise de rendez-vous en langage naturel, et enrichissement RAG en direct.

### Limitations (documentées honnêtement)
- **Performances VPS limitées** — RAM/CPU modestes → temps de réponse IA parfois longs, surtout au chargement initial des modèles. C'est la raison principale pour laquelle l'instance 100% locale n'a pas été maintenue en production.
- **Installation complexe** — orchestrer Docker, Supabase, Ollama et Traefik demande des compétences avancées.
- **Stabilité des modèles** — certains modèles sont gourmands en mémoire et peuvent planter sous charge.
- **Latence d'embedding** — vectorisation lente sur de gros volumes.

### Solutions apportées
- **Instance cloud de démo** (n8n.io + embeddings/GPT-4 OpenAI) déployée pour des présentations fluides, répliquant la logique sans changer l'architecture.
- **Optimisation des documents** — tailles de fragments contrôlées (300–500 tokens).
- **Calibration du prompt** — instructions renforcées pour des réponses plus structurées.

### À retenir
Ce projet démontre qu'un système d'automatisation intelligent, fiable et auto-hébergé avec IA locale est réalisable même sur une infrastructure contrainte — tout en posant une base modulaire pour des évolutions futures (interface web, tableau de bord admin, multilingue).

---

## 🛠️ Tech Stack
`n8n` · `Docker` · `Docker Compose` · `Ollama` · `LLaMA 3` · `BGE Embeddings` · `Supabase` · `PostgreSQL / pgvector` · `Traefik` · `Telegram Bot API` · `Google Sheets API` · `Google Calendar API` · `OAuth2` · `DuckDNS` · `RAG`

## 📌 Status
Functional prototype — validated through test scenarios. Fully-local production deployment paused due to VPS resource constraints; cloud demo instance used for presentations.

## 👤 Author
**Mbong Joseph Lustigier** — MSc Computer Science, TUSUR (group 544-M)
GitHub: [benus-code](https://github.com/benus-code)
