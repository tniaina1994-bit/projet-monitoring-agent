# AGENTS.md

## Source de vérité

- `Prompt/Server_monitoring.md` = cahier des charges complet du projet. Le lire avant tout code : il impose l'arborescence, les formes JSON de chaque métrique, la config et le workflow en 12 étapes.
- Dépôt encore vierge (aucun code) : seule `requirements.txt` existe (`psutil`, `pytest`) et `.venv/` est déjà installé.

## Commandes (vérifiées, depuis la racine du dépôt)

```bash
# installation (déjà faite ; à refaire seulement si requirements.txt change)
python3 -m venv .venv && .venv/bin/pip install -r requirements.txt

# exécution (spec, pas encore de run.py)
.venv/bin/python run.py          # boucle continue, Ctrl+C propre
.venv/bin/python run.py --once   # une seule collecte

# tests
.venv/bin/pytest -q
.venv/bin/pytest tests/test_cpu.py   # un seul fichier
```

- Python système : 3.14.7 (le spec exige 3.14+). Pas besoin d'activer le venv, utiliser les binaires `.venv/bin/`.

## Règles dures du cahier des charges

- **Phase 1 = agent local Python uniquement.** Aucun FastAPI, PostgreSQL, React, dashboard, alertes ou service systemd — ce sont les phases 2 à 8, à préparer architecturalement mais ne pas implémenter.
- **Dépendances limitées à `psutil`** (+ `pytest` pour les tests). Tout le reste en stdlib (`platform`, `socket`, `datetime`, `json`, `logging`, `pathlib`). Ne rien ajouter d'inutile.
- **Ne jamais enchaîner sur l'étape suivante sans laissser l'utilisateur vérifier la précédente.** Le spec impose un travail étape par étape (structure → system → cpu → memory → disk → network → processes → collector → logging → run.py → tests → README) avec explication + méthode de test après chaque étape.
- **Un collector en échec ne doit jamais tuer l'agent** : chaque collector est isolé, le JSON final reste exploitable avec des erreurs partielles (`⚠`) et des logs compréhensibles.
- Compatibilité Linux **et** Windows : gérer proprement les différences psutil (load average, frequency, `AccessDenied`, partitions inaccessibles).
- Tests `pytest` sans valeurs fixes propres à une machine ; tester aussi la structure des données et la gestion des erreurs.

## Structure attendue (imposée par le spec)

- `run.py` à la racine = point d'entrée ; `agent/main.py`, `agent/collector.py` (agrège tout → JSON final avec `timestamp` + `hostname`), `agent/config.py`, `agent/logger.py`.
- `agent/collectors/` : un fichier par métrique — `system.py`, `cpu.py`, `memory.py`, `disk.py`, `network.py`, `processes.py`.
- `tests/` : `test_cpu.py`, `test_memory.py`, `test_disk.py`, `test_network.py`, `test_system.py`. `logs/` pour les logs.
- Config : `COLLECTION_INTERVAL`, `LOG_LEVEL`, `TOP_PROCESSES` (10 processus par défaut) ; le code doit être prêt à lire un `.env` plus tard sans l'exiger maintenant.

## Conventions

- Type hints, docstrings utiles, fonctions courtes, noms explicites, zéro valeur en dur, pas de design pattern sans justification — l'objectif pédagogique prime.
- Communiquer en français avec l'utilisateur (spec et demandes en français).
