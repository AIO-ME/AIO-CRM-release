# AIO-CRM-release
📦 Releases officielles d'AIO-CRM - CRM pour thérapeutes holistiques. Contient les binaires (.dmg, .exe) et le manifest update.json pour les mises à jour automatiques.

## Skill CRM AIO pour Codex et Claude Code

Le skill `crm-aio-patient` permet à un agent local d'utiliser le CLI de CRM AIO 1.2.6 ou ultérieur pour lire un dossier ou ses séances. Le CLI reste livré avec l'application ; ce dépôt public ne contient pas le code privé du CRM.

Installer ou actualiser le skill depuis n'importe quel projet :

```sh
npx skills@1.7.4 add AIO-ME/AIO-CRM-release --skill crm-aio-patient
```

Pour tous les projets, en ciblant Codex et Claude Code :

```sh
npx skills@1.7.4 add AIO-ME/AIO-CRM-release --skill crm-aio-patient --agent codex claude-code --global
```

Le skill sait retrouver les emplacements du CRM sur macOS et Windows lors de la première demande, puis mémorise le chemin dans la configuration locale du CLI. Le chemin personnel et les données patient ne sont jamais écrits dans le skill partagé. Les interprétations IA et les contacts sont exclus par défaut. Une ambiguïté entre installations ou dossiers nécessite un choix de l'utilisateur.

Mettre à jour le skill ne demande pas de reconstruire ou réinstaller l'application. Les copies déjà embarquées dans les anciens bundles restent utilisables mais peuvent être moins récentes que cette version GitHub.
