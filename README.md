# 🎼 RiffToSheets

![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)
![Docker](https://img.shields.io/badge/Docker-Compose%20%2F%20Portainer-2496ED?logo=docker&logoColor=white)
![Status](https://img.shields.io/badge/Status-En%20d%C3%A9veloppement-amber)

**RiffToSheets** est une application web open-source et auto-hébergée de **reconnaissance d'instruments et de transcription musicale assistée par IA**. Elle permet de transformer un jeu instrumental en direct ou des fichiers audio complets en **partitions PDF**, **tablatures PDF** (guitare/basse) et fichiers **MIDI (`.mid`)**, le tout exécuté en local sur GPU NVIDIA.

---

## ✨ Fonctionnalités principales

### 🔴 Mode Live *(Temps réel)*
* **Capture microphone directe** depuis navigateur (PC et mobile via HTTPS / WebSockets).
* **Sélection de l'instrument avant capture** pour calibrer les plages de fréquences et la clé musicale :
  * *Très fiables :* Piano / Clavier, Guitare (acoustique ou électrique son clair / léger crunch), Basse électrique, Voix.
  * *Fiables :* Cordes frottées (Violon, Alto, Violoncelle), Vents (Flûte, Clarinette, Saxophone, Trompette), Synthétiseur propre.
  * *Exclus en direct :* Batterie / percussions non accordées et guitares à forte distorsion.
* **Choix Partition ou Tablature** dès le démarrage pour la guitare et la basse.
* **Aides rythmiques intégrées :** réglage manuel du tempo (BPM), signature rythmique (`4/4`, `3/4`, `6/8`…) et métronome visuel et sonore exécuté en local côté navigateur (Web Audio API).
* **Affichage en direct** de la portée pendant le jeu.

### 🔵 Mode Import *(Traitement de fichiers)*
* **Multi-formats audio :** `FLAC`, `AAC`, `MP3`, `WAV`, `OGG`, `M4A`, `Opus` (normalisés côté serveur via FFmpeg).
* **Envois volumineux :** jusqu'à **5 fichiers** et **3 Go cumulés** par lot, sans limite de durée, grâce à un téléversement découpé par morceaux de 50 Mo (*chunked upload*, compatible avec la limite de 100 Mo de Cloudflare Tunnel).
* **Deux niveaux de précision :**
  * **Normal :** analyse rapide mono-GPU sur fichier complet.
  * **Maximum :** séparation multi-pistes fine (`htdemucs_ft`), transcription piste par piste et mise en forme avancée par LLM.
* **Exports flexibles par instrument :** téléchargement individuel pour chaque piste détectée (ou en lot complet) aux formats **PDF Partition**, **Tablature PDF** (guitare/basse) et **MIDI (`.mid`)**.

### 👤 Comptes, File d'attente & Rétention
* **Authentification :** inscription par e-mail et mot de passe (sans serveur SMTP requis) et connexion **Google OAuth** optionnelle.
* **File d'attente FIFO stricte :** une seule analyse lourde exécutée à la fois sur le serveur pour préserver la mémoire vidéo (VRAM), avec affichage de la position en temps réel.
* **Politique de rétention automatique :**
  * Partitions générées et fichiers audio **< 1 Go** : conservés **30 jours** puis purgés automatiquement.
  * Fichiers audio **≥ 1 Go** : supprimés **immédiatement** après la génération des exports.

---

## 🧠 Architecture technique & IA

| Couche | Technologies utilisées |
| :--- | :--- |
| **Frontend** | React (Vite + Tailwind CSS), interface responsive sobre (mode clair par défaut + mode sombre), rendu musical (`abcjs` / `alphaTab`) |
| **Backend & Queue** | Python (FastAPI), WebSockets, Redis (file d'attente FIFO), PostgreSQL |
| **Traitement audio (AMT)** | FFmpeg, Spotify Basic-Pitch, HTDemucs (`htdemucs` / `htdemucs_ft`), YourMT3+ |
| **Mise en forme IA (LLM)** | Ollama intégré en conteneur (`qwen3.5:9b` en Live, `gpt-oss:20b` en Normal, `qwen3.8:27b` en Maximum) |
| **Génération d'exports** | `music21`, algorithme de doigtés guitare/basse, LilyPond (PDF), export MIDI |
| **Déploiement** | Docker Compose / Portainer sur le port `9134`, compatible reverse proxy / Cloudflare Tunnel |

---

## 🛠️ Prérequis matériel & logiciel

* **Système :** Docker + Docker Compose (ou Portainer) avec **NVIDIA Container Toolkit** activé.
* **GPU recommandé :** Carte NVIDIA compatible CUDA (16 Go de VRAM recommandés pour exploiter confortablement le pipeline temps réel et le mode Import ; architecture optimisée pour les configurations mono ou bi-GPU).
* **Réseau / HTTPS :** L'accès au microphone sur mobile ou depuis un autre appareil du réseau (`navigator.mediaDevices.getUserMedia`) **exige impérativement une connexion HTTPS** (sauf sur `localhost`). L'utilisation d'un reverse proxy SSL ou d'un **Cloudflare Tunnel** est recommandée.

---

## 🚀 Installation & Déploiement rapide

1. **Cloner le dépôt :**
   ```bash
   git clone [https://github.com/](https://github.com/)<TON_PSEUDO>/RiffToSheets.git
   cd RiffToSheets
   ```

2. **Configurer les variables d'environnement :**
   Copiez le fichier d'exemple et renseignez vos secrets (clé JWT, identifiants PostgreSQL, et optionnellement vos clés Google OAuth) :
   ```bash
   cp .env.example .env
   ```

3. **Lancer la stack (Docker Compose ou Portainer) :**
   ```bash
   docker compose up -d --build
   ```
   L'application est ensuite accessible localement sur `http://<IP_DU_SERVEUR>:9134`.

4. **Exposition distante (exemple Cloudflare Tunnel) :**
   Dans votre tunnel, créez une route publique pointant `https://rifttosheets.votre-domaine.tld` vers `http://<IP_DU_SERVEUR>:9134` (en vous assurant que le port TCP `9134` est autorisé sur le pare-feu local de la machine hôte).

---

## ⚖️ Mentions légales, Droits d'auteur & Licence

### 1. Licence du code source (MIT)
Le code source original de **RiffToSheets** contenu dans ce dépôt est distribué sous **licence MIT**. Vous êtes libre d'utiliser, copier, modifier, fusionner, publier et distribuer ce code, sous réserve de conserver la mention de copyright et le texte de la licence.

Consultez le fichier [`LICENSE`](./LICENSE) à la racine du dépôt pour le texte complet.

> *Copyright (c) 2026 ItsLolilol*

### 2. Composants, bibliothèques et modèles tiers
La licence MIT de ce dépôt s'applique **uniquement au code source de RiffToSheets**. L'application s'appuie sur des outils externes, bibliothèques open-source et modèles d'intelligence artificielle téléchargés ou exécutés séparément (notamment via Docker, Python et Ollama), qui **restent intégralement régis par leurs propres licences respectives**, parmi lesquelles :

* **Basic-Pitch** (Spotify) — Apache-2.0
* **Demucs / HTDemucs** (Meta) — MIT
* **YourMT3+** — Apache-2.0
* **Ollama** — MIT
* **Modèles LLM** (`Qwen`, `GPT-OSS`, `Gemma`, etc.) — soumis aux licences et conditions d'utilisation propres à leurs éditeurs respectifs lors de leur téléchargement via Ollama
* **FFmpeg** — LGPL v2.1+ / GPL v2+ *(exécuté en tant que processus système externe)*
* **LilyPond** — GNU GPL v3 *(exécuté en tant que processus système externe)*
* **music21** — BSD-3-Clause

Aucune clause de ce dépôt ne modifie ni ne remplace les licences de ces projets tiers. Il incombe à chaque utilisateur ou déployeur d'en respecter les termes (notamment en cas de redistribution de conteneurs ou d'usage commercial).

### 3. Propriété intellectuelle & Droits d'auteur musicaux
**RiffToSheets** est un outil technique de transcription destiné à un **usage personnel, éducatif, analytique ou créatif** (compositions originales, œuvres du domaine public ou enregistrements pour lesquels l'utilisateur détient les droits ou autorisations nécessaires).

* L'auteur de ce projet **n'encourage ni ne cautionne aucune atteinte aux droits d'auteur**.
* Chaque utilisateur est **seul responsable** des fichiers audio qu'il importe et de l'usage (partage, publication, exploitation) qu'il fait des partitions, tablatures ou fichiers MIDI générés.

### 4. Auto-hébergement & Protection des données (RGPD)
RiffToSheets est un logiciel destiné à être **auto-hébergé**. L'auteur du dépôt ne collecte, ne stocke et ne traite aucune donnée issue des instances déployées par des tiers.

Si vous déployez une instance accessible au public (notamment avec création de comptes, authentification Google OAuth et stockage temporaire de fichiers audio), **vous agissez en qualité de responsable de traitement** au sens du RGPD et il vous appartient de publier vos propres mentions légales, conditions d'utilisation et politique de confidentialité.

### 5. Limitation de responsabilité & Précision de l'IA
Conformément à la licence MIT, ce logiciel est fourni **« en l'état » (*AS IS*), sans garantie d'aucune sorte**. Les transcriptions produites par intelligence artificielle (en particulier en Mode Live) sont de nature probabiliste et peuvent comporter des imprécisions rythmiques, harmoniques ou de doigtés.
