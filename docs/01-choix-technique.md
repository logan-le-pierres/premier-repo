# Choix technique

## Le contexte qui décide de tout

- **Catalogue minuscule** : 10 à 30 couteaux en ligne, presque tous des **pièces uniques** (stock = 1).
- **Le coutelier doit être autonome** pour ajouter un couteau (3 photos, un prix, un texte) depuis son téléphone, à l'atelier.
- **Clientèle de 50 ans et plus** : peu de pages, un parcours d'achat très court, le téléphone mis en avant.
- **Un développeur TypeScript** (toi) pour construire et maintenir.
- **Budget mensuel faible** : petit artisan, souvent micro-entrepreneur.

## Comparatif

| | PrestaShop | WordPress + WooCommerce | Shopify | Sur mesure TypeScript |
|---|---|---|---|---|
| Adapté à 20 pièces uniques | Non, surdimensionné | Oui | Oui | Oui |
| Autonomie du coutelier | Moyenne (back-office complexe) | Bonne | Excellente | Bonne avec un back-office (Payload) |
| Maintenance / sécurité | Lourde (PHP, modules) | Lourde (mises à jour d'extensions, cible n°1 des piratages) | Aucune | Faible (peu de dépendances, pas de plugins tiers) |
| Liberté de design épuré | Faible | Moyenne (thèmes chargés) | Moyenne | Totale |
| Coût mensuel | Hébergement ~10 € | Hébergement ~10 € + extensions | 36 € + frais | 0 à 15 € |
| Commission paiement | Stripe ~1,5 % + 0,25 € | idem | Shopify Payments ~1,8 % + 0,25 € | Stripe ~1,5 % + 0,25 € |

**PrestaShop est à écarter** : il est fait pour des catalogues de centaines de références avec déclinaisons, pas pour 20 pièces uniques.

**WordPress/WooCommerce suffirait**, mais tu passerais ton temps à combattre un thème et à mettre à jour des extensions. Ce n'est pas le bon outil pour un développeur.

**Shopify** est la seule vraie alternative à envisager **si personne ne peut assurer la maintenance dans le temps**. C'est la question à se poser honnêtement : le jour où tu n'es plus disponible, le coutelier doit-il pouvoir continuer seul ?

## Recommandation : sur mesure, en TypeScript

Comme tu es développeur et que le besoin est très simple, le sur mesure est le meilleur rapport qualité / effort ici.

```
Next.js 15 (App Router, TypeScript)
 ├─ Site public          → pages statiques/ISR, ultra rapides
 ├─ Payload CMS 3        → back-office /admin intégré dans le même projet Next.js
 │                         (couteaux, photos, pages, commandes)
 ├─ Stripe Checkout      → page de paiement hébergée par Stripe
 │   └─ Webhook          → marque le couteau « vendu », enregistre la commande, envoie les e-mails
 ├─ PostgreSQL           → base de Payload
 └─ E-mails              → Brevo (français) ou Resend
```

**Pourquoi Payload CMS** : il est écrit en TypeScript, s'installe *dans* l'application Next.js (un seul projet, un seul déploiement) et génère automatiquement un back-office propre à partir de tes schémas. Tu décris une collection `couteaux` en TS, le coutelier obtient un formulaire simple pour l'ajouter. Il est open source (MIT), sans abonnement.

Alternative plus légère si tu ne veux pas de base de données : **Astro** + catalogue géré directement dans **Stripe** (produits, photos, prix). Mais le tableau de bord Stripe est moins agréable pour le coutelier qu'un back-office dédié.

### Stripe : utiliser Checkout, pas un formulaire maison

- **Stripe Checkout** (page hébergée) gère la carte, Apple Pay / Google Pay, l'authentification forte 3D Secure (obligatoire, DSP2), la collecte de l'adresse et du téléphone, et l'e-mail de reçu.
- Ta conformité PCI-DSS se limite au questionnaire le plus simple (**SAQ A**), car aucune donnée de carte ne transite par ton serveur.
- `consent_collection.terms_of_service` permet d'imposer l'acceptation des CGV, `custom_fields` peut porter la case « J'ai 18 ans ou plus ».
- `submit_type: 'pay'` affiche le bouton « Payer », formulation non ambiguë acceptable pour l'obligation du bouton de commande (article L221-14 du Code de la consommation).

### Gestion des pièces uniques (le seul point délicat)

1. Au clic sur « Continuer vers le paiement » : créer la session Checkout avec `expires_at` = +30 min, et passer le couteau en statut `réservé` (avec la date d'expiration).
2. Webhook `checkout.session.completed` → statut `vendu`, création de la commande, e-mails au client et au coutelier.
3. Webhook `checkout.session.expired` → retour au statut `disponible`.
4. Vérifier la signature des webhooks et rendre le traitement **idempotent** (Stripe peut renvoyer un événement plusieurs fois).

### Hébergement

Rester **dans l'Union européenne** simplifie le RGPD : Scaleway, OVHcloud ou Clever Cloud (français), ou Vercel avec une base Neon en région UE. Compter 0 à 15 €/mois pour ce volume. Nom de domaine en `.fr` : ~10 €/an.

### Accessibilité pour un public de 50 ans et plus

- Texte de base à 18 px minimum, contraste AA au minimum, polices sans fantaisie.
- Un seul bouton principal par écran, avec un libellé explicite (« Ajouter au panier », pas une icône seule).
- Pas de carrousel, pas de fenêtre surgissante, pas de compte client obligatoire (achat en invité).
- Numéro de téléphone visible partout : une partie des clients voudra appeler avant d'acheter, et la commande par téléphone (paiement par chèque ou virement) doit rester possible.
- Zones cliquables d'au moins 48 px de haut.

### Estimation

| Lot | Durée indicative |
|---|---|
| Mise en place Next.js + Payload + design | 4 à 6 jours |
| Catalogue, fiche, panier | 3 à 4 jours |
| Stripe Checkout, webhooks, e-mails, réservation | 3 à 4 jours |
| Pages légales, rétractation, cookies | 1 à 2 jours |
| Formation du coutelier, mise en ligne | 1 jour |
