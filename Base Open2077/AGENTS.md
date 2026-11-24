# Travailler sur des ressources OPEN//77 avec cette base

1. Confirme le build local (serveur, client et ressources) dans les fichiers ou les logs du projet. Commence par [l’accueil](Accueil%20Open2077.md), [l’API](02%20API%20Lua/Index%20API%20Lua.md) et [le Devkit](04%20Devkit%20et%20donn%C3%A9es/Index%20Devkit%20et%20donn%C3%A9es.md).
2. Cherche chaque appel par **runtime**. Ouvre la carte exacte et relève la signature, les arguments, le résultat, les erreurs, la permission et le build minimal lorsqu’ils sont disponibles. Confirme les événements, payloads et identifiants dans l’index officiel du Devkit.
3. Vérifie `open77.lua`, les dépendances, `resources.load` et la répartition client/serveur. Cherche une ressource officielle qui utilise le même système sous `03 Exemples/Code officiel/`. Confirme les alias FiveM et les fonctionnalités natives de jeu dans les contrats OPEN//77.
4. Garde les décisions de gameplay, l’identité, les permissions et la persistance sous autorité du serveur. Côté serveur, valide les données qui franchissent la limite réseau. Côté client, traite les opérations asynchrones et les états de démarrage ou d’arrêt.
5. Pour tout code produit, explicite les chemins, les permissions et les dépendances. Contrôle le manifeste et les signatures. Si tu as accès à une instance de test, vérifie le démarrage, le rechargement, les logs et les scénarios réseau concernés.
6. Dans ta réponse, cite les notes ou URL précises consultées, le build retenu, et distingue la validation statique du résultat observé en jeu.

Le référentiel de l’API contient 1934 entrées publiques. Le manifeste du Devkit archivé indique `2.50.0+op77.208`. Le Markdown officiel de chaque page reste accessible depuis sa note locale.
