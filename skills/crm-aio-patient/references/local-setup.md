# Trouver l'installation locale du CRM

Cette étape ne demande ni dépôt de développement, ni nouveau bundle. Le skill est distribué sur GitHub ; le CLI natif reste livré avec CRM AIO 1.2.6 ou ultérieur.

## Règles communes

Utiliser le profil existant en premier. Chercher ensuite uniquement les exécutables et fichiers de configuration de l'installation CRM, à partir des emplacements ci-dessous et de leurs métadonnées. Recueillir les chemins existants, les résoudre et les dédupliquer avant de choisir.

Un seul chemin : valider et enregistrer ce profil avec le CLI. Plusieurs chemins : demander quelle installation utiliser, sans lire les dossiers de chaque base ni choisir automatiquement la plus récente. Aucun fichier : demander si le CRM a été ouvert après installation ; faire préciser son emplacement si nécessaire. Ne pas créer de base ni parcourir tout le disque, les sauvegardes ou les profils d'autres comptes.

## macOS

1. Chercher `crm-aio-cli` dans le PATH, puis dans `Contents/MacOS` de `CRM AIO.app` sous `/Applications` ou `$HOME/Applications`. Pour une autre installation, utiliser l'emplacement indiqué par l'utilisateur.
2. Lire `CFBundleIdentifier` dans `Contents/Info.plist` de l'application choisie. Le chemin SQLite de la distribution directe correspond à `$HOME/Library/Application Support/<CFBundleIdentifier>/thera.db`.

Exemple après sélection de l'application :

```sh
crm_app="/Applications/CRM AIO.app"
crm_cli="$crm_app/Contents/MacOS/crm-aio-cli"
crm_identifier=$(/usr/libexec/PlistBuddy -c 'Print :CFBundleIdentifier' "$crm_app/Contents/Info.plist")
crm_database="$HOME/Library/Application Support/$crm_identifier/thera.db"
test -f "$crm_database" && printf '%s\n' "$crm_database"
```

Ces commandes examinent des métadonnées, pas le contenu de SQLite. Un ancien conteneur App Store ou une copie de sauvegarde ne remplace pas automatiquement la base active de l'installation directe.

## Windows

Chercher d'abord `crm-aio-cli.exe` avec `Get-Command`. Pour la version Microsoft Store, lire les métadonnées du paquet CRM :

```powershell
$crmPackages = @(Get-AppxPackage -Name 'TonyLatree.CRMAIO')
$crmPackages | Select-Object Name, InstallLocation, PackageFamilyName
```

Le CLI se trouve à la racine de `InstallLocation`, à côté de `CRMAIO.exe`. Pour une installation directe, vérifier uniquement les emplacements CRM sous `$env:ProgramFiles`, `${env:ProgramFiles(x86)}` ou `$env:LOCALAPPDATA\Programs`, et l'`InstallLocation` de l'entrée **CRM AIO** du registre de désinstallation. Demander l'emplacement si ces métadonnées ne suffisent pas ; ne pas télécharger un exécutable trouvé au hasard.

Chemins de données candidats de l'utilisateur courant :

```powershell
$crmCandidates = @(
  (Join-Path $env:APPDATA 'TonyLatree.CRMAIO\thera.db'),
  (Join-Path $env:APPDATA 'com.crm-aio.app\thera.db')
)
foreach ($crmPackage in $crmPackages) {
  $crmPackageData = Join-Path $env:LOCALAPPDATA ('Packages\' + $crmPackage.PackageFamilyName)
  $crmCandidates += Join-Path $crmPackageData 'LocalCache\Roaming\TonyLatree.CRMAIO\thera.db'
  $crmCandidates += Join-Path $crmPackageData 'LocalState\TonyLatree.CRMAIO\thera.db'
  $crmCandidates += Join-Path $crmPackageData 'RoamingState\TonyLatree.CRMAIO\thera.db'
}
$crmDatabases = @($crmCandidates | Where-Object { Test-Path -LiteralPath $_ -PathType Leaf } |
  ForEach-Object { (Get-Item -LiteralPath $_).FullName } | Sort-Object -Unique)
$crmDatabases
```

La version Store peut virtualiser AppData. Si aucun chemin standard n'existe, limiter une éventuelle recherche de `thera.db` au dossier exact `Packages\<PackageFamilyName>` obtenu ci-dessus, avec profondeur bornée. Exclure les répertoires de restauration, sauvegardes et exports. Une installation ou un ancien profil restant ambigu nécessite le choix de l'utilisateur ; aucune recherche globale dans AppData ou les autres disques.

## Enregistrer et réutiliser

Sur le fichier unique retenu, utiliser le chemin complet du CLI et `configure --db`, puis relancer la commande de lecture sans `--db`. Le CLI valide l'existence et le schéma avant d'enregistrer son profil, sans migration.

La configuration est locale à l'utilisateur : `~/Library/Application Support/crm-aio-cli/config.json` sur macOS, `%APPDATA%\crm-aio-cli\config.json` sur Windows. Respecter `CRMAIO_CLI_CONFIG` lorsqu'il est défini. Ne pas écrire ce chemin dans `SKILL.md`, une mémoire partagée ou GitHub. Deux appels successifs doivent réutiliser le même profil, pas répéter la recherche.

Dans un environnement agent restreint, SQLite peut avoir besoin d'accéder aux fichiers de coordination WAL/SHM à côté de la base. Une erreur de lecture ne prouve donc pas à elle seule un schéma incompatible : vérifier les permissions et, si nécessaire, demander l'exécution autorisée de la même commande CLI en lecture seule, sans copier la base ou modifier ses permissions globales.

Si la base est incompatible, faire ouvrir le CRM compatible pour sa migration normale. Si elle est inaccessible, signaler le problème de permissions. Garder les interprétations IA et les contacts exclus par défaut pendant la première lecture comme lors des suivantes.
