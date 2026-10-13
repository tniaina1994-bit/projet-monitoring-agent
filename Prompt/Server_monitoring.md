# [02] Server Monitoring Agent — Phase 1 : Agent local V1

Je veux développer un projet nommé **Server Monitoring Agent** destiné à améliorer mes compétences en administration système, réseau et cybersécurité.

## 1. Objectif du projet

Créer un agent léger en **Python** capable de collecter localement les informations et métriques importantes d'un serveur ou d'un poste Linux/Windows.

Pour cette première phase, **ne pas développer FastAPI, PostgreSQL, React ou le dashboard**.

L'objectif est uniquement de construire un **agent local fiable**, propre et extensible.

## 2. Technologies

Utiliser :

* Python 3.14+
* `psutil`
* `platform`
* `socket`
* `datetime`
* `json`
* `logging`
* `pathlib`
* `dataclasses` ou Pydantic uniquement si réellement nécessaire
* Git

Ne pas ajouter de dépendance inutile.

## 3. Architecture

Créer cette structure :

```text
server-monitoring-agent/
│
├── agent/
│   ├── collectors/
│   │   ├── __init__.py
│   │   ├── cpu.py
│   │   ├── memory.py
│   │   ├── disk.py
│   │   ├── network.py
│   │   ├── processes.py
│   │   └── system.py
│   │
│   ├── config.py
│   ├── collector.py
│   ├── main.py
│   └── logger.py
│
├── tests/
│   ├── test_cpu.py
│   ├── test_memory.py
│   ├── test_disk.py
│   ├── test_network.py
│   └── test_system.py
│
├── logs/
│
├── .gitignore
├── requirements.txt
├── README.md
└── run.py
```

## 4. Métriques à collecter

### CPU

Utiliser `psutil` pour récupérer :

* utilisation CPU globale en %
* nombre de CPU physiques
* nombre de CPU logiques
* load average lorsque disponible
* fréquence CPU lorsque disponible

Exemple :

```json
{
  "usage_percent": 42.5,
  "physical_cores": 4,
  "logical_cores": 8,
  "load_average": {
    "1m": 1.21,
    "5m": 0.95,
    "15m": 0.87
  }
}
```

Attention : certaines informations ne sont pas disponibles de la même manière sur Windows et Linux.

Le code doit gérer ces différences proprement.

## 5. Mémoire RAM

Collecter :

* RAM totale
* RAM utilisée
* RAM disponible
* pourcentage utilisé
* éventuellement swap

Retour attendu :

```json
{
  "total_mb": 8192,
  "used_mb": 5432,
  "available_mb": 2760,
  "usage_percent": 66.3,
  "swap": {
    "total_mb": 4096,
    "used_mb": 512,
    "usage_percent": 12.5
  }
}
```

## 6. Disques

Identifier les partitions disponibles et collecter :

* point de montage
* espace total
* espace utilisé
* espace libre
* pourcentage utilisé

Exemple :

```json
{
  "partitions": [
    {
      "mountpoint": "/",
      "total_gb": 500,
      "used_gb": 350,
      "free_gb": 150,
      "usage_percent": 70
    }
  ]
}
```

Le code doit éviter les partitions ou périphériques qui provoquent des erreurs d'accès.

Il doit fonctionner correctement sur Linux et Windows autant que possible.

## 7. Réseau

Collecter :

* interfaces réseau
* adresse IP
* adresse MAC lorsque disponible
* bytes reçus
* bytes envoyés
* paquets reçus
* paquets envoyés
* erreurs réseau si disponibles

Exemple :

```json
{
  "interfaces": [
    {
      "name": "eth0",
      "addresses": [],
      "bytes_sent": 15234567,
      "bytes_received": 45345678
    }
  ]
}
```

Ne pas exposer inutilement des informations sensibles.

## 8. Processus

Créer un collector permettant d'obtenir les processus consommant le plus de ressources.

Pour chaque processus, récupérer lorsque disponible :

* PID
* nom
* utilisateur
* CPU %
* mémoire %
* mémoire RSS
* statut

Retourner par défaut les **10 processus les plus consommateurs**.

Exemple :

```json
{
  "top_processes": [
    {
      "pid": 1234,
      "name": "nginx",
      "username": "root",
      "cpu_percent": 12.4,
      "memory_percent": 2.8,
      "status": "running"
    }
  ]
}
```

Gérer correctement :

* `AccessDenied`
* processus terminés pendant la collecte
* processus inexistants

Une erreur sur un processus ne doit pas arrêter tout l'agent.

## 9. Informations système

Collecter :

* hostname
* OS
* version OS
* architecture
* version Python
* uptime
* date/heure de collecte

Exemple :

```json
{
  "hostname": "server01",
  "os": "Linux",
  "os_version": "...",
  "architecture": "x86_64",
  "python_version": "3.11.x",
  "uptime_seconds": 86400
}
```

## 10. Collector principal

Créer un module central qui exécute tous les collectors :

```text
CPU
RAM
Disk
Network
Processes
System
   ↓
Collector principal
   ↓
JSON final
```

Le résultat global doit avoir cette forme :

```json
{
  "timestamp": "2026-09-28T12:00:00Z",
  "hostname": "server01",
  "system": {},
  "cpu": {},
  "memory": {},
  "disk": {},
  "network": {},
  "processes": []
}
```

## 11. Configuration

Créer un fichier de configuration permettant de définir notamment :

```text
COLLECTION_INTERVAL=10
LOG_LEVEL=INFO
TOP_PROCESSES=10
```

Pour cette première version, la configuration peut être simple.

Préparer cependant le code pour pouvoir utiliser plus tard un fichier `.env`.

## 12. Logging

Mettre en place un système de logs avec :

* INFO
* WARNING
* ERROR
* DEBUG

Les erreurs doivent être compréhensibles.

Exemple :

```text
2026-09-28 12:00:00 INFO Monitoring agent started
2026-09-28 12:00:01 INFO Collecting system metrics
2026-09-28 12:00:02 WARNING Unable to access process PID 1234
```

## 13. Exécution

Créer une commande simple :

```bash
python run.py
```

L'agent doit :

1. démarrer
2. collecter les métriques
3. afficher le résultat dans le terminal
4. produire un JSON propre
5. continuer périodiquement selon l'intervalle configuré
6. gérer proprement `Ctrl+C`

Ajouter également une option permettant de réaliser **une seule collecte**, par exemple :

```bash
python run.py --once
```

## 14. Gestion des erreurs

Le projet doit être robuste.

Ne jamais arrêter complètement l'agent parce qu'un seul collector rencontre une erreur.

Exemple :

```text
CPU       ✓
RAM       ✓
Disk      ✓
Network   ✓
Process   ⚠ erreur partielle
System    ✓
```

Le résultat final doit toujours être exploitable.

## 15. Tests

Créer des tests unitaires avec `pytest`.

Tester au minimum :

* récupération CPU
* récupération RAM
* récupération disque
* récupération réseau
* récupération informations système
* structure des données retournées
* gestion des erreurs
* fonctionnement du collector principal

Les tests ne doivent pas dépendre de valeurs fixes propres à une machine.

## 16. README

Créer un README expliquant :

### Installation

```bash
git clone ...
cd server-monitoring-agent

python -m venv .venv
```

Linux/macOS :

```bash
source .venv/bin/activate
```

Windows :

```powershell
.venv\Scripts\activate
```

Puis :

```bash
pip install -r requirements.txt
```

### Utilisation

```bash
python run.py
```

et :

```bash
python run.py --once
```

### Architecture

Expliquer clairement le rôle de chaque fichier.

### Exemple de sortie

Ajouter un exemple JSON réaliste.

### Compatibilité

Indiquer clairement les fonctionnalités compatibles Linux/Windows et les éventuelles limitations propres à chaque OS.

## 17. Qualité du code

Je veux apprendre de ce projet.

Donc :

* code propre
* fonctions courtes
* séparation des responsabilités
* type hints
* docstrings utiles
* noms de variables explicites
* pas de code inutilement complexe
* gestion propre des exceptions
* éviter les fonctions gigantesques
* éviter les valeurs codées en dur
* privilégier une architecture facilement extensible

Ne pas utiliser de design pattern complexe sans justification.

## 18. Préparation pour les prochaines phases

L'architecture doit permettre d'ajouter plus tard :

```text
Phase 1
Agent local
    ↓
Phase 2
FastAPI
    ↓
Phase 3
PostgreSQL
    ↓
Phase 4
Monitoring multi-serveurs
    ↓
Phase 5
Dashboard React
    ↓
Phase 6
Alertes
    ↓
Phase 7
Authentification / sécurité
    ↓
Phase 8
Déploiement systemd / Windows Service
```

Ne pas implémenter ces fonctionnalités maintenant.

Mais organiser le code afin qu'elles puissent être ajoutées sans devoir réécrire complètement l'agent.

## 19. Méthode de développement

Ne génère pas tout le projet d'un seul coup.

Travaille étape par étape.

Commence par :

### Étape 1

Créer la structure du projet.

### Étape 2

Implémenter `system.py`.

### Étape 3

Implémenter `cpu.py`.

### Étape 4

Implémenter `memory.py`.

### Étape 5

Implémenter `disk.py`.

### Étape 6

Implémenter `network.py`.

### Étape 7

Implémenter `processes.py`.

### Étape 8

Créer le collector principal.

### Étape 9

Ajouter le logging.

### Étape 10

Créer `run.py`.

### Étape 11

Ajouter les tests.

### Étape 12

Finaliser le README.

Après chaque étape :

1. expliquer ce qui a été créé ;
2. expliquer pourquoi ;
3. afficher les fichiers modifiés ;
4. fournir le code ;
5. expliquer comment tester ;
6. vérifier les erreurs éventuelles avant de passer à l'étape suivante.

**Important : ne passe pas automatiquement à l'étape suivante sans me laisser vérifier la précédente.**

Le but est que je puisse comprendre et apprendre l'administration système Python en construisant réellement cet outil.
