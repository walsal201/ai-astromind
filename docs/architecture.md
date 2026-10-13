# AstroMind — Extended Architecture Notes

## System Architecture Deep Dive

### 1. AI Pipeline

```
User Input (Natural Language)
        │
        ▼
┌─────────────────────┐
│  Prompt Engineering │  ← Domain constants injected (Δv tables, planetary data)
│  Layer              │  ← Chain-of-thought template applied
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  IBM watsonx.ai     │
│  Granite-13b-       │
│  Instruct-v2        │  ← Primary reasoning engine
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Response Parser    │  ← Structured output extraction
│  + Renderer         │  ← Mission card / risk card generation
└─────────────────────┘
```

### 2. Prompt Engineering Strategy

The system uses a **grounded domain prompt** pattern:

```
SYSTEM: You are AstroMind, an expert AI mission planning assistant...
[Domain constants: Δv budgets, communication delays, crew consumables]

USER: Generate a mission profile for [destination] [type] [year] [crew]

ASSISTANT: [Structured mission profile with specific numbers]
```

Key techniques:
- **Domain grounding**: Inject known physical constants and mission heritage data
- **Structured output format**: Prompt specifies expected section headers
- **Chain-of-thought**: For complex trajectory questions, force step-by-step reasoning
- **Stop sequences**: Prevent model from hallucinating follow-up questions

### 3. Frontend Architecture

The prototype is a **single-file HTML application** — no build step, no server required:

```
prototype.html
├── <style> — Dark space theme, responsive grid layout
├── <body>  — 5-tab interface (Dashboard, Trajectory, Risks, Chat, Brief)
└── <script>
    ├── MISSION_DATA       — Orbital mechanics constants
    ├── callWatsonx()      — IBM IAM auth + Granite API call
    ├── SYSTEM_PROMPT      — Domain-grounded AI persona
    ├── DEMO_RESPONSES     — Offline mode pre-generated responses
    ├── Action handlers    — generateMissionProfile, analyzeRisks, etc.
    └── UI utilities       — typeText animation, risk card renderer, SVG orbit viz
```

### 4. IBM watsonx Integration

Authentication flow:
1. User provides API key + Project ID (stored in-memory, never persisted)
2. POST to IBM IAM (`iam.cloud.ibm.com`) to exchange API key for Bearer token
3. POST to watsonx text generation endpoint with Granite model
4. Parse `results[0].generated_text` from response

Model parameters:
- `decoding_method: "greedy"` — deterministic outputs for mission-critical data
- `max_new_tokens: 800` — sufficient for full mission profiles
- `temperature: 0.7` — balanced creativity for document generation

### 5. Orbital Mechanics Module

The `MISSION_DATA` object provides grounding constants:

| Destination | Δv (km/s) | Transit (days) | Peak Dist (AU) | Comm Delay (min) |
|---|---|---|---|---|
| Mars | 5.6–6.3 | 180–270 | 1.52 | 3–22 |
| Europa | 8.8–14.2 | 600–900 | 5.2 | 32–52 |
| Titan | 9.4–15.7 | 900–1800 | 9.58 | 68–84 |
| Asteroid | 3.5–7.2 | 90–400 | 0.8–1.3 | 2–12 |
| Lunar Gateway | 3.2–3.9 | 3–5 | 0.0026 | 0.02 |
| Venus Flyby | 4.7–5.8 | 100–150 | 0.72 | 2–14 |

These constants are injected into the AI prompt, preventing hallucination of physically impossible values.

### 6. Scalability Roadmap

Future extensions planned:
- **Real telemetry integration**: Connect to NASA HORIZONS API for live ephemeris data
- **Multi-leg trajectory optimizer**: Branch-and-bound search for gravity assist sequences
- **Crew simulation module**: Monte Carlo health risk simulation over mission duration
- **Collaborative planning**: Multi-user session with shared mission state
- **Voice interface**: IBM Speech-to-Text integration for hands-free mission planning
