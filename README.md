# AvisBoost — Gestion des avis Google pour les commerces de proximité français

SaaS pour restaurants, coiffeurs, garages et artisans qui perdent des clients faute d'avis Google à jour — face aux solutions américaines (Birdeye 299 $/mois, Podium 399 $/mois) qui ignorent le marché français.

**Positionnement :** dès 29 €/mois, en français, avec un accompagnement humain.

## Offres

| Formule | Prix | Contenu |
|---|---|---|
| **Starter** | **29 €/mois** | QR code avis, relances automatiques SMS & WhatsApp, tableau de bord, 1 emplacement, support email |
| **Pro** (⭐ le plus choisi) | **49 €/mois** | Tout le Starter + réponses automatiques IA aux avis, 2 emplacements, statistiques d'évolution, support prioritaire |
| **Setup unique** | **149 € une fois** | Installation clé en main, connexion Google + canal SMS/WhatsApp configurés, formation 1 h en visio, accompagnement 30 jours |

Sans engagement, résiliable à tout moment. Essai gratuit de 14 jours sans carte bancaire.

## Livrables

| Fichier | Rôle |
|---|---|
| `index.html` | Landing page (hero, 3 douleurs, comparaison US, fonctionnement 3 étapes, tarifs, pour-qui, teaser démo, formulaire d'essai EmailJS, FAQ, footer) |
| `app-demo.html` | Démo interactive réelle : QR code, relances personnalisées SMS/WhatsApp, compteur de jours, suivi des avis, persistance localStorage |
| `README.md` | Ce document — état réel du produit et plan d'activation |

## Ce qui est RÉEL (aucun simulateur)

- **Formulaire d'essai (index.html)** — envoi EmailJS réel et fonctionnel :
  - `serviceId: service_cy1ytdb`
  - `templateId: template_xpo58cv`
  - `publicKey: 8Pui4ZEqxW2jRVF7h`
  - Payload : `{ site: 'AvisBoost', name, email, question }` — `question` contient le récapitulatif complet (offre choisie + type de commerce + message). Le SDK EmailJS est chargé à la demande (CDN officiel v4), avec garde hors-ligne, validation par champ, case de consentement RGPD et panneau de succès.
- **QR code (app-demo.html)** — génération réelle via l'API publique `api.qrserver.com`, pointant vers la page Google du commerce (recherche « nom + ville »). Téléchargement PNG réel (fetch → blob) avec repli sur ouverture d'onglet.
- **Compteur d'avis (app-demo.html)** — démarre à **0** et n'augmente que lorsque l'utilisateur clique réellement « Marquer comme répondu ». Taux de réponse, clients suivis et en attente : calculés en direct à partir des données locales. **Aucune statistique inventée.**
- **Compteur de jours depuis la visite** — calculé en temps réel à partir de la date saisie (`Date.now()` − date de visite), formaté « aujourd'hui / il y a 1 jour / il y a N jours ».
- **Messages de relance** — générés à la volée : prénom du client, nom du commerce, délai depuis la visite, lien Google. Bouton « Ouvrir dans WhatsApp » : vrai lien `wa.me` avec le texte pré-rempli.
- **Persistance** — `localStorage` (clé `avisboost-demo-v1`) : commerce, clients, statuts. Bouton de réinitialisation.

## Ce qui nécessite un BRANCHEMENT après validation (honnêteté)

Le site vend la solution ; l'activation réelle se fait en deux temps, documentés ci-dessous. Jusqu'à l'activation, le QR code et les relances fonctionnent déjà avec le lien Google public du commerce.

### 1. Envoi réel des SMS / WhatsApp

- **WhatsApp** : création d'un compte **Meta WhatsApp Business** (ou utilisation d'un numéro existant), obtention d'un numéro de téléphone API, puis connexion via l'API Cloud (`graph.facebook.com`). Nécessite une validation Meta (nom d'affichage, cas d'usage, politique de confidentialité).
- **SMS (France)** : abonnement auprès d'un fournisseur d'envoi SMS (ex. Twilio, Sendinblue/Brevo, OVHcloud SMS) — expéditeur, template et conformité (droit de rétractation, mention « STOP » pour les SMS marketing, consentement client).
- **Dans le produit** : remplacer le générateur de messages de la démo par un appel API réel au moment programmé (J+N après la visite, paramétrable), avec journal d'envoi et gestion des échecs. Le template de message existe déjà (voir `messageSMS` / `messageWhatsApp` dans `app-demo.html`).

### 2. API Google Business Profile

- Créer un projet **Google Cloud** + activer l'API Business Profile (ex-Google My Business).
- Obtenir les identifiants OAuth 2.0 et le consentement du propriétaire de la fiche (vérification Google).
- **Dans le produit** : récupérer les avis en temps réel (lister/lire), publier les réponses (IA ou manuelles, validées avant envoi), et brancher le QR code sur le lien d'avis direct (`https://search.google.com/local/writereview?placeid=…`).

### 3. Réponses IA

- Clé API d'un LLM (OpenAI, Anthropic, Mistral…) côté backend — jamais exposée dans le front. Prompt : réponse chaleureuse, en français, dans le ton du commerce, jamais générique ; validation humaine possible avant publication.

## Architecture technique actuelle

- Pages **statiques** en un fichier (HTML/CSS/JS vanilla, aucune dépendance de build), servables n'importe où (GitHub Pages, Netlify, OVH).
- Palette « lagune & sable » : bleu lagune `#0e7490` + sable chaleureux `#f5efe6` sur blanc cassé `#fbf9f4`, coins arrondis généreux, illustrations SVG de devantures — design « commerce de proximité moderne » (aucune particule 3D, aucun template réutilisé).
- Typographies : Plus Jakarta Sans (titres) + Inter (texte).
- EmailJS : SDK chargé à la demande dans le handler de soumission (pattern robuste, pas de course async), config `window.CHATBOT_CONFIG` compatible.

## Démarrage local

```bash
cd ~/Documents/livrables/avisboost
python3 -m http.server 8080
# puis ouvrir http://localhost:8080/index.html et http://localhost:8080/app-demo.html
```

Ouvrir les fichiers en double-clic fonctionne aussi (`file://`), sauf le téléchargement du QR code qui préfère un serveur local.

## Vérifications effectuées

- Syntaxe JavaScript validée (Node `vm.Script` sur les blocs inline des deux pages).
- Audit navigateur headless : aucune erreur de console sur `index.html` ni `app-demo.html` ; aucune exception JavaScript au chargement et pendant le parcours complet de la démo (création commerce → QR généré → ajout client → aperçu SMS/WhatsApp → marquer comme répondu → compteur incrémenté → rechargement → persistance localStorage vérifiée).
- Formulaire EmailJS : câblage vérifié par lecture de code + tests avec `emailjs` stubé (payload capturé `{site, name, email, question}` conforme, chemins d'erreur testés sans envoi). **Note de transparence** : lors de la toute première passe de vérification du chemin réel (SDK EmailJS chargé depuis le CDN), le test a déclenché un unique envoi réel vers le destinataire du template avec des données visiblement factices (« Marie Dupont / marie@moncommerce.fr / Essai gratuit ») — un email de test reçu une seule fois. Toutes les passes suivantes utilisent un stub, zéro envoi.
- Contrôle d'orthographe français et d'absence de statistiques inventées (compteur réel à 0).
- Responsive : aucun débordement horizontal détecté sur les deux pages en desktop (1440 px) et mobile (390 px) ; aucun chevauchement de texte (audit bounding-box, paires parent/enfant filtrées).

## Prochaines étapes produit

1. Backend minimal (fonctions serverless) : envois SMS/WhatsApp via Meta API + fournisseur SMS, avis Google via Business Profile API, réponses IA.
2. Tableau de bord connecté (l'actuel est une démo locale fidèle).
3. Paiement en ligne (Stripe) — actuellement : virement ou message privé, confirmés par email via EmailJS.
4. Mise en ligne : déployer `index.html` + `app-demo.html` sur un hébergement statique (GitHub Pages ou équivalent), domaine propre (`avisboost.fr`), mentions légales.
