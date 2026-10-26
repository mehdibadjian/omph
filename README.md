# OMPH — Zero-Cost Resilient Agentic Workflow (`Oh My Pi` + `Oh My Hermes` + OpenRouter Free Tier)

Production configuration and architecture documentation for running **Oh My Pi (`omp`)** and **Oh My Hermes (`omh`)** exclusively on **OpenRouter's `:free` models** with multi-vendor rate-limit retry and automatic failover cascades.

## Repository Structure

```text
.
├── README.md
├── docs/
│   ├── BRD.md                      # Business Requirements Document
│   └── SPEC.md                     # Technical Architecture & Implementation Spec
└── config/
    ├── .env.example                # OpenRouter API key environment template
    ├── omp-config.example.yml      # Oh My Pi (~/.omp/agent/config.yml)
    ├── omh-model-chains.json       # Oh My Hermes (~/.omh/routing/model-chains.json)
    └── hermes-config.example.yaml  # Hermes Agent (~/.hermes/config.yaml)
```

## Quick Setup

```bash
mkdir -p ~/.omp/agent ~/.omh/routing ~/.hermes
cp config/omp-config.example.yml ~/.omp/agent/config.yml
cp config/omh-model-chains.json ~/.omh/routing/model-chains.json
cp config/hermes-config.example.yaml ~/.hermes/config.yaml
```
