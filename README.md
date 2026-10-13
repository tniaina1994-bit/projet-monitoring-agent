# Server Monitoring Agent

Agent léger en Python qui collecte localement les métriques d'un serveur ou poste Linux/Windows.

> Phase 1 : agent local uniquement (voir `Prompt/Server_monitoring.md` pour le cahier des charges complet).

## Installation

```bash
python -m venv .venv
source .venv/bin/activate        # Windows : .venv\Scripts\activate
pip install -r requirements.txt
```

## Utilisation

```bash
python run.py          # collecte continue
python run.py --once   # une seule collecte
```

## Tests

```bash
pytest -q
```

## Architecture

_(à compléter — Étape 12)_

## Exemple de sortie

_(à compléter — Étape 12)_

## Compatibilité

_(à compléter — Étape 12)_
