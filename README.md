# Musicify

**Musicify** est un lecteur de musique simple développé avec **Vue 3, Vite et JavaScript**. Les informations sur les chansons, les pochettes d’album et les extraits audio proviennent de l’API publique **[iTunes Search API](https://developer.apple.com/library/archive/documentation/AudioVideo/Conceptual/iTuneSearchAPI/)** (aucune clé API n’est nécessaire).

## Installation et lancement

```bash
npm install
npm run dev
```

Ouvrez ensuite l’adresse affichée dans le terminal (généralement [http://localhost:5173](http://localhost:5173)). Une connexion Internet est nécessaire.

Pour générer la version de production :

```bash
npm run build
```

Les fichiers générés seront disponibles dans le dossier `dist/`. Pour prévisualiser cette version :

```bash
npm run preview
```

## Fonctionnalités

- Lecture, pause, chanson suivante et précédente.
- Barre de progression avec affichage du temps écoulé et de la durée.
- Réglage du volume et fonction muet.
- Passage automatique à la chanson suivante lorsque la lecture se termine.
- Recherche de chansons par titre ou par artiste.
- Trois pages principales : Accueil, Découvrir et Titres favoris.
- Sauvegarde des chansons favorites dans `localStorage`.
- Interface responsive adaptée aux ordinateurs et aux appareils mobiles.

## Sélection des chansons

Le fichier `src/App.vue` contient une courte liste appelée `CATEGORIES`. Chaque catégorie correspond à une recherche iTunes basée sur un artiste ou un mot-clé.

Pour chaque recherche, l'application :

1. Recherche des chansons non explicites (`explicit=No`).
2. Conserve uniquement les chansons disposant d'un extrait audio (`previewUrl`) et d'une pochette.
3. Supprime les doublons.
4. Sélectionne les six premières chansons correspondantes.

Pour modifier les chansons proposées, il suffit de changer le paramètre `term` des catégories dans `src/App.vue`.

## Informations importantes

- Les fichiers audio sont des **extraits de 30 secondes** fournis par Apple, et non des chansons complètes. L'interface informe clairement l'utilisateur de cette limitation.
- Si un extrait audio ne peut pas être chargé, un message explicatif est affiché au lieu de laisser l'utilisateur sans indication.
- Si le navigateur bloque la lecture automatique, l'application invite l'utilisateur à lancer la lecture manuellement.
- L'application ne nécessite aucun compte utilisateur ni système d'authentification.

## Structure du projet

```text
src/
├── components/
│   ├── Sidebar.vue       # Navigation latérale
│   ├── SongCard.vue      # Carte représentant une chanson
│   └── MusicPlayer.vue   # Lecteur audio inférieur
│                         # Contient l'élément <audio>
├── App.vue               # Chargement des chansons,
│                         # recherche, favoris et pages
├── main.js               # Point d'entrée de l'application
└── style.css             # Styles de l'application
```
