j'ai utilisé l'intelligence artificielle pour créer ses codes alors il vous suffit d'aller sur cet URL pour comprendre comment il marche et pour le modifier immédiatement 
https://chat.deepseek.com/share/3yae3l0nntan8qqiug

# 🎨 Egon Schiele Lab — Générateur d'œuvres par Agnes AI

**Transformez une simple photo en une série de portraits et de scènes stylisées par Egon Schiele, en conservant uniquement le visage, les habits et le décor — la palette de couleurs est entièrement celle de l'artiste viennois.**

Ce dépôt contient **deux applications HTML autonomes** (aucun serveur, aucune compilation) qui exploitent l'API **Agnes AI** pour générer des images dans le style d'Egon Schiele. Chaque fichier est un « lab » indépendant, avec sa propre interface et son propre jeu de concepts.

---

## 📂 Contenu du dépôt

| Fichier | Version | Description |
|---|---|---|
| `deepseek_html_20261016_a8841e.html` | **v2** | 20 concepts viennois · analyse du visage seul · génération 100 % texte-vers-image · fallback manuel · console de diagnostic |
| `deepseek_html_20261017_fc5c42 (1).html` | **v3** | 40 œuvres pré-chargées · analyse visage + habits + décor · palette Schiele imposée · bouton **Continuer (+40)** pour étendre la liste via LLM |

Ouvrez simplement le fichier HTML de votre choix dans un navigateur moderne (Chrome, Firefox, Edge, Safari). Tout fonctionne côté client.

---

## ✨ Fonctionnalités communes (v2 & v3)

### 🔑 Gestion multi-clés API
- **Saisie multiligne** : collez plusieurs clés Agnes AI (une par ligne) dans un seul champ texte.
- **Validation par ping** : chaque clé est testée avant d'être enregistrée (code HTTP 401/403 = rejetée).
- **Rotation automatique** : les clés valides sont utilisées en alternance pour répartir la charge.
- **Paceur RPM** : ~3,4 secondes entre deux requêtes par clé (≈ 12 requêtes/minute/clé).
- **Retry ×6** sur les erreurs transitoires `429`, `502`, `503`, `504`, avec backoff exponentiel.

### 🖼️ Analyse de la photo source
- **Envoi unique à l'API de vision** (jamais à l'API de génération d'images).
- **Compression automatique** de l'image (max 1024 px, JPEG qualité 0,85 → 1,2 Mo) avant envoi.
- **Parsing JSON tolérant** à 4 niveaux : JSON direct, JSON réparé (guillemets/échappements), extraction d'objet équilibré, puis fallback sur le texte brut.
- **Fallback manuel** : si l'analyse automatique échoue, un panneau permet de saisir soi-même les descriptions (visage seul en v2 ; visage + habits + décor en v3).
- **Panneau de diagnostic** : affichage de la réponse brute du modèle en cas d'erreur.

### 🎨 Génération d'images
- **100 % texte-vers-image pur** : aucun `extra_body.image` n'est envoyé à l'endpoint `/images/generations`. Le modèle d'image ne reçoit que le prompt textuel.
- **Format vertical 9:16** (1024 × 1792 px) adapté aux portraits et aux compositions de Schiele.
- **Prompts structurés** : style de l'œuvre, ADN du personnage (verrouillé), pose/composition, technique Schiele, règles absolues.
- **Copie des prompts** : chaque carte de résultat expose le prompt image et le prompt vidéo (pour Google Flow) avec un bouton « Copier ».

### 🖥️ Interface et confort
- **Thème sombre** inspiré de la palette de Schiele (ocres, bruns, carmin).
- **Console double** : journal humain (messages lisibles) + journal de débogage (HTTP, erreurs, réponses brutes).
- **Barre de progression**, **compteurs en temps réel**, **notifications toast**, **barre de statut** avec spinner.
- **Persistance multi-niveau** : `localStorage` → `sessionStorage` → mémoire (fallback si le stockage est bloqué).
- **Bouton Stop** pour interrompre une génération en cours.
- **Responsive** : de 1 à 4 colonnes selon la largeur de l'écran.

---

## 🆕 Spécificités de la version 3

### 40 œuvres d'Egon Schiele pré-chargées
La v3 intègre directement dans le code source **40 œuvres** soigneusement sélectionnées, couvrant :
- **Autoportraits** (aux lanternes chinoises, épaule levée, grimaces…)
- **Portraits** (Edith Schiele, Wally Neuzil, Arthur Roessler…)
- **Paysages et villes** (Krumau, Trieste, ville morte, maisons en hiver…)
- **Scènes allégoriques et familiales** (Mère et enfant, La Famille, Le Cardinal et la Nonne…)
- **Natures mortes** (Tournesols, arbres…)

Chaque œuvre est décrite par un **style** (technique et palette), une **pose** (composition) et un **prompt vidéo** pour l'animation.

### 🎨 Palette Schiele exclusive
Le prompt de génération contient une règle explicite :
> *« DO NOT use the colors of the reference photograph. The reference only provides face, clothes shape and decor shape. Colors come EXCLUSIVELY from Egon Schiele's palette. »*

L'analyse de la photo source demande au modèle de vision de **ne pas décrire les couleurs**, mais uniquement les **formes, coupes, tissus et contexte**. Ainsi, chaque œuvre générée possède sa propre ambiance colorée, toujours dans les tons d'Egon Schiele (ocres poussiéreux, ombres brun-rouge, chairs pâles, gris éteints, accents carmin).

### 🚫 Interdiction stricte de nudité
Même lorsque l'œuvre originale de Schiele contient un nu, le prompt impose des **figures entièrement vêtues** :
> *« FIGURES FULLY CLOTHED. No nudity, no nakedness, no exposed torso, no bare chest, no revealed genitals. Even if the original artwork depicts nudity, this version shows fully dressed figures. »*

Les 40 œuvres de la v3 ont été sélectionnées pour être **compatibles avec cette règle** (ou adaptées en conséquence).

### ➕ Bouton « Continuer (+40) »
Une fois les 40 œuvres initiales générées, le bouton **Continuer (+40)** :
1. Envoie au LLM Agnes la liste des titres déjà utilisés.
2. Demande **40 nouvelles œuvres** (réelles ou plausibles) au format JSON, sans doublons et sans nudité.
3. Ajoute les nouvelles œuvres à la galerie.
4. Lance automatiquement la génération des images correspondantes.

**Garde-fou tokens** : au-delà de **300 œuvres**, seules les **200 dernières** sont renvoyées au LLM pour éviter la saturation du contexte. Cela permet de viser **400 à 500 œuvres** au total sans dépasser les limites de tokens.

---

## 🚀 Démarrage rapide

### Prérequis
- Un navigateur moderne.
- Une ou plusieurs **clés API Agnes AI** (format `sk-...`).

### Étapes
1. **Ouvrez** le fichier HTML souhaité (v2 ou v3) dans votre navigateur.
2. **Collez vos clés API** dans le champ multiligne, puis cliquez sur **« Valider et enregistrer »**.
3. **Chargez une photo** (visage, buste, scène) en cliquant sur la zone de dépôt.
4. **Cliquez sur « Analyser »** :
   - v2 : extrait une description forensique du visage.
   - v3 : extrait le visage, les habits et le décor (sans les couleurs).
5. **Sélectionnez les concepts** (v2) ou utilisez les **40 œuvres pré-chargées** (v3).
6. **Lancez la génération** :
   - v2 : choisissez le nombre d'images par concept, puis cliquez sur **« Générer »**.
   - v3 : cliquez sur **« Générer toutes les œuvres »**.
7. **Suivez la progression** dans la console et la barre de progression.
8. (v3 uniquement) Cliquez sur **« Continuer (+40) »** pour étendre la collection.

---

## ⚙️ Détails techniques

### API utilisée
- **Base URL** : `https://apihub.agnes-ai.com/v1`
- **Modèle texte** : `agnes-2.5-flash` (analyse vision + propositions LLM)
- **Modèle image** : `agnes-image-2.1-flash` (génération d'images)
- **Taille d'image** : `1024x1792` (format vertical 9:16)

### Endpoints
| Fonction | Endpoint | Méthode |
|---|---|---|
| Analyse de la photo | `/chat/completions` | POST (avec `image_url`) |
| Proposition de nouvelles œuvres (v3) | `/chat/completions` | POST (texte seul) |
| Génération d'image | `/images/generations` | POST (texte seul) |

### Prompt de génération (structure)
Chaque prompt est construit en sections :

```
════════ EGON SCHIELE STYLE — {année} ════════
{style de l'œuvre}

════════ CHARACTER — IDENTICAL ACROSS ALL IMAGES ════════
This is a {femme/homme}, {âge}.
FACE (do not alter any feature): {ADN du visage}
CLOTHES (style/cut/fabric — colors will be Schiele's): {habits}
DECOR (setting/objects — colors will be Schiele's): {décor}
DISTINCTIVE FEATURE: {marques distinctives}

════════ POSE / COMPOSITION ════════
{pose}

════════ SCHIELE PALETTE — MANDATORY ════════
- Dusty ochre, brown-red shadows, pale flesh, muted greys.
- Occasional vivid carmine red accent.
- DO NOT use the colors of the reference photograph.

════════ TECHNIQUE SCHIELE ════════
- Raw expressive black ink contour lines.
- Angular, gaunt, distorted anatomy.
- Muted watercolor and gouache on visible aged paper.
- Viennese Expressionism 1910s.

════════ ABSOLUTE RULES ════════
- FIGURES FULLY CLOTHED. No nudity.
- The FACE is IDENTICAL to the reference description.
- Hand-painted ARTWORK, NOT a photograph.
- Vertical 9:16 composition.
- NO text, NO watermark, NO signature.
- Not photorealistic, not 3D, not CGI.
```

### Analyse de la photo (v3) — prompt de vision
Le modèle de vision reçoit l'instruction de décrire **uniquement les formes et le contexte**, jamais les couleurs :
- `face_dna` : 100–140 mots en anglais (forme du crâne, mâchoire, yeux, nez, lèvres, peau, cheveux).
- `clothes_dna` : 30–50 mots (style, coupe, tissu, accessoires — **sans couleurs**).
- `decor_dna` : 30–50 mots (arrière-plan, objets, atmosphère — **sans couleurs**).
- `distinctive` : 15–30 mots (marques, tatouages, cicatrices, bijoux).

### Gestion des erreurs
| Situation | Comportement |
|---|---|
| Clé invalide (401/403) | Rejetée à la validation |
| `429` (rate limit) | Attente puis retry (×6) |
| `502`/`503`/`504` | Attente exponentielle puis retry |
| Timeout (90 s) | Retry |
| Réponse sans image | Retry |
| Analyse JSON illisible | Fallback texte brut → fallback manuel |
| LLM v3 renvoie du JSON invalide | Extraction d'array tolérante + réparation |

---

## 🎯 Cas d'usage

- **Portraits personnalisés** : à partir d'un selfie, obtenez une galerie de portraits dans l'esprit de Schiele.
- **Illustration de personnages** : verrouillez un visage et des habits, puis déclinez-le dans 40+ styles de Schiele.
- **Exploration artistique** : découvrez la palette et les compositions de l'artiste viennois appliquées à vos propres images.
- **Base de prompts** : copiez les prompts générés (image + vidéo) pour les réutiliser dans d'autres outils (Google Flow, etc.).

---

## 📝 Notes et limites

- **Aucune donnée n'est envoyée à un serveur autre que l'API Agnes AI**. Les clés et les images sont stockées localement dans le navigateur (`localStorage`/`sessionStorage`).
- **La photo de référence n'est jamais transmise à l'API de génération d'images** : elle sert uniquement à produire une description textuelle via le modèle de vision.
- **Les couleurs de la photo source sont ignorées** : c'est une contrainte volontaire pour garantir que chaque œuvre possède l'ambiance colorée de Schiele.
- **Le nombre d'œuvres est limité par le contexte du LLM** : le garde-fou des 300 œuvres (v3) permet de viser 400–500 générations sans dépasser les tokens.
- **Les résultats dépendent de la qualité de l'analyse** : si le modèle de vision échoue, utilisez le fallback manuel pour une description plus fiable.

---

## 🙏 Remerciements

- **Egon Schiele** (1890–1918) pour son œuvre expressionniste viennoise.
- **Agnes AI** pour l'accès aux modèles de vision, de texte et de génération d'images.
- **Laurent Voanh** pour la conception et le développement des labs.

---

**Bon portrait !** 🎨
