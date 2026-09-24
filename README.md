# 🏠 Agent IA — Assistant Conversationnel pour Conciergeries & Locations Courte Durée

> Un agent IA multi-canal capable de renseigner les prospects, gérer les réservations et servir les clients d'une conciergerie, en s'appuyant sur les données réelles d'un Channel Manager — sans solliciter le propriétaire.

![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python)
![LLM](https://img.shields.io/badge/LLM-Agent_IA-green)
![Status](https://img.shields.io/badge/Status-En%20développement-orange)

---

## 🎯 Objectif

Les conciergeries et gestionnaires de locations courte durée traitent en continu les mêmes demandes : disponibilités, tarifs, équipements, réservations, informations d'accès, services additionnels…

Ce projet propose un **agent IA conversationnel** qui répond automatiquement à ces demandes, s'adapte au profil de l'interlocuteur (visiteur, client confirmé, propriétaire), et s'appuie sur les **données réelles** du logement via un Channel Manager — jamais sur des informations inventées.

---

## ✨ Fonctionnalités

**Pour le prospect / visiteur :**
- Consulter les disponibilités (avec calendrier visuel généré à la volée)
- Obtenir un devis chiffré pour des dates précises
- Découvrir les caractéristiques du logement, les avis clients, la FAQ
- Explorer les extras (services) et produits proposés
- Lancer une demande de réservation avec lien de paiement pré-rempli
- Être recontacté (capture de lead multi-canal)

**Pour le client confirmé :**
- Accéder aux informations sensibles (adresse, code d'accès, WiFi) après vérification
- Consulter sa réservation, demander une modification de dates
- Commander et payer des extras

**Pour le propriétaire :**
- Consulter l'ensemble des réservations et les tâches en attente

---

## 🏗️ Architecture

Le principe central : **l'agent ne parle jamais directement à la base de données**. Tout passe par un backend Flask qui gère l'authentification, les rôles et la traduction des données.

```
   Canaux (Web / WhatsApp / Instagram)
                │
                ▼
          Agent IA (LLM + tools)
                │  n'appelle QUE le backend
                ▼
        Backend Flask (proxy)
                │  génère le jeton, filtre par rôle
                ▼
        Channel Manager (source des données)
```

Chaque couche a une responsabilité stricte :
- **L'agent** décide quel outil appeler ; il ne détient aucun secret.
- **Flask** génère les jetons du Channel Manager, applique le contrôle d'accès par rôle et traduit les données.
- **Le Channel Manager** reste la seule source de vérité (logements, réservations, disponibilités).

---

## 🔐 Sécurité par conception

- **Contrôle d'accès côté Python, jamais par le LLM** : les données sensibles ne sont renvoyées qu'après vérification du rôle.
- **Filtrage des outils par rôle** : un prospect ne voit que les outils publics ; les outils propriétaire/client lui sont invisibles.
- **Jetons gérés côté serveur** : la clé d'organisation ne quitte jamais le backend.
- **Séparation stricte** entre données publiques (nom, prix, équipements) et sensibles (adresse, code, WiFi).

---

## 🧠 Choix techniques

| Composant | Choix | Justification |
|---|---|---|
| Agent conversationnel | LLM avec tool calling | Compréhension du langage + actions réelles via outils |
| Modèles | Local (Ollama) ou API (Groq) | Comparaison local/API, bascule possible |
| Accès aux données | Backend Flask → Channel Manager | Sécurité, contrôle par rôle, aucune donnée inventée |
| Recherche FAQ | RAG (embeddings + similarité) | Répondre même à des formulations différentes |
| Calendrier | Génération d'image (Plotly) | Disponibilités visuelles |
| Interface de test | Streamlit | Test multi-profils et multi-canaux |

---

## 🧰 Les outils de l'agent

L'agent dispose d'outils métier, filtrés selon le rôle de l'interlocuteur :

- `obtenir_disponibilites`, `obtenir_devis`, `obtenir_infos_appartement`
- `obtenir_faq`, `obtenir_reviews`, `obtenir_extras`, `obtenir_produit`
- `demander_reservation`, `send_lead`, `noter_contexte`
- `obtenir_information_sensible`, `demander_modification` (client confirmé)
- `obtenir_reservation_proprio`, `obtenir_taches` (propriétaire)

---

## 🚀 Installation

```bash
git clone <url-du-repo>
cd <repo>

python -m venv .venv
source .venv/bin/activate      # Windows : .venv\Scripts\activate

pip install -r requirements.txt
```

Copiez le fichier d'exemple de configuration et renseignez vos propres valeurs :

```bash
cp .env.example .env
# puis éditez .env avec vos clés et URLs
```

Lancez le backend Flask puis l'interface de test :

```bash
python -m src.myapp.run           # backend Flask
streamlit run streamlit_test.py   # interface de test
```

---

## ⚙️ Configuration (.env)

Les secrets ne sont **jamais** versionnés. Renseignez-les dans un fichier `.env` local :

```
AGENT_API_KEY=votre_cle_organisation
URL_TOKEN_CM=url_du_endpoint_de_jeton
URL_API_CM=url_de_base_du_channel_manager
GROQ_API_KEY=votre_cle_groq        # si usage API
```

Le fichier `.env` est ignoré par Git (voir `.gitignore`). Un `.env.example` documente les variables attendues.

---

## 🗺️ Roadmap

- [x] Architecture agent → Flask → Channel Manager
- [x] Gestion des rôles (prospect / client confirmé / propriétaire)
- [x] Outils : disponibilités, devis, infos, FAQ, avis, extras, produits
- [x] Réservation avec lien de paiement et extras pré-remplis
- [x] Capture de leads multi-canal avec fusion
- [x] Calendrier visuel des disponibilités
- [x] RAG sur la FAQ
- [x] Filtrage des outils par rôle
- [ ] Mémoire de contexte robuste (extraction automatique)
- [ ] Sécurité renforcée (JWT interne) pour la mise en production
- [ ] Canaux réels (WhatsApp, Instagram)
- [ ] Dashboard propriétaire
- [ ] Hébergement dédié (souveraineté des données)

---

## 💡 Cas d'usage visés

- Conciergeries et gestionnaires de biens
- Locations Airbnb et courte durée
- Hôtels et résidences de tourisme

---

## 🛠️ Stack

`Python` · `Flask` · `LLM (Ollama / Groq)` · `Tool calling` · `RAG` · `Plotly` · `Streamlit`

---

## ⚠️ Statut

Projet en développement actif. L'architecture, les rôles et les outils principaux sont fonctionnels ; la sécurité avancée (JWT interne) et les canaux de production (WhatsApp, Instagram) sont en cours.

---

## 📄 Licence

GPL-3.0
