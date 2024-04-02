# play.infra
Play Economy Infrastructure Components

## Add the GitHub package source
```powershell
$owner="play-economy-microservices"
$gh_pat="[PAT HERE]"

dotnet nuget add source --username USERNAME --password $gh_pat --store-password-in-clear-text --
name github "https://nuget.pkg.github.com/$owner/index.json"
```