# Première ressource serveur

Cet exemple ajoute une commande de diagnostic indépendante aux ressources existantes. Il repose sur le manifeste `open77.lua` et sur le global serveur `RegisterCommand`. Il sert à vérifier le chargement d’une ressource et la différence entre le contexte console et le contexte joueur.

## 1. Créer les fichiers

```text
resources/
└── op77_smokecheck/
    ├── open77.lua
    └── server/
        └── main.lua
```

`open77.lua`

```lua
resource "op77_smokecheck"
version "1.0.0"
auto_start true
server_script "server/main.lua"
```

`server/main.lua`

```lua
RegisterCommand("op77_smokecheck", function(source)
    local origin = source == 0 and "console" or ("joueur %d"):format(source)
    print(("[op77_smokecheck] Commande reçue depuis %s"):format(origin))
end, false)
```

## 2. Charger la ressource

Si ta configuration utilise une liste explicite dans `resources.load`, ajoute `"op77_smokecheck"` en conservant les entrées existantes. Suis la procédure de rechargement de ton build. La console serveur accepte `ensure op77_smokecheck` lorsque la ressource est sélectionnée dans la configuration. L’autodémarrage dépend aussi de `resources.autoStart`.

## 3. Vérifier son comportement

Depuis la console serveur, saisis `op77_smokecheck` et vérifie que la ligne contient `console`. Depuis un client connecté, utilise `/op77_smokecheck` si le chat est disponible. La ligne côté serveur doit alors indiquer l’identifiant de session du joueur. Après un rechargement, relance la commande et vérifie qu’elle n’a qu’un seul gestionnaire actif.

Pour la suite, lis le manifeste et le code client/serveur d’un exemple officiel avant d’ajouter une permission ou un événement réseau. La commande ci-dessus est un exemple écrit pour le vault. Sa syntaxe a été comparée aux contrats publiés et son exécution reste à vérifier sur ton serveur.

- [[Base Open2077/01 Documentation/server-resources|Ressources serveur]]
- [[Base Open2077/02 API Lua/Serveur/globals#RegisterCommand|Contrat de RegisterCommand]]
- [[Base Open2077/03 Exemples/Ressources officielles|Exemples officiels]]
