# Guide de déploiement — WhatsApp Commercial AI

## 1. Prérequis
- Node.js 18+ installé sur votre machine
- Un compte Cloudflare (gratuit)
- Votre Google Sheet + script Apps Script déjà déployé en Web App (voir `apps-script/Code.gs`)

## 2. Configuration de la Google Sheet "Config_Admin"
Ajoutez ces colonnes (ligne d'en-tête) si elles n'existent pas déjà :

```
prix_pack_vip | delai_essai_jours | whatsapp_support | one_time_enabled |
price_vip_cfa | subscription_enabled | price_monthly | price_monthly_cfa |
price_quarterly | price_quarterly_cfa | price_annual | price_annual_cfa
```

Remplissez la ligne 2 avec vos valeurs (`one_time_enabled` et `subscription_enabled` acceptent `TRUE`/`FALSE`).

Redéployez votre Apps Script (`apps-script/Code.gs`) en tant que **nouvelle version** de Web App après l'avoir mis à jour, puis vérifiez que l'URL dans `src/services/api.ts` (`GAS_API_URL`) correspond bien à votre déploiement.

## 3. Clé API Gemini (retouche photo)
1. Récupérez une clé API gratuite sur [Google AI Studio](https://aistudio.google.com/).
2. Créez un fichier `.env` à la racine du projet (à ne jamais commiter) :
   ```
   VITE_GEMINI_API_KEY=votre_clé_ici
   ```
3. Sans clé, la retouche photo bascule automatiquement sur l'amélioration locale (Canvas) — l'app reste fonctionnelle.

⚠️ Le nom du modèle Gemini (`gemini-2.0-flash-exp` dans `src/services/gemini.ts`) et le format de réponse de l'API peuvent avoir changé depuis la rédaction de ce code. Testez avec votre clé et ajustez si nécessaire en consultant la documentation officielle à jour.

## 4. Icônes PWA
Des icônes placeholder (`public/icons/icon-192.png`, `icon-512.png`) ont été générées automatiquement. **Remplacez-les par votre vrai logo** (même nom de fichier, mêmes dimensions) avant la mise en production.

## 5. Build local
```bash
npm install
npm run build
```
Cela génère le dossier `dist/` — c'est ce dossier qu'on téléverse sur Cloudflare Pages.

## 6. Déploiement sur Cloudflare Pages (Direct Upload)
1. Connectez-vous sur [dash.cloudflare.com](https://dash.cloudflare.com) → **Workers & Pages** → **Create application** → **Pages** → **Upload assets**.
2. Donnez un nom à votre projet (ex. `whatsapp-commercial-ai`).
3. Glissez-déposez le **contenu du dossier `dist/`** (pas le dossier lui-même, son contenu).
4. Cliquez sur **Deploy site**.
5. Votre app est en ligne sur `https://votre-projet.pages.dev`.

## 7. Vérifications post-déploiement
- `https://votre-projet.pages.dev` → écran de connexion/inscription vendeur
- `https://votre-projet.pages.dev/?vendeur=212612345678` → catalogue public (sans invite PWA)
- `https://votre-projet.pages.dev/?admin=true` → portail Super Admin
- Depuis le tableau de bord vendeur, testez le bouton **"Copier mon lien"** et l'installation PWA (icône d'installation dans la barre d'adresse du navigateur, uniquement visible côté vendeur/admin).

## 8. Mises à jour futures
Pour chaque mise à jour : `npm run build` puis retéléversez le contenu de `dist/` via **Upload assets** sur le même projet Cloudflare Pages (ou connectez un dépôt Git pour un déploiement automatique).

---

### Récapitulatif de la logique métier VIP
- **Essai gratuit** : durée réglable (`trial_days`) depuis l'Admin. Passé ce délai, le catalogue vendeur passe automatiquement en mode bridé (4 produits max).
- **Option A — Achat unique** : prix fixe, activable/désactivable depuis l'Admin.
- **Option B — Abonnements** : Mensuel / Trimestriel / Annuel, prix paramétrables, activable/désactivable depuis l'Admin.
- Le vendeur choisit une offre dans la modale VIP → redirection WhatsApp vers le support avec l'offre pré-remplie.
- **Activation VIP manuelle** : depuis le Portail Admin, bouton "VIP" à côté de chaque vendeur (à utiliser une fois le paiement reçu).
