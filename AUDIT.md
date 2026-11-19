# AUDIT — Jarvis PWA v0.1 vs la vision

Date : 19 novembre 2026 · Code audité : `/root/dev/jarvis-pwa/` (commit 5f0c551) · En ligne : https://nidnorg.github.io/jarvis/

---

## Verdict en une phrase

La PWA est un **prototype de conversation**, pas Jarvis. Elle fait bien une chose (parler avec un cerveau au choix, avec une bulle animée et une validation OUI/NON), et il lui manque **tout le reste** : la progression, les compagnons, les agents, l'Habitat, la proactivité, et surtout la présence permanente par-dessus les autres applications.

Mesure objective : **1 048 lignes** au total (158 HTML, 251 CSS, 567 JS, 52 SW, 20 manifest).

---

## Ce qui existe réellement

| Fonction | État | Preuve |
|---|---|---|
| Chat français en streaming | ✅ | `streamChat()`, SSE, `MAX_CONTEXT=16` |
| Cerveau au choix (4 presets) | ✅ | `PRESETS` : OpenAI, Gemini, OpenRouter, Ollama |
| Mémoire de faits durable | ✅ | `K_FACTS`, réinjectée dans `buildContext()` |
| Mémoire de conversation | ✅ | `K_MSGS`, plafond 200 |
| Bulle flottante déplaçable | ✅ mais dans l'app seulement | `pointerdown/move/up` |
| Émotions du compagnon | ⚠️ **7 sur 8** | curious, idle, listening, sad, speaking, surprised, thinking — il manque « happy » |
| Validation OUI/NON | ✅ | `askConfirm()`, `runAction()`, protocole `<<<ACTION>>>` |
| Mode Live vocal | ⚠️ partiel | `startLive()` + Web Speech API, **uniquement app ouverte et au premier plan** |
| Installation Android | ✅ | manifest standalone + service worker |
| Hors-ligne | ⚠️ coquille seulement | le chat ne fonctionne pas sans réseau |

---

## Ce qui manque, par gravité

### 1. La présence permanente — le trou le plus grave
La bulle ne vit **que dans l'onglet de l'app**. Dès que tu ouvres WhatsApp ou Instagram, elle disparaît. Or ta vision dit « présence permanente, te voir faire, être là ». Une PWA ne peut pas dessiner par-dessus une autre application : c'est une limite du navigateur, pas un oubli.
**Conséquence :** sans application native Android, Jarvis ne sera jamais Jarvis. Il sera un site web avec un joli robot.

### 2. Aucune progression
`grep` : **0 occurrence** de « niveau », d'XP structurée, de compteur. Le compagnon est exactement le même au premier jour et au centième. Les 8 niveaux (Nouveau-né → Transcendant) n'existent que sur le papier. Aucune raison de revenir.

### 3. Un seul compagnon
Le robot est dessiné en SVG en dur. Pas de korrigan, pas d'alien, pas de chien, pas de renard. Pas de choix d'espèce, pas de personnalisation (6 styles, 10 couleurs de corps, 8 couleurs d'yeux : 0 % implémenté).

### 4. Zéro agent
`grep agent` : **0**. Pas de création d'agent, pas de rôle, pas de conversation entre eux, pas de panneau « Discussions agents », pas d'affinité. C'est la moitié de ta vision qui n'existe pas du tout.

### 5. Zéro Habitat
`grep habitat` : **0**. Pas d'espace visible, pas de décor jour/nuit, pas de cycle de vie, pas d'événements.

### 6. Zéro proactivité
`grep` : 0 occurrence de *mission*, *briefing*, *notification*, *wake*, *routine*, *cron*. Jarvis ne fait rien tout seul. Il attend que tu écrives. Pas de réveil, pas de briefing du matin, pas de « j'ai vu que… », pas de rappel. Le wake word n'existe pas : il faut ouvrir l'app et appuyer sur le micro.

### 7. La publication n'est pas branchée à l'app
Le moteur de publication existe (`/root/dev/jarvis-publisher/`, testé, publication réelle OK), mais il vit **à côté**, sur le serveur. Dans l'app, tu n'as aucun écran de publications, aucune file, aucun bouton OUI. Un utilisateur de l'app ne peut pas publier.

### 8. Sécurité : une promesse non tenue
La clé API est stockée **en clair** dans `localStorage`. Il n'y a ni coffre, ni chiffrement, ni rédaction des données sensibles, ni piste d'audit. `grep chiffr` : 1 occurrence, dans un texte décoratif.
Quiconque a accès au téléphone déverrouillé lit ta clé Gemini en trois taps.

### 9. Missions et routines
`grep mission` : 0. Aucune tâche récurrente, aucun « chaque lundi prépare le point ventes ».

### 10. Intégration Android
`grep` : 0 pour *widget*, *PiP*, *share*, *clipboard*. Pas de barre de partage, pas de widget, pas de Picture-in-Picture, pas de tuile rapide.

### 11. Onboarding
Aucun. Première ouverture : une page de chat vide et un bouton ⚙️. Pas d'explication, pas de choix du compagnon, pas de saisie guidée de la clé.

### 12. Robustesse
Pas de reprise d'erreur réseau, pas de file de messages en attente, pas de limite de débit, pas de gestion du quota. Un 429 d'OpenAI = un message d'erreur brut.

### 13. Ce qui n'a jamais été vérifié
- Le Mode Live sur **ton** téléphone (testé en code seulement)
- Le rendu de la bulle en conditions réelles (les captures d'écran ne montrent pas les éléments fixes)
- Le comportement du service worker après une mise à jour

---

## Le vrai diagnostic

J'ai livré en une soirée ce qui demanderait normalement trois semaines, et c'est pour ça que ça ressemble à une démo : **le socle est solide mais il n'y a aucune chair autour du squelette.**

Le problème n'est pas la quantité de travail, c'est **l'ordre**. J'ai construit le moteur de conversation avant la progression, les agents et l'Habitat — c'est-à-dire les choses qui donnent envie de revenir. Un assistant qui répond bien, il y en a cinquante. Un compagnon qui grandit, qui a des amis, qui vit dans son atelier quand tu dors : il n'y en a aucun.

Et il y a le plafond structurel : **aucune PWA ne donnera jamais la bulle par-dessus les autres applications.** Soit on l'accepte (Jarvis = app qu'on ouvre), soit on passe au natif.

---

## Trois chemins, à toi de trancher

**A. Finir la PWA au maximum de son plafond** — niveaux + XP, 4 compagnons, personnalisation, multi-agents visibles dans un panneau, Habitat en plein écran dans l'app, missions, briefing, notifications quand l'app est ouverte, publication intégrée, réglages sécurisés.
*Ce que tu obtiens :* un Jarvis complet **quand tu l'ouvres**. 4 à 6 sessions de travail.
*Ce que tu n'obtiens jamais :* la bulle par-dessus WhatsApp.

**B. Passer au natif Android** — même app, mais en Kotlin : vraie bulle système, wake word permanent, service en arrière-plan, widgets, PiP, notifications réelles.
*Ce que tu obtiens :* la vision entière.
*Le obstacle :* l'APK ne compile pas sur cette machine (AAPT2 cassé sous proot). Il faut un build dans le cloud. Je peux le faire — c'est faisable, ça demande un pipeline GitHub Actions et 20 à 40 minutes de calcul par version.

**C. L'ordre inverse** — arrêter d'ajouter des fonctions et construire d'abord **la boucle qui donne envie de revenir** : niveaux, XP, déblocages, un compagnon qui te reconnaît et se souvient de toi. Puis le reste.

---

## Ma recommandation

**C, puis B.** Parce que la progression et la personnalité sont ce qui rend Jarvis attachant, et que ça se code aussi bien en web qu'en natif — donc ça ne sera pas perdu au passage. Construire le natif d'abord serait reproduire l'erreur actuelle : une belle coquille sans âme.

Et une chose à dire clairement : **la PWA restera en ligne et utilisable** pendant tout ce temps. Tu ne perds rien, tu ajoutes.
