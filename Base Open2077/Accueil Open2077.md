---
tags:
  - open2077
  - lua
---

# Open2077

Base de référence pour héberger et programmer un serveur OPEN//77. Capture des sources publiques du **24 novembre 2026**. Les synthèses sont en français et les contrats et exemples officiels gardent leur langue d’origine.

| Point d’entrée | Contenu |
| --- | --- |
| [[Base Open2077/01 Documentation/Index de la documentation\|Documentation]] | 189 notes de documentation, en incluant le guide d’accueil officiel, plus le lien vers l’API |
| [[Base Open2077/02 API Lua/Index API Lua\|API Lua complète]] | 1934 fonctions sur 154 pages, séparées entre serveur (1109) et client (825) |
| [[Base Open2077/03 Exemples/Ressources officielles\|Exemples officiels]] | 155 manifestes, scripts Lua, WebUI, gamemodes, bibliothèques et pack RP |
| [[Base Open2077/04 Devkit et données/Index Devkit et données\|Devkit et données de jeu]] | Événements, permissions, schémas, stubs, builds et catalogues JSON |
| [[Base Open2077/AGENTS\|Instructions pour ton IA]] | Méthode de recherche, vérification des contrats et tests |

## Démarrer ton serveur

1. Choisis le build et le canal dans [[Base Open2077/01 Documentation/host-a-server|Héberger un serveur]]. Sur le serveur dédié, configure `server.jsonc` et la licence. Utilise `OP77_LICENSE_KEY` pour garder la clé hors des fichiers partagés.
2. Consulte [[Base Open2077/01 Documentation/server-startup|Démarrage et logs]] et [[Base Open2077/01 Documentation/server-resources|Ressources serveur]] pour le lancement, la configuration et les dépendances.
3. Ajoute une [[Base Open2077/03 Exemples/Première ressource serveur|première ressource serveur]] sur une instance de test. Parcours ensuite [[Base Open2077/01 Documentation/writing-a-gamemode|Écrire un gamemode]] et le code officiel à partir d’un exemple compatible.

Pour un projet en cours, note le build de ton serveur et de ton client dans le projet. La documentation publique et l’index du Devkit peuvent évoluer à des rythmes différents. [Versions serveur officielles](https://open2077.net/host) · [Documentation en ligne](https://open2077.net/docs)
