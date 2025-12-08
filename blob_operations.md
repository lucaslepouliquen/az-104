# Opérations sur les Blobs Azure - Guide Complet

**Concept Clé pour AZ-104 : Les opérations sur les blobs permettent de manipuler les données dans Azure Blob Storage via des API REST, CLI, PowerShell ou SDKs. Comprendre ces opérations est essentiel pour gérer efficacement le stockage et optimiser les performances.**

---

## Vue d'ensemble des Opérations

**Types d'opérations disponibles :**

- **Opérations de création** : Put Blob, Put Block, Put Block List
- **Opérations de lecture** : Get Blob, Get Blob Properties, Get Blob Metadata
- **Opérations de modification** : Set Blob Metadata, Set Blob Properties, Copy Blob
- **Opérations de suppression** : Delete Blob
- **Opérations avancées** : Snapshot Blob, Lease Blob

---

## 1. Put Blob - Création et Mise à Jour

### Définition

**Put Blob** crée un nouveau blob (block, page, ou append) ou met à jour le contenu d'un block blob existant. Lors de la mise à jour d'un block blob existant, **TOUTES les métadonnées existantes sont écrasées**.

### Caractéristiques clés

- **Taille maximale** : 5000 MiB (environ 5.24 GB) par opération Put Blob
- **Opération atomique** : Le blob est créé ou mis à jour en une seule transaction
- **Écrasement complet** : Les métadonnées existantes sont remplacées (pas fusionnées)
- **Support des conditions** : If-Match, If-None-Match, If-Modified-Since, If-Unmodified-Since
- **Optimisation réseau** : Compression et encodage supportés

### Commandes Azure CLI

```bash
# Upload d'un fichier simple
az storage blob upload \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myblob.txt \
  --file ./local-file.txt \
  --content-type "text/plain"

# Upload avec métadonnées
az storage blob upload \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name document.pdf \
  --file ./document.pdf \
  --metadata project=azure department=it

# Upload avec tier spécifique
az storage blob upload \
  --account-name mystorageaccount \
  --container-name archive \
  --name backup.zip \
  --file ./backup.zip \
  --tier Cool
```

### Commandes PowerShell

```powershell
# Upload simple
$ctx = New-AzStorageContext -StorageAccountName "mystorageaccount" -StorageAccountKey "xxxx"
Set-AzStorageBlobContent `
  -File "./local-file.txt" `
  -Container "mycontainer" `
  -Blob "myblob.txt" `
  -Context $ctx

# Upload avec métadonnées et propriétés
$metadata = @{"project"="azure"; "department"="it"}
Set-AzStorageBlobContent `
  -File "./document.pdf" `
  -Container "mycontainer" `
  -Blob "document.pdf" `
  -Context $ctx `
  -Metadata $metadata `
  -Properties @{ContentType="application/pdf"; CacheControl="max-age=3600"}
```

### Use Cases Put Blob

**Use Case 1 : Upload de fichiers utilisateurs**

```
Scénario :
- Application web permettant l'upload de photos
- Fichiers < 5 GB
- Métadonnées : user_id, upload_date, category

Solution :
# Upload avec métadonnées utilisateur
az storage blob upload \
  --account-name photostorage \
  --container-name user-photos \
  --name user123/photo_$(date +%Y%m%d_%H%M%S).jpg \
  --file ./photo.jpg \
  --metadata user_id=123 upload_date=$(date +%Y-%m-%d) category=vacation
```

**Use Case 2 : Mise à jour de configuration**

```
Scénario :
- Fichier de configuration application (config.json)
- Mise à jour fréquente
- Besoin d'atomicité (pas de lecture pendant mise à jour)

Solution :
# Put Blob écrase atomiquement le blob existant
az storage blob upload \
  --account-name configstorage \
  --container-name app-config \
  --name config.json \
  --file ./new-config.json \
  --overwrite
```

### ⚠️ Limitations et Considérations

- **5 GB maximum** : Pour fichiers > 5 GB, utiliser Put Block + Put Block List
- **Écrasement des métadonnées** : Spécifier TOUTES les métadonnées à chaque mise à jour
- **Pas de reprise** : Si échec, recommencer l'opération complète
- **Network** : Optimiser avec compression côté client si possible

---

## 2. Put Block & Put Block List - Upload de Gros Fichiers

### Concept

Pour les fichiers > 5 GB ou nécessitant reprise en cas d'échec, Azure utilise un système de blocs :

1. **Put Block** : Upload des blocs individuels (chunks)
2. **Put Block List** : Commit des blocs pour créer le blob final

### Architecture Block Blob

Un block blob peut contenir jusqu'à **50,000 blocs** de **4000 MiB** chacun, soit **~190.7 TiB** total.

### Workflow Put Block

```
┌─────────────────────────────────────────────┐
│  Fichier local : video.mp4 (10 GB)         │
└─────────────────┬───────────────────────────┘
                  │ Découpage en blocs
                  ▼
        ┌─────────────────────┐
        │  Block ID: AAA      │ (100 MB)
        │  Block ID: AAB      │ (100 MB)
        │  Block ID: AAC      │ (100 MB)
        │       ...           │
        │  Block ID: ZZZ      │ (100 MB)
        └──────────┬──────────┘
                   │ Put Block (chaque bloc)
                   ▼
        ┌──────────────────────┐
        │  Azure Blob Storage  │
        │  Uncommitted Blocks  │
        └──────────┬───────────┘
                   │ Put Block List (commit)
                   ▼
        ┌──────────────────────┐
        │  Blob Final Committed│
        │  video.mp4 (10 GB)   │
        └──────────────────────┘
```

### Caractéristiques Put Block

- **Taille par bloc** : Jusqu'à 4000 MiB (environ 4.19 GB)
- **Block ID** : Chaîne base64 unique (max 64 caractères)
- **État** : Uncommitted jusqu'au Put Block List
- **Durée de vie** : 7 jours pour blocs uncommitted (puis suppression auto)
- **Parallélisme** : Blocs uploadables en parallèle

### Exemple Azure CLI avec AzCopy (Recommandé)

```bash
# AzCopy gère automatiquement Put Block + Put Block List
azcopy copy './large-file.zip' \
  'https://mystorageaccount.blob.core.windows.net/mycontainer/large-file.zip?<SAS>' \
  --block-size-mb 100 \
  --blob-type BlockBlob

# Upload avec reprise automatique en cas d'échec
azcopy copy './huge-video.mp4' \
  'https://mystorageaccount.blob.core.windows.net/videos/huge-video.mp4?<SAS>' \
  --block-size-mb 100 \
  --blob-type BlockBlob \
  --check-length=false
```

### Exemple PowerShell (Bas niveau)

```powershell
# Upload par blocs manuel
$ctx = New-AzStorageContext -StorageAccountName "mystorageaccount" -StorageAccountKey "xxxx"
$file = Get-Item "./large-file.zip"
$blockSize = 100MB
$blockIds = @()

# Lire et uploader chaque bloc
$stream = [System.IO.File]::OpenRead($file.FullName)
for ($i = 0; $i -lt [Math]::Ceiling($file.Length / $blockSize); $i++) {
    $blockId = [Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes($i.ToString("d6")))
    $blockIds += $blockId
    
    $buffer = New-Object byte[] $blockSize
    $bytesRead = $stream.Read($buffer, 0, $blockSize)
    
    # Put Block
    Set-AzStorageBlobContent `
        -Container "mycontainer" `
        -Blob "large-file.zip" `
        -Context $ctx `
        -BlockId $blockId `
        -BlobContent $buffer[0..($bytesRead-1)]
}
$stream.Close()

# Put Block List (commit)
Set-AzStorageBlobContent `
    -Container "mycontainer" `
    -Blob "large-file.zip" `
    -Context $ctx `
    -BlockList $blockIds
```

### ⚠️ Comportement Important

**Ce qui est stocké :**
- Blocs dans état **uncommitted** pendant 7 jours
- Après Put Block List : blob devient **committed**
- Blocs uncommitted automatiquement supprimés après 7 jours

**Optimisations :**
- Upload parallèle de blocs (jusqu'à 32 simultanés)
- Reprise automatique des blocs échoués
- Pas besoin de re-uploader blocs réussis

### Use Cases Put Block + Put Block List

**Use Case 1 : Upload de fichiers volumineux avec reprise**

```
Scénario :
- Upload de fichiers vidéo 4K (50 GB)
- Connexion instable
- Besoin de reprendre l'upload en cas d'échec

Solution :
# AzCopy avec reprise automatique
azcopy copy './video-4k.mp4' \
  'https://videostorage.blob.core.windows.net/videos/video-4k.mp4?<SAS>' \
  --block-size-mb 100 \
  --blob-type BlockBlob

Avantages :
Reprise automatique des blocs échoués
Upload parallèle (jusqu'à 32 blocs simultanés)
Optimisation réseau automatique
```

**Use Case 2 : Upload distribué (multiple sources)**

```
Scénario :
- Fichier de backup distribué (parties sur plusieurs serveurs)
- Assemblage dans Azure

Solution :
1. Serveur 1 : Put Block (ID: 000001, 000002, 000003)
2. Serveur 2 : Put Block (ID: 000004, 000005, 000006)
3. Serveur 3 : Put Block (ID: 000007, 000008, 000009)
4. Coordination : Put Block List avec ordre [000001-000009]

Résultat : Blob final assemblé dans Azure
```

### Best Practices Put Block

- **Taille de bloc optimale** : 100-256 MB (compromis performance/reprise)
- **Block IDs uniques** : Utiliser convention de nommage claire (ex: base64 encoded indexes)
- **Gestion 7 jours** : Commit ou cleanup des uncommitted blocks
- **Parallélisme** : 8-32 threads pour uploads rapides
- **Monitoring** : Tracker progression avec compteur de blocs
- **Idempotence** : Blocs avec même ID écrasent les précédents (permet retry)

---

## 3. Copy Blob - Copie et Migration

### Définition

**Copy Blob** copie un blob vers une destination au sein du même storage account ou entre storage accounts. L'opération peut être **synchrone** (blobs < 256 MB) ou **asynchrone** (blobs > 256 MB).

### Types de Copie

| Type | Taille Blob | Comportement | Temps |
|------|-------------|--------------|-------|
| **Synchrone** | < 256 MB | Opération bloquante | Immédiat |
| **Asynchrone** | > 256 MB | Arrière-plan, polling requis | Minutes à heures |

### Caractéristiques Copy Blob

- **Source** : Blob dans le même account ou account différent
- **Destination** : Nouveau nom ou écrasement
- **Métadonnées** : Préservées par défaut ou spécifiées
- **Blob tiers** : Peut changer pendant la copie
- **Snapshots** : Source peut être un snapshot
- **Cross-region** : Supporté entre régions Azure
- **Server-side** : Copie effectuée par Azure (pas de bandwidth client)

### Commandes Azure CLI

```bash
# Copie simple dans le même container
az storage blob copy start \
  --account-name mystorageaccount \
  --destination-blob backup-file.txt \
  --destination-container backups \
  --source-uri 'https://mystorageaccount.blob.core.windows.net/source/file.txt'

# Copie entre storage accounts
az storage blob copy start \
  --account-name deststorage \
  --destination-blob migrated-data.zip \
  --destination-container data \
  --source-uri 'https://sourcestorage.blob.core.windows.net/data/data.zip?<SAS>'

# Copie avec changement de tier
az storage blob copy start \
  --account-name mystorageaccount \
  --destination-blob archived-file.zip \
  --destination-container archive \
  --source-uri 'https://mystorageaccount.blob.core.windows.net/hot/file.zip' \
  --tier Archive

# Vérifier le statut de copie asynchrone
az storage blob show \
  --account-name mystorageaccount \
  --container-name backups \
  --name backup-file.txt \
  --query "properties.copy.status"
```

### Commandes PowerShell

```powershell
# Copie simple
$srcCtx = New-AzStorageContext -StorageAccountName "sourcestorage" -StorageAccountKey "xxxx"
$destCtx = New-AzStorageContext -StorageAccountName "deststorage" -StorageAccountKey "yyyy"

Start-AzStorageBlobCopy `
  -SrcContainer "source" `
  -SrcBlob "file.txt" `
  -Context $srcCtx `
  -DestContainer "destination" `
  -DestBlob "file-copy.txt" `
  -DestContext $destCtx

# Monitoring de copie asynchrone
$blob = Get-AzStorageBlobCopyState `
  -Container "destination" `
  -Blob "large-file.zip" `
  -Context $destCtx

while ($blob.Status -eq "Pending") {
    Start-Sleep -Seconds 10
    $blob = Get-AzStorageBlobCopyState `
        -Container "destination" `
        -Blob "large-file.zip" `
        -Context $destCtx
    Write-Host "Progress: $($blob.BytesCopied) / $($blob.TotalBytes)"
}
Write-Host "Copy complete: $($blob.Status)"
```

### Statuts de Copie

- **Pending** : Copie en cours
- **Success** : Copie terminée avec succès
- **Failed** : Échec de la copie
- **Aborted** : Copie annulée manuellement

### Use Cases Copy Blob

**Use Case 1 : Backup et Versioning**

```
Scénario :
- Créer backup avant modification d'un blob
- Préserver version précédente

Solution :
# Copier le blob avant modification
az storage blob copy start \
  --account-name mystorageaccount \
  --destination-blob important-file-backup-$(date +%Y%m%d).txt \
  --destination-container backups \
  --source-uri 'https://mystorageaccount.blob.core.windows.net/production/important-file.txt'

# Modifier l'original
az storage blob upload \
  --account-name mystorageaccount \
  --container-name production \
  --name important-file.txt \
  --file ./modified-file.txt \
  --overwrite
```

**Use Case 2 : Migration entre régions**

```
Scénario :
- Migrer données de West Europe vers North Europe
- Minimiser downtime
- 100 GB de données

Solution :
# Copie asynchrone cross-region
az storage blob copy start-batch \
  --source-account-name westeuropestorage \
  --source-container data \
  --destination-account-name northeuropestorage \
  --destination-container data \
  --pattern '*'

Avantages :
Copie serveur-side (pas de bandwidth client)
Asynchrone (application continue de fonctionner)
Pas d'egress charges Azure interne (selon régions)
```

**Use Case 3 : Réhydratation Archive avec copie**

```
Scénario :
- Blob en Archive tier
- Besoin d'accès sans modifier l'original

Solution :
# Copier avec réhydratation vers Hot
az storage blob copy start \
  --account-name mystorageaccount \
  --destination-blob rehydrated-file.zip \
  --destination-container hot-data \
  --source-uri 'https://mystorageaccount.blob.core.windows.net/archive/archived-file.zip' \
  --tier Hot \
  --rehydrate-priority High

Résultat :
Original reste en Archive (économies)
Copie disponible en Hot (accès rapide)
Pas besoin de modifier l'original
```

### Erreurs Courantes

- **SAS Token manquant** : Nécessaire pour copie entre accounts différents
- **Permissions insuffisantes** : Source doit avoir permission de lecture
- **Timeout** : Copie asynchrone peut prendre heures pour gros blobs
- **Abort** : Utiliser `az storage blob copy cancel` pour annuler copie en cours

### Points Clés pour l'Examen

- Copy Blob est **serveur-side** (Azure fait la copie, pas le client)
- **< 256 MB** : Synchrone (immédiat)
- **> 256 MB** : Asynchrone (polling requis)
- **SAS requis** pour cross-account copy
- Peut copier **entre régions** (cross-region)
- **Métadonnées préservées** par défaut

---

## 4. Set Blob Metadata - Gestion des Métadonnées

### Définition

**Set Blob Metadata** définit les métadonnées définies par l'utilisateur pour un blob spécifié sous forme de paires nom-valeur. Cette opération **ÉCRASE toutes les métadonnées existantes**.

### Caractéristiques Métadonnées

- **Format** : Paires clé-valeur (key=value)
- **Limite** : 8 KB total pour toutes les métadonnées
- **Nommage** : Noms doivent suivre convention C# identifiers
- **Case-insensitive** : Clés converties en lowercase par Azure
- **Caractères** : Alphanumériques uniquement (ASCII)
- **Stockage** : Stockées avec le blob, pas de coût supplémentaire
- **Accès** : Récupérables via Get Blob Properties/Metadata

### Règles de Nommage

**Valide** :
- `author`, `department`, `project_name`
- `CreatedBy`, `LastModified`, `Version123`

**Invalide** :
- `user-name` (tiret non autorisé)
- `étoile` (caractères non-ASCII)
- `2024year` (commence par chiffre)

### Commandes Azure CLI

```bash
# Définir métadonnées
az storage blob metadata update \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name document.pdf \
  --metadata author="John Doe" department=IT project=Azure classification=confidential

# Lire métadonnées
az storage blob metadata show \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name document.pdf

# Output exemple :
# {
#   "author": "john doe",
#   "classification": "confidential",
#   "department": "it",
#   "project": "azure"
# }
```

### Commandes PowerShell

```powershell
# Définir métadonnées
$ctx = New-AzStorageContext -StorageAccountName "mystorageaccount" -StorageAccountKey "xxxx"
$metadata = @{
    "author" = "John Doe"
    "department" = "IT"
    "project" = "Azure"
    "classification" = "confidential"
}

Set-AzStorageBlobMetadata `
  -Container "mycontainer" `
  -Blob "document.pdf" `
  -Context $ctx `
  -Metadata $metadata

# Lire métadonnées
$blob = Get-AzStorageBlob -Container "mycontainer" -Blob "document.pdf" -Context $ctx
$blob.ICloudBlob.Metadata

# Ajouter une métadonnée (sans écraser les autres)
$blob.ICloudBlob.FetchAttributes()  # Charger métadonnées existantes
$blob.ICloudBlob.Metadata.Add("new_key", "new_value")
$blob.ICloudBlob.SetMetadata()  # Sauvegarder
```

### Use Cases Set Blob Metadata

**Use Case 1 : Gestion documentaire**

```
Scénario :
- Bibliothèque de documents d'entreprise
- Recherche par auteur, département, projet
- Classification de sécurité

Solution :
# Ajouter métadonnées riches
az storage blob metadata update \
  --account-name docstorage \
  --container-name corporate-docs \
  --name Q3-Report.pdf \
  --metadata \
    author="Jane Smith" \
    department="Finance" \
    quarter="Q3" \
    year="2024" \
    classification="internal" \
    reviewed="true" \
    approver="John Manager"

# Recherche via Azure Cognitive Search ou custom indexing
# Filtrer par metadata.department = "Finance"
# Filtrer par metadata.quarter = "Q3"
```

**Use Case 2 : Workflow processing**

```
Scénario :
- Pipeline de traitement d'images
- Tracker statut : uploaded, processing, processed, error
- Identifier worker et timestamp

Solution :
# Image uploadée
az storage blob upload ... --metadata status=uploaded upload_time=$(date -u +%s)

# Worker commence traitement
az storage blob metadata update \
  --metadata status=processing worker_id=worker-3 start_time=$(date -u +%Y-%m-%dT%H:%M:%SZ)

# Traitement terminé
az storage blob metadata update \
  --metadata status=processed worker_id=worker-3 end_time=$(date -u +%Y-%m-%dT%H:%M:%SZ) duration_seconds=45

# Query blobs par statut pour monitoring
az storage blob list --include m --query "[?metadata.status=='processing']"
```

**Use Case 3 : Cache invalidation**

```
Scénario :
- CDN cache de fichiers statiques
- Besoin d'invalider cache lors de mise à jour

Solution :
# Ajouter version metadata
az storage blob metadata update \
  --account-name cdnstorage \
  --container-name static-assets \
  --name styles.css \
  --metadata version=$(date +%Y%m%d%H%M%S) last_modified=$(date -u +%Y-%m-%dT%H:%M:%SZ)

# CDN peut vérifier metadata.version pour cache invalidation
# Application inclut ?v={version} dans URL
# https://cdn.example.com/styles.css?v=20241208143022
```

**Use Case 4 : Compliance et Audit**

```
Scénario :
- Documents légaux avec traçabilité
- Audit trail requis
- Rétention policy

Solution :
az storage blob metadata update \
  --account-name legalstorage \
  --container-name contracts \
  --name contract-2024-001.pdf \
  --metadata \
    document_type="contract" \
    created_by="legal@company.com" \
    created_date="2024-12-08" \
    retention_years="7" \
    confidentiality="high" \
    client_id="CLIENT-12345" \
    matter_number="M-2024-001"
```

### ⚠️ Best Practices Métadonnées

**À FAIRE :**
- Convention de nommage cohérente (snake_case ou camelCase)
- Limiter à métadonnées essentielles (<8KB total)
- Utiliser pour indexation et recherche
- Stocker identifiants et références (user_id, project_id)
- Timestamps en format ISO 8601

**À ÉVITER :**
- Ne jamais stocker PII sensibles (SSN, passwords, credit cards)
- Ne pas stocker données volumineuses (utiliser blob content)
- Éviter caractères spéciaux et non-ASCII
- Ne pas oublier que Set Metadata ÉCRASE tout

### Différence avec Blob Properties

| Aspect | Metadata | Properties |
|--------|----------|------------|
| **Définition** | User-defined key-value | System properties |
| **Exemples** | author, department, project | ContentType, ContentLength, ETag |
| **Modification** | Set Blob Metadata | Set Blob Properties |
| **Limite** | 8 KB | Pas de limite |
| **Écrasement** | OUI, écrase tout | Non, modifie propriété ciblée |

---

## 5. Get Blob - Lecture et Téléchargement

### Définition

**Get Blob** lit ou télécharge un blob depuis Azure Storage, incluant ses métadonnées et propriétés. Supporte la **lecture partielle** (range requests) pour optimisation.

### Types de Lecture

- **Full Download** : Télécharger le blob complet
- **Range Request** : Télécharger une plage spécifique d'octets (bytes=0-1023)
- **Streaming** : Lire le blob en flux continu
- **Conditional** : Télécharger seulement si conditions remplies (If-Modified-Since, ETag)
- **Properties/Metadata Only** : Récupérer info sans télécharger contenu

### Commandes Azure CLI

```bash
# Télécharger blob complet
az storage blob download \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name document.pdf \
  --file ./downloaded-document.pdf

# Télécharger plusieurs blobs (batch)
az storage blob download-batch \
  --source mycontainer \
  --destination ./downloads \
  --account-name mystorageaccount \
  --pattern "*.pdf"

# Lire propriétés et métadonnées (sans télécharger contenu)
az storage blob show \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name document.pdf

# Output inclut :
# - properties.contentLength
# - properties.contentType
# - properties.etag
# - properties.lastModified
# - metadata {}
```

### Commandes PowerShell

```powershell
# Télécharger blob
$ctx = New-AzStorageContext -StorageAccountName "mystorageaccount" -StorageAccountKey "xxxx"
Get-AzStorageBlobContent `
  -Container "mycontainer" `
  -Blob "document.pdf" `
  -Destination "./downloaded-document.pdf" `
  -Context $ctx

# Streaming vers variable (pour fichiers petits)
$blob = Get-AzStorageBlob -Container "mycontainer" -Blob "config.json" -Context $ctx
$stream = New-Object System.IO.MemoryStream
$blob.ICloudBlob.DownloadToStream($stream)
$content = [System.Text.Encoding]::UTF8.GetString($stream.ToArray())
Write-Host $content

# Télécharger avec condition (si modifié après date)
Get-AzStorageBlobContent `
  -Container "mycontainer" `
  -Blob "data.csv" `
  -Destination "./data.csv" `
  -Context $ctx `
  -ModifiedSince (Get-Date).AddDays(-7)
```

### Range Requests (Lecture Partielle)

```bash
# Via cURL avec REST API
curl -H "x-ms-version: 2021-06-08" \
  -H "x-ms-range: bytes=0-1023" \
  -H "Authorization: Bearer <token>" \
  "https://mystorageaccount.blob.core.windows.net/mycontainer/largefile.bin" \
  -o first-1kb.bin

# Lire les 10 premiers MB d'un fichier
curl -H "x-ms-range: bytes=0-10485759" ... # 10MB = 10*1024*1024-1
```

**Use case Range Requests :**
- Lire header de fichier vidéo (premiers 10 MB) sans télécharger vidéo complète (10 GB)
- Reprendre download interrompu (bytes=5242880-10485759 pour continuer à 5 MB)
- Streaming adaptatif (demander segments selon bandwidth)

### Use Cases Get Blob

**Use Case 1 : Streaming vidéo adaptatif**

```
Scénario :
- Vidéo 4K (10 GB)
- Streaming vers client web/mobile
- Adapter qualité selon bandwidth client

Solution :
# Client demande ranges spécifiques via HTTP Range header
GET /videos/4k-video.mp4
Range: bytes=0-10485759  # Premiers 10 MB (header + début)

# Serveur répond avec 206 Partial Content
Content-Range: bytes 0-10485759/10737418240
Content-Length: 10485760

# Client continue avec ranges suivants
Range: bytes=10485760-20971519  # Segment suivant

Avantages Range Requests :
Pas besoin de télécharger vidéo complète
Seek dans vidéo (sauter vers position spécifique)
Économie bandwidth considérable
Meilleure UX (start playback immédiat)
```

**Use Case 2 : Conditional download (optimisation cache)**

```
Scénario :
- Application avec cache local de fichiers
- Vérifier si blob modifié avant re-téléchargement
- Économiser bandwidth

Solution :
# Première fois : télécharger et noter ETag + Last-Modified
az storage blob download \
  --account-name mystorageaccount \
  --container-name configs \
  --name app-config.json \
  --file ./cache/config.json

ETAG=$(az storage blob show ... --query "properties.etag" -o tsv)
LAST_MODIFIED=$(az storage blob show ... --query "properties.lastModified" -o tsv)

# Sauvegarder ETag localement
echo "$ETAG" > ./cache/config.json.etag

# Prochaine fois : vérifier si modifié
CURRENT_ETAG=$(az storage blob show ... --query "properties.etag" -o tsv)

if [ "$ETAG" == "$CURRENT_ETAG" ]; then
    echo "Using cached version"
    # Utiliser ./cache/config.json
else
    echo "Downloading updated version"
    az storage blob download ... --file ./cache/config.json
    echo "$CURRENT_ETAG" > ./cache/config.json.etag
fi
```

**Use Case 3 : Téléchargement parallèle de gros fichier**

```
Scénario :
- Fichier backup volumineux (100 GB)
- Connexion haute vitesse (1 Gbps)
- Maximiser throughput

Solution : Télécharger en parallèle avec AzCopy
# AzCopy utilise automatiquement range requests parallèles
azcopy copy \
  'https://mystorageaccount.blob.core.windows.net/backups/huge-backup.zip?<SAS>' \
  './huge-backup.zip' \
  --check-md5 FailIfDifferent

# AzCopy découpe automatiquement :
# - Thread 1 : bytes=0-104857599 (100 MB)
# - Thread 2 : bytes=104857600-209715199 (100 MB)
# - Thread 3 : bytes=209715200-314572799 (100 MB)
# - ...
# - Assemblage final

Résultat : Vitesse maximale (utilise pleinement la bande passante)
```

### Optimisations Get Blob

**Performance :**
- **Parallélisme** : AzCopy télécharge 8-32 ranges en parallèle
- **Range requests** : Télécharger en chunks pour gros fichiers
- **Compression** : Activer côté client si blob compressible (gzip)
- **CDN** : Utiliser Azure CDN pour contenu fréquemment lu (images, CSS, JS)

**Caching :**
- **ETag validation** : Vérifier ETag avant re-téléchargement
- **If-Modified-Since** : Condition HTTP pour download conditionnel
- **Cache-Control** : Headers HTTP pour contrôle cache navigateur/CDN

**Network :**
- **Retry policy** : Implémenter retry avec exponential backoff
- **Timeout** : Configurer timeout approprié (long pour gros blobs)
- **Bandwidth throttling** : Limiter vitesse si nécessaire (--cap-mbps avec AzCopy)

---

## 6. Delete Blob - Suppression

### Définition

**Delete Blob** supprime un blob du storage account. Si **Soft Delete** est activé, le blob est conservé pendant la période de rétention configurée.

### Types de Suppression

| Type | Description | Réversible | Durée |
|------|-------------|------------|-------|
| **Hard Delete** | Suppression définitive | Non | Immédiat |
| **Soft Delete** | Suppression réversible | Oui | 7-365 jours |

### Options de Suppression

- **Delete blob only** : Supprimer le blob base uniquement
- **Delete snapshots only** : Supprimer uniquement les snapshots
- **Delete blob + snapshots** : Supprimer blob et tous snapshots (`--delete-snapshots include`)

### Commandes Azure CLI

```bash
# Suppression simple
az storage blob delete \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name file-to-delete.txt

# Suppression avec snapshots
az storage blob delete \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name file-with-snapshots.txt \
  --delete-snapshots include

# Suppression conditionnelle (seulement si non modifié depuis)
az storage blob delete \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name important-file.txt \
  --if-match "<ETag>"

# Suppression batch (tous les .log dans container)
az storage blob delete-batch \
  --source logs \
  --account-name mystorageaccount \
  --pattern "*.log"
```

### Commandes PowerShell

```powershell
# Suppression simple
$ctx = New-AzStorageContext -StorageAccountName "mystorageaccount" -StorageAccountKey "xxxx"
Remove-AzStorageBlob `
  -Container "mycontainer" `
  -Blob "file-to-delete.txt" `
  -Context $ctx

# Suppression batch par critères (logs > 30 jours)
Get-AzStorageBlob -Container "logs" -Context $ctx | Where-Object {
    $_.Name -like "*.log" -and $_.LastModified -lt (Get-Date).AddDays(-30)
} | Remove-AzStorageBlob

# Suppression avec confirmation
Get-AzStorageBlob -Container "mycontainer" -Blob "critical-file.txt" -Context $ctx | 
  Remove-AzStorageBlob -Confirm
```

### Soft Delete - Récupération

```bash
# Si Soft Delete activé, restaurer blob supprimé
az storage blob undelete \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name deleted-file.txt

# Lister blobs supprimés (soft deleted)
az storage blob list \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --include d \
  --query "[?properties.deletedTime!=null].{Name:name, DeletedTime:properties.deletedTime}"
```

### Use Cases Delete Blob

**Use Case 1 : Nettoyage automatique de logs**

```
Scénario :
- Logs applicatifs anciens (>90 jours)
- Nettoyage quotidien automatisé
- Économiser coûts de stockage

Solution Option 1 : Lifecycle Management (Recommandé)
# Règle de lifecycle pour suppression automatique
{
  "rules": [{
    "name": "DeleteOldLogs",
    "enabled": true,
    "type": "Lifecycle",
    "definition": {
      "filters": {
        "blobTypes": ["blockBlob"],
        "prefixMatch": ["logs/"]
      },
      "actions": {
        "baseBlob": {
          "delete": {
            "daysAfterModificationGreaterThan": 90
          }
        }
      }
    }
  }]
}

Solution Option 2 : Script manuel/cron
# Supprimer logs > 90 jours
az storage blob delete-batch \
  --source logs \
  --account-name mystorageaccount \
  --if-unmodified-since $(date -d '90 days ago' +%Y-%m-%d)

Avantages Lifecycle :
Automatique (pas de script à maintenir)
Natif Azure (pas de compute/scheduling)
Gratuit
```

**Use Case 2 : Suppression sécurisée avec validation**

```
Scénario :
- Suppression de fichiers sensibles (données confidentielles)
- Vérifier que personne n'a modifié le fichier avant suppression
- Éviter race condition

Solution :
# Lire ETag actuel
ETAG=$(az storage blob show \
  --account-name mystorageaccount \
  --container-name sensitive \
  --name confidential.pdf \
  --query "properties.etag" -o tsv)

# Supprimer seulement si ETag inchangé
az storage blob delete \
  --account-name mystorageaccount \
  --container-name sensitive \
  --name confidential.pdf \
  --if-match "$ETAG"

# Si ETag changé (quelqu'un a modifié) : erreur 412 Precondition Failed
```

**Use Case 3 : Nettoyage de données temporaires**

```
Scénario :
- Fichiers temporaires de traitement (uploads, conversions)
- Nettoyer après traitement réussi
- Libérer espace

Solution :
# Application crée fichiers temp dans container
az storage blob upload --container-name temp ...

# Après traitement réussi
if process_success; then
    # Supprimer fichiers temp
    az storage blob delete --container-name temp --name temp-file-12345.bin
fi

# Filet de sécurité : Lifecycle rule pour cleanup
# Au cas où application crash avant cleanup manuel
{
  "rules": [{
    "name": "CleanupTemp",
    "definition": {
      "filters": {"prefixMatch": ["temp/"]},
      "actions": {
        "baseBlob": {
          "delete": {"daysAfterModificationGreaterThan": 1}
        }
      }
    }
  }]
}
```

### Considérations Delete Blob

**Sécurité :**
- **Soft Delete** : TOUJOURS activer pour protection contre suppressions accidentelles
- **RBAC** : Limiter permissions Delete à utilisateurs autorisés
- **Conditions** : Utiliser If-Match/ETag pour éviter race conditions
- **Audit** : Logger toutes suppressions pour compliance

**Snapshots :**
- Par défaut : Erreur si blob a snapshots
- `--delete-snapshots only` : Supprimer snapshots, garder blob
- `--delete-snapshots include` : Supprimer blob + tous snapshots

**Performance :**
- **Batch** : Utiliser delete-batch pour supprimer en masse (plus efficace)
- **Async** : Suppression est généralement très rapide (millisecondes)

---

## 7. Snapshot Blob - Versioning et Backup

### Définition

**Snapshot Blob** crée une copie **read-only** d'un blob à un instant T. Les snapshots sont **incrémentaux** (stockent uniquement les différences par rapport au blob base).

### Caractéristiques Snapshots

- **Read-only** : Snapshots ne peuvent jamais être modifiés
- **Incrémental** : Seulement les pages/blocs modifiés sont stockés
- **Timestamp** : Identifié par date/heure de création (ISO 8601)
- **Métadonnées** : Peuvent avoir métadonnées distinctes du blob base
- **Coût** : Facturé uniquement pour pages/blocs uniques (différences)
- **Hiérarchie** : Snapshot lié au blob base (si base supprimé, snapshots persistent)

### Commandes Azure CLI

```bash
# Créer snapshot
az storage blob snapshot \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name important-file.txt

# Output : snapshot timestamp
# "2024-12-08T10:30:45.1234567Z"

# Lister tous snapshots d'un blob
az storage blob list \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --include s \
  --prefix important-file.txt \
  --query "[?snapshot!=null].{Name:name, Snapshot:snapshot}"

# Télécharger snapshot spécifique
az storage blob download \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name important-file.txt \
  --snapshot "2024-12-08T10:30:45.1234567Z" \
  --file ./snapshot-recovery.txt

# Supprimer un snapshot spécifique
az storage blob delete \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name important-file.txt \
  --snapshot "2024-12-08T10:30:45.1234567Z"
```

### Commandes PowerShell

```powershell
# Créer snapshot
$ctx = New-AzStorageContext -StorageAccountName "mystorageaccount" -StorageAccountKey "xxxx"
$blob = Get-AzStorageBlob -Container "mycontainer" -Blob "important-file.txt" -Context $ctx
$snapshot = $blob.ICloudBlob.CreateSnapshot()
Write-Host "Snapshot created: $($snapshot.SnapshotTime)"

# Lister tous snapshots
$snapshots = Get-AzStorageBlob -Container "mycontainer" -Context $ctx | 
  Where-Object { $_.ICloudBlob.IsSnapshot }

# Restaurer depuis snapshot (copier snapshot vers blob base)
$latestSnapshot = $snapshots | 
  Where-Object { $_.Name -eq "important-file.txt" } |
  Sort-Object SnapshotTime -Descending | 
  Select-Object -First 1

# Copier snapshot vers blob base (écrase blob actuel)
Start-AzStorageBlobCopy `
  -SrcContainer "mycontainer" `
  -SrcBlob "important-file.txt" `
  -SrcBlobSnapshotTime $latestSnapshot.SnapshotTime `
  -DestContainer "mycontainer" `
  -DestBlob "important-file.txt" `
  -Context $ctx `
  -Force
```

### Architecture Snapshots Incrémentaux

```
┌──────────────────────────────────────────────────────┐
│  Blob Base (version actuelle)                        │
│  ┌────────┬────────┬────────┬────────┬────────┐      │
│  │ Block1 │ Block2 │ Block3 │ Block4 │ Block5 │      │
│  │ (V3)   │ (V1)   │ (V3)   │ (V2)   │ (V3)   │      │
│  └────────┴────────┴────────┴────────┴────────┘      │
└──────────────────────────────────────────────────────┘
         │          │          │          │          │
         │          │          │          │          │
         ▼          │          ▼          │          ▼
┌─────────────┐     │    ┌─────────┐     │    ┌─────────┐
│  Snapshot 3 │     │    │  Snap 3 │     │    │  Snap 3 │
│  Block1(V3) │     │    │  Block3 │     │    │  Block5 │
│  (unique)   │     │    │  (V3)   │     │    │  (V3)   │
└─────────────┘     │    └─────────┘     │    └─────────┘
                    │                    │
                    ▼                    ▼
              ┌─────────┐          ┌─────────┐
              │  Snap 1 │          │  Snap 2 │
              │  Block2 │          │  Block4 │
              │  (V1)   │          │  (V2)   │
              └─────────┘          └─────────┘

Coût : Seulement les blocs uniques sont facturés
- Snapshot 1 : Block2 (V1) uniquement
- Snapshot 2 : Block4 (V2) uniquement  
- Snapshot 3 : Block1 (V3) + Block3 (V3) + Block5 (V3)
- Blob Base : Pointe vers versions actuelles
```

### Use Cases Snapshot Blob

**Use Case 1 : Backup avant modification critique**

```
Scénario :
- Modification de configuration application
- Besoin de rollback rapide si problème
- Minimiser downtime

Solution :
# Créer snapshot avant modification
az storage blob snapshot \
  --account-name appstorage \
  --container-name config \
  --name app-config.json

SNAPSHOT_TIME=$(az storage blob list \
  --container-name config \
  --include s \
  --prefix app-config.json \
  --query "[?snapshot!=null] | [-1].snapshot" \
  -o tsv)

echo "Snapshot créé : $SNAPSHOT_TIME"

# Modifier la config
az storage blob upload \
  --account-name appstorage \
  --container-name config \
  --name app-config.json \
  --file ./new-config.json \
  --overwrite

# Si problème détecté : restaurer depuis snapshot
if error_detected; then
    echo "Rollback vers snapshot..."
    az storage blob download \
      --container-name config \
      --name app-config.json \
      --snapshot "$SNAPSHOT_TIME" \
      --file ./rollback-config.json
    
    az storage blob upload \
      --container-name config \
      --name app-config.json \
      --file ./rollback-config.json \
      --overwrite
fi
```

**Use Case 2 : Versioning manuel avec rétention policy**

```
Scénario :
- Document partagé édité par plusieurs utilisateurs
- Garder historique des 30 dernières versions
- Audit trail

Solution :
# Créer snapshot à chaque sauvegarde
az storage blob snapshot \
  --account-name docstorage \
  --container-name shared-docs \
  --name project-plan.docx \
  --metadata editor="alice@company.com" version="v$(date +%Y%m%d%H%M)" reason="Updated milestones"

# Script de nettoyage quotidien (garder 30 derniers snapshots)
#!/bin/bash
SNAPSHOTS=$(az storage blob list \
  --container-name shared-docs \
  --include s \
  --prefix project-plan.docx \
  --query "[?snapshot!=null].snapshot" \
  -o tsv | sort -r)

# Garder les 30 premiers, supprimer le reste
echo "$SNAPSHOTS" | tail -n +31 | while read snapshot; do
    az storage blob delete \
      --container-name shared-docs \
      --name project-plan.docx \
      --snapshot "$snapshot"
done
```

**Use Case 3 : Testing et Development**

```
Scénario :
- Base de données de test
- Snapshot avant tests destructifs
- Restauration rapide après tests

Solution :
# Créer snapshot de DB export avant tests
az storage blob snapshot \
  --container-name test-data \
  --name testdb.bacpac

# Exécuter tests (peuvent modifier/corrompre données)
run_integration_tests

# Restaurer depuis snapshot pour prochain run
SNAPSHOT=$(az storage blob list \
  --container-name test-data \
  --include s \
  --prefix testdb.bacpac \
  --query "[-1].snapshot" -o tsv)

az storage blob copy start \
  --source-container test-data \
  --source-blob testdb.bacpac \
  --source-snapshot "$SNAPSHOT" \
  --destination-container test-data \
  --destination-blob testdb.bacpac
```

### ⚠️ Best Practices Snapshots

**Rétention :**
- Implémenter politique de nettoyage (lifecycle ou script)
- Définir nombre max de snapshots (ex: 30 derniers)
- Supprimer snapshots après période (ex: 90 jours)

**Coût :**
- Monitorer coût incrémental (pages/blocs uniques)
- Snapshots fréquents avec peu de changements = coût faible
- Blob base modifié complètement = snapshot coûte ~100% du blob

**Métadonnées :**
- Ajouter métadonnées descriptives (éditeur, raison, version)
- Timestamp dans metadata pour tri/recherche

**Alternative moderne :**
- Considérer **Blob Versioning** (2024) pour versioning automatique
- Blob Versioning vs Snapshots :
  - Versioning : Automatique à chaque modification
  - Snapshots : Manuel, contrôle précis du timing

---

## 8. Lease Blob - Gestion de Concurrence

### Définition

**Lease Blob** établit un **lock exclusif** sur un blob pour gérer l'accès concurrent. Seul le détenteur du lease (avec lease ID) peut modifier ou supprimer le blob.

### Concept de Lease

Un lease est comme un "jeton" qui donne l'accès exclusif en écriture :
- Lecture : Toujours possible (lease ne bloque pas lecture)
- Écriture/Suppression : Nécessite le lease ID
- Durée : Infinie ou fixe (15-60 secondes)

### Types de Lease

| Type | Durée | Use Case |
|------|-------|----------|
| **Infinite Lease** | Illimitée | Lock permanent jusqu'à release explicite |
| **Fixed Duration** | 15-60 secondes | Lock temporaire avec auto-expiration |

### Opérations Lease

| Opération | Description |
|-----------|-------------|
| **Acquire** | Obtenir un lease sur un blob (génère lease ID) |
| **Renew** | Renouveler un lease existant (prolonger durée) |
| **Change** | Changer le lease ID (transférer propriété) |
| **Release** | Libérer un lease (volontairement) |
| **Break** | Forcer la rupture d'un lease (cas exceptionnel) |

### Commandes Azure CLI

```bash
# Acquérir lease infini
LEASE_ID=$(az storage blob lease acquire \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --blob-name critical-file.txt \
  --lease-duration -1 \
  --query "id" -o tsv)

echo "Lease ID: $LEASE_ID"
# Output : "9c6e8d8b-7f5a-4e2c-9b1a-3d4e5f6a7b8c"

# Modifier blob avec lease (nécessite lease ID)
az storage blob upload \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name critical-file.txt \
  --file ./updated-file.txt \
  --lease-id "$LEASE_ID" \
  --overwrite

# Tenter de modifier sans lease ID : ERREUR 412
# "There is currently a lease on the blob and no lease ID was specified"

# Renouveler lease (fixed duration)
az storage blob lease renew \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --blob-name critical-file.txt \
  --lease-id "$LEASE_ID"

# Libérer lease
az storage blob lease release \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --blob-name critical-file.txt \
  --lease-id "$LEASE_ID"

# Forcer rupture de lease (cas exceptionnel)
az storage blob lease break \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --blob-name critical-file.txt \
  --lease-break-period 0
```

### Commandes PowerShell

```powershell
# Acquérir lease infini
$ctx = New-AzStorageContext -StorageAccountName "mystorageaccount" -StorageAccountKey "xxxx"
$blob = Get-AzStorageBlob -Container "mycontainer" -Blob "critical-file.txt" -Context $ctx
$leaseId = $blob.ICloudBlob.AcquireLease(
    [TimeSpan]::FromSeconds(-1), # -1 = Infini
    $null, # Lease ID auto-généré
    $null, # Condition
    $null, # Options
    $null  # Context
)

Write-Host "Lease acquired: $leaseId"

# Opération avec lease
$condition = New-Object Microsoft.WindowsAzure.Storage.AccessCondition
$condition.LeaseId = $leaseId

Set-AzStorageBlobContent `
  -File "./updated-file.txt" `
  -Container "mycontainer" `
  -Blob "critical-file.txt" `
  -Context $ctx `
  -AccessCondition $condition `
  -Force

# Release lease
$blob.ICloudBlob.ReleaseLease($condition)
```

### Use Cases Lease Blob

**Use Case 1 : Distributed lock pour job processing**

```
Scénario :
- 5 workers traitent des jobs depuis une queue
- Jobs représentés par blobs JSON dans container "jobs"
- Un seul worker doit traiter chaque job (éviter duplication)

Solution : Utiliser lease comme distributed lock
#!/bin/bash
# Worker script

while true; do
    # Lister jobs disponibles
    JOBS=$(az storage blob list --container-name jobs --query "[].name" -o tsv)
    
    for JOB in $JOBS; do
        # Tenter d'acquérir lease (60 secondes)
        LEASE_ID=$(az storage blob lease acquire \
          --container-name jobs \
          --blob-name "$JOB" \
          --lease-duration 60 \
          --query "id" -o tsv 2>/dev/null)
        
        if [ -n "$LEASE_ID" ]; then
            echo "Worker $(hostname) acquired job: $JOB"
            
            # Télécharger job data
            az storage blob download \
              --container-name jobs \
              --name "$JOB" \
              --file "./job.json" \
              --lease-id "$LEASE_ID"
            
            # Traiter job (peut prendre 30-50 secondes)
            process_job "./job.json"
            
            # Supprimer job après traitement réussi
            az storage blob delete \
              --container-name jobs \
              --name "$JOB" \
              --lease-id "$LEASE_ID"
            
            echo "Job $JOB completed"
            break
        else
            # Lease déjà pris par autre worker
            echo "Job $JOB already being processed by another worker"
        fi
    done
    
    sleep 5
done

Avantages :
Pas de duplication de traitement
Auto-healing : Si worker crash, lease expire après 60s
Simple : Pas besoin de Redis/Zookeeper
```

**Use Case 2 : Maintenance exclusive sur fichier partagé**

```
Scénario :
- Fichier de configuration partagé par 20 applications
- Opération de maintenance nécessitant accès exclusif (compaction, réindexation)
- Autres applications doivent attendre

Solution :
# Application maintenance acquiert lease infini
LEASE_ID=$(az storage blob lease acquire \
  --container-name configs \
  --blob-name shared-index.db \
  --lease-duration -1 \
  --query "id" -o tsv)

# Maintenance (peut prendre plusieurs minutes)
echo "Starting maintenance with lease $LEASE_ID"

# Télécharger avec lease
az storage blob download \
  --container-name configs \
  --name shared-index.db \
  --file ./index.db \
  --lease-id "$LEASE_ID"

# Traitement (compaction, vacuum, réindexation)
sqlite3 ./index.db "VACUUM;"
sqlite3 ./index.db "REINDEX;"

# Upload avec lease
az storage blob upload \
  --container-name configs \
  --name shared-index.db \
  --file ./index.db \
  --lease-id "$LEASE_ID" \
  --overwrite

# Libérer lease
az storage blob lease release \
  --container-name configs \
  --blob-name shared-index.db \
  --lease-id "$LEASE_ID"

echo "Maintenance complete, lease released"

# Pendant maintenance, autres applications :
# - Peuvent LIRE (lecture non bloquée)
# - Ne peuvent PAS ÉCRIRE (erreur 412)
```

**Use Case 3 : Leader election (simple)**

```
Scénario :
- Cluster de 3 nodes
- Un seul node doit être leader (exécuter scheduled tasks)
- Leader election simple

Solution :
#!/bin/bash
# Node startup script

LEADER_BLOB="cluster-leader.lock"

while true; do
    # Tenter de devenir leader
    LEASE_ID=$(az storage blob lease acquire \
      --container-name cluster-state \
      --blob-name "$LEADER_BLOB" \
      --lease-duration 30 \
      --query "id" -o tsv 2>/dev/null)
    
    if [ -n "$LEASE_ID" ]; then
        echo "$(hostname) is now LEADER"
        
        # Exécuter tâches de leader
        while [ -n "$LEASE_ID" ]; do
            # Renouveler lease toutes les 20 secondes
            az storage blob lease renew \
              --container-name cluster-state \
              --blob-name "$LEADER_BLOB" \
              --lease-id "$LEASE_ID" 2>/dev/null
            
            if [ $? -eq 0 ]; then
                echo "Leader tasks..."
                run_scheduled_tasks
                sleep 20
            else
                echo "Lost leadership"
                LEASE_ID=""
            fi
        done
    else
        echo "$(hostname) is FOLLOWER (leader exists)"
        sleep 10
    fi
done

# Si leader node crash :
# - Lease expire après 30 secondes
# - Autre node peut acquérir lease et devenir leader
```

### État du Blob avec Lease

```bash
# Vérifier état du lease
az storage blob show \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name critical-file.txt \
  --query "properties.lease"

# Output :
# {
#   "duration": "infinite",
#   "state": "leased",
#   "status": "locked"
# }

# États possibles :
# - state: "available" (pas de lease)
# - state: "leased" (lease actif)
# - state: "expired" (lease expiré)
# - state: "breaking" (lease en cours de rupture)
# - state: "broken" (lease cassé)
```

### ⚠️ Best Practices Lease

**Durée :**
- ✅ **Privilégier fixed duration** (60s) pour éviter leases orphelins
- ✅ Renouveler lease régulièrement si traitement > durée lease
- ❌ Éviter infinite lease sauf cas très spécifiques

**Exception Handling :**
- ✅ **TOUJOURS release lease en cas d'erreur** (try/finally block)
- ✅ Implémenter retry avec exponential backoff
- ❌ Ne jamais laisser lease actif si traitement échoue

**Break Lease :**
- ❌ Utiliser Break en **dernier recours** uniquement
- ✅ Break = situation exceptionnelle (worker mort, deadlock)
- ✅ Logger toutes les opérations break pour audit

**Monitoring :**
- ✅ Logger acquisitions/releases/renewals pour debugging
- ✅ Alerter si lease duration trop longue (> 5 minutes)
- ✅ Monitorer taux de collisions (acquire failures)

**Exemple avec Error Handling :**

```bash
#!/bin/bash
set -e  # Exit on error

LEASE_ID=""

# Cleanup function
cleanup() {
    if [ -n "$LEASE_ID" ]; then
        echo "Releasing lease..."
        az storage blob lease release \
          --container-name jobs \
          --blob-name "$JOB" \
          --lease-id "$LEASE_ID" 2>/dev/null || true
    fi
}

# Trap EXIT to ensure cleanup
trap cleanup EXIT

# Acquire lease
LEASE_ID=$(az storage blob lease acquire \
  --container-name jobs \
  --blob-name "job-12345.json" \
  --lease-duration 60 \
  --query "id" -o tsv)

# Process job (any error will trigger cleanup via trap)
process_job

# Explicit cleanup (trap will also run)
cleanup
```

---

## Tableau Récapitulatif des Opérations

| Opération | Max Size | Use Case Principal | Caractéristique Clé |
|-----------|----------|-------------------|---------------------|
| **Put Blob** | 5 GB | Upload simple | Atomique, écrase métadonnées |
| **Put Block + Put Block List** | 190.7 TiB | Gros fichiers | Reprise, parallèle, uncommitted 7j |
| **Copy Blob** | Illimité | Migration, backup | Serveur-side, async >256MB |
| **Set Blob Metadata** | 8 KB | Tagging, indexation | Écrase tout, lowercase keys |
| **Get Blob** | Illimité | Téléchargement | Range requests, conditional |
| **Delete Blob** | - | Nettoyage | Soft delete, snapshots option |
| **Snapshot Blob** | Illimité | Versioning, backup | Read-only, incrémental |
| **Lease Blob** | - | Concurrence | Lock exclusif, 15-60s ou infini |

---

## Points Clés pour l'Examen AZ-104

### Put Blob
- ✅ Max **5 GB** par opération
- ✅ **Écrase** toutes métadonnées existantes
- ✅ Opération **atomique**
- ❌ Pas de reprise en cas d'échec

### Put Block + Put Block List
- ✅ Pour fichiers **> 5 GB**
- ✅ Max **50,000 blocs** × **4000 MiB** = **190.7 TiB**
- ✅ Blocs **uncommitted** pendant **7 jours**
- ✅ Support **reprise** et **parallélisme**
- ✅ AzCopy gère automatiquement

### Copy Blob
- ✅ **Serveur-side** (Azure fait la copie)
- ✅ **Synchrone** < 256 MB, **Asynchrone** > 256 MB
- ✅ SAS token requis pour cross-account
- ✅ Cross-region supporté

### Set Blob Metadata
- ✅ **ÉCRASE** toutes métadonnées (pas de merge)
- ✅ Max **8 KB** total
- ✅ Clés converties en **lowercase**
- ❌ Pas de caractères spéciaux

### Get Blob
- ✅ Support **range requests** (bytes=0-1023)
- ✅ **Conditional** download (If-Modified-Since, ETag)
- ✅ CDN pour optimisation
- ✅ Lecture TOUJOURS possible (même avec lease)

### Delete Blob
- ✅ **Soft Delete** pour protection (7-365 jours)
- ✅ `--delete-snapshots include` pour supprimer snapshots
- ✅ Lifecycle Management pour automatisation
- ✅ Conditional delete avec ETag

### Snapshot Blob
- ✅ **Read-only** et **incrémental**
- ✅ Identifié par **timestamp** ISO 8601
- ✅ Coût uniquement pour **blocs/pages uniques**
- ✅ Snapshots persistent même si blob base supprimé

### Lease Blob
- ✅ **Lock exclusif** en écriture (lecture non bloquée)
- ✅ **Infinite** ou **Fixed** duration (15-60s)
- ✅ Distributed lock pattern
- ❌ Break en dernier recours

---

## Erreurs Courantes aux QCM

| Question | Réponse Correcte ❌ | Explication |
|----------|---------------------|-------------|
| **Put Blob peut uploader fichiers de 100 GB ?** | **Non** | Max 5 GB. Utiliser Put Block + Put Block List |
| **Set Metadata fusionne avec métadonnées existantes ?** | **Non** | ÉCRASE tout. Spécifier toutes les clés |
| **Copy Blob est toujours synchrone ?** | **Non** | Asynchrone pour blobs > 256 MB |
| **Lease Blob empêche lecture du blob ?** | **Non** | Empêche seulement écriture et suppression |
| **Snapshots sont facturés au prix complet du blob ?** | **Non** | Facturé incrémentalement (blocs uniques) |
| **Put Block List peut être appelé plusieurs fois ?** | **Oui** | Recommit avec nouvelle liste de blocks |
| **Blocs uncommitted persistent indéfiniment ?** | **Non** | Supprimés après 7 jours |
| **Get Blob avec range request change le blob ?** | **Non** | Lecture seule, pas de modification |
| **Delete Blob avec Soft Delete supprime définitivement ?** | **Non** | Conservé période rétention (7-365j) |
| **Snapshot peut être modifié après création ?** | **Non** | Read-only, immuable |

---

## Commandes Essentielles à Mémoriser

```bash
# Upload simple (< 5 GB)
az storage blob upload \
  --account-name X --container-name Y --name Z --file F

# Upload gros fichier (> 5 GB)
azcopy copy './file' 'https://account.blob.core.windows.net/container/file?SAS'

# Copier blob
az storage blob copy start \
  --source-uri X --destination-blob Y --destination-container Z

# Métadonnées
az storage blob metadata update \
  --metadata key1=value1 key2=value2

# Télécharger
az storage blob download \
  --container-name X --name Y --file Z

# Supprimer
az storage blob delete \
  --container-name X --name Y

# Snapshot
az storage blob snapshot \
  --container-name X --name Y

# Lease acquire
az storage blob lease acquire \
  --container-name X --blob-name Y --lease-duration 60

# Lease release
az storage blob lease release \
  --container-name X --blob-name Y --lease-id LEASE_ID
```

---

## Scénarios d'Examen Types

**Scénario 1 :**
> Vous devez uploader un fichier vidéo de 50 GB. Quelle commande utilisez-vous ?

**Réponse :** AzCopy (gère automatiquement Put Block + Put Block List)
```bash
azcopy copy './video.mp4' 'https://storage.blob.core.windows.net/videos/video.mp4?SAS'
```

**Scénario 2 :**
> Vous devez migrer 100 GB de données entre deux storage accounts dans différentes régions. Comment minimiser les coûts de bandwidth ?

**Réponse :** Copy Blob (serveur-side, pas d'egress depuis client)
```bash
az storage blob copy start-batch \
  --source-account-name source \
  --destination-account-name dest \
  --pattern '*'
```

**Scénario 3 :**
> Vous avez un blob avec métadonnées `author=Alice`. Vous voulez ajouter `department=IT` sans perdre `author`.

**Réponse :** Récupérer métadonnées existantes, puis Set Metadata avec TOUTES les clés
```bash
# Lire métadonnées existantes
AUTHOR=$(az storage blob metadata show ... --query "author" -o tsv)

# Set avec TOUTES les métadonnées
az storage blob metadata update \
  --metadata author="$AUTHOR" department=IT
```

**Scénario 4 :**
> Vous avez 3 workers qui doivent traiter des jobs. Comment éviter qu'un job soit traité deux fois ?

**Réponse :** Lease Blob avec fixed duration (60s)
```bash
LEASE_ID=$(az storage blob lease acquire \
  --blob-name job-123.json \
  --lease-duration 60 \
  --query "id" -o tsv)

if [ -n "$LEASE_ID" ]; then
    # Traiter job
    process_job
    # Supprimer avec lease
    az storage blob delete --lease-id "$LEASE_ID" ...
fi
```

---

**Bonne préparation pour l'AZ-104 !** 🚀
