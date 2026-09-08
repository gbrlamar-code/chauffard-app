# Chauffard

Web app pour signaler les automobilistes dangereux en quelques taps.
Carte des signalements en direct, fiches « chauffards » pour les plaques signalées plusieurs fois.

Le signalement se fait en 5 étapes : **adresse de l'incident** (recherche
géocodée ou « Utiliser ma position ») + **date/heure** (défaut : maintenant) →
comportements (mots-clés) → plaque (pavé tactile) → type de véhicule → récap.
L'onglet **Carte** affiche toutes les épingles sur un fond OpenStreetMap.

## Fichiers

| Fichier | Rôle |
|---|---|
| `index.html` | L'application. Fichier unique, aucune dépendance à installer, aucune étape de build. Carte **Leaflet + OpenStreetMap**, vue par défaut Paris intra-muros. |
| `chauffard.html` | Variante conçue pour tourner comme *Artifact* Claude (carte schématique SVG, stockage `claude.use('db')`). Ne pas héberger en statique. |

## Lancer en local

```bash
python3 -m http.server 8000
# puis ouvrir http://localhost:8000
```

(N'importe quel serveur statique fait l'affaire : `npx serve`, extension Live Server, etc.
Ouvrir le fichier directement en `file://` fonctionne aussi mais `localStorage` peut être bridé.)

## Configuration

Tout se règle dans le bloc `CONFIGURATION` en haut de la balise `<script>` de `index.html`.

### Fond de carte — `TILE_URL`
Par défaut : tuiles OpenStreetMap standard (libres, sans clé, attribution incluse).
Correct pour un trafic modéré ; pour de la production, passer par un fournisseur dédié
(MapTiler, Stadia, Thunderforest…) ou héberger ses tuiles. Alternatives prêtes à coller
en commentaire dans le fichier.

### Vue par défaut — `HOME`
Centre, zoom, zoom minimum et limites de déplacement de la carte (Paris par défaut).

### Stockage partagé — `FIREBASE_CONFIG`
- **Vide (défaut)** : les signalements sont stockés dans le `localStorage` du navigateur —
  visibles sur cet appareil uniquement. Des exemples sont chargés au premier lancement
  (bouton « Réinitialiser » dans l'écran *À propos* pour les remettre).
- **Renseigné** : créer un projet [Firebase](https://console.firebase.google.com/),
  activer **Cloud Firestore**, coller la config web. Les signalements deviennent alors
  partagés entre tous les visiteurs. Règles Firestore minimales (démo) :

  ```
  match /reports/{d} { allow read: if true; allow create: if true; }
  match /plates/{d}  { allow read, write: if true; }
  ```

## Déploiement

Fichier statique unique : déposer le dépôt sur n'importe quel hébergeur statique
(GitHub Pages, Netlify, Vercel, Cloudflare Pages). `index.html` est servi à la racine.

## Limites connues (avant une vraie mise en service)

- L'inscription aux alertes e-mail est enregistrée mais **aucun e-mail n'est réellement
  envoyé** (il faut un service de messagerie côté serveur). En attendant, l'alerte est
  visuelle dans l'app.
- Pas de modération ni d'anti-spam, pas de procédure de contestation d'un signalement.
  Une plaque d'immatriculation est une **donnée personnelle** (RGPD).
- La politique d'usage d'OpenStreetMap impose un fournisseur de tuiles dédié dès que le
  trafic augmente.
