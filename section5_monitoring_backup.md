## 5. Monitor and Backup Azure Resources (10-15%)

### 5.1 Azure Monitor

#### Architecture et Fonctionnalités
**Azure Monitor** aide à maximiser la disponibilité et les performances des applications et services

- **Data Collection** : Métriques, logs, traces
- **Storage** : Metrics store, Log Analytics workspace
- **Analysis** : KQL queries, workbooks, dashboards
- **Actions** : Alerts, autoscale, automation

**Différenciation des services de monitoring Azure :**

**Azure Monitor**
- **Fonction** : Maximiser disponibilité et performance des applications
- **Capacités** : Collecte de métriques, logs, alertes, dashboards
- **Usage** : Monitoring complet des ressources Azure
- **API** : HTTP Data Collector API pour envoyer logs vers Log Analytics

**Azure Network Watcher**
- **Fonction** : Monitoring et diagnostic réseau
- **Capacités** : IP Flow Verify, NSG Flow Logs, Packet Capture, Connection Monitor
- **Usage** : Troubleshooting connectivité et diagnostic réseau
- **Scope** : Ressources réseau uniquement (VNets, NSGs, VMs)

**Azure Resource Manager**
- **Fonction** : Service de déploiement et gestion Azure
- **Capacités** : Déploiement, gestion du cycle de vie, RBAC
- **Usage** : Infrastructure as Code, gestion des ressources
- **Scope** : Pas un outil de monitoring

**Network Security Groups (NSG)**
- **Fonction** : Sécurité réseau (firewall)
- **Capacités** : Règles de filtrage de trafic
- **Usage** : Contrôle d'accès réseau
- **Limitation** : Sécurité uniquement, pas de monitoring

#### Cost Management et Budgets

** Configuration des budgets et actions automatiques :**

**Processus d'édition de budget :**
1. **Cost Management + Billing** → **Budgets**
2. **Éditer le budget** associé aux ressources du groupe de ressources
3. **Créer un nouveau Action Group** de type **Runbook**
4. **Choisir "Stop VM"** comme action

** Points clés identifiés :**
- **Cost analysis** : Ne peut pas arrêter automatiquement les VMs
- **Scale Up VM action group** : Non requis pour arrêter les VMs
- **Runbook type** : Obligatoire pour actions d'automatisation
- **Stop VM action** : Action spécifique pour arrêter les machines virtuelles

** Azure Advisor - Cost Optimization :**
- **Cost blade** : Optimisation et réduction des dépenses Azure
- **Identification** : VMs sous-utilisées
- **Performance blade** : Amélioration de la vitesse des applications
- **High availability** : Non disponible via Azure Advisor
- **Operational Excellence** : Efficacité des processus et workflows, gestion des ressources, meilleures pratiques de déploiement

**🎯 Concept clé identifié :** Target Resource pour alertes
- **VM Events/Syslog** → Target = **Log Analytics Workspace**
- **VM Metrics** → Target = **Virtual Machine**
- Les VMs envoient logs vers Log Analytics pour analyse

#### Diagnostic Settings - Collecte de Données de Diagnostic

**⚠️ Erreur Courante QCM : Destinations disponibles pour Diagnostic Settings**

**Vue d'ensemble :**
Les Diagnostic Settings permettent de collecter les **Platform Logs** et **Platform Metrics** des ressources Azure et de les envoyer vers différentes destinations pour analyse, archivage ou intégration.

**Types de données collectées :**
- **Activity Logs** : Opérations sur les ressources (création, suppression, modification)
- **Resource Logs** : Logs internes des ressources (ex: SQL queries, Storage operations)
- **Metrics** : Métriques de performance (CPU, Memory, Network, Storage)

**Destinations Disponibles - Vue Complète :**

| Destination | Use Case | Rétention | Coût | Requiert |
|-------------|----------|-----------|------|----------|
| **Log Analytics Workspace** | Analyse et requêtes KQL | Configurable (30-730 jours) | Ingestion + Rétention | Log Analytics Workspace |
| **Storage Account** | Archivage long terme | Illimitée | Stockage Blob | Storage Account |
| **Event Hub** | Streaming vers outils externes | Temps réel | Throughput | Event Hub Namespace |
| **Partner Solutions** | SIEM tiers (Splunk, Datadog) | Selon partenaire | Selon partenaire | Integration configurée |

**1. Log Analytics Workspace - Analyse et Alertes**

**⚠️ Destination PRINCIPALE pour l'examen AZ-104**

**Use Cases :**
- **Analyse avec KQL** : Requêtes complexes sur logs
- **Alertes** : Créer alertes basées sur logs
- **Dashboards** : Visualisations Azure Monitor
- **Workbooks** : Rapports interactifs
- **Insights** : VM Insights, Container Insights, Application Insights

**Configuration via Azure CLI :**
```bash
# Créer un Log Analytics Workspace
az monitor log-analytics workspace create \
  --resource-group myRG \
  --workspace-name myWorkspace \
  --location eastus \
  --retention-time 90

# Configurer Diagnostic Settings pour une VM
az monitor diagnostic-settings create \
  --name VMDiagnostics \
  --resource /subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Compute/virtualMachines/myVM \
  --workspace /subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.OperationalInsights/workspaces/myWorkspace \
  --logs '[{"category":"Administrative","enabled":true},{"category":"Security","enabled":true}]' \
  --metrics '[{"category":"AllMetrics","enabled":true}]'
```

**Configuration via PowerShell :**
```powershell
# Créer Log Analytics Workspace
New-AzOperationalInsightsWorkspace `
  -ResourceGroupName "myRG" `
  -Name "myWorkspace" `
  -Location "East US" `
  -Sku "PerGB2018" `
  -RetentionInDays 90

# Configurer Diagnostic Settings pour Storage Account
$storageAccount = Get-AzStorageAccount -ResourceGroupName "myRG" -Name "mystorageaccount"
$workspace = Get-AzOperationalInsightsWorkspace -ResourceGroupName "myRG" -Name "myWorkspace"

Set-AzDiagnosticSetting `
  -ResourceId $storageAccount.Id `
  -Name "StorageDiagnostics" `
  -WorkspaceId $workspace.ResourceId `
  -Enabled $true `
  -Category @("StorageWrite", "StorageRead", "StorageDelete")
```

**Requêtes KQL Utiles :**
```kusto
// Toutes les opérations de write sur Storage Account
StorageBlobLogs
| where OperationName == "PutBlob"
| project TimeGenerated, AccountName, Uri, StatusCode, CallerIpAddress

// VMs avec CPU > 80%
Perf
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| where CounterValue > 80
| summarize avg(CounterValue) by Computer, bin(TimeGenerated, 5m)

// Activité administrative
AzureActivity
| where OperationNameValue contains "Microsoft.Compute/virtualMachines"
| where ActivityStatusValue == "Success"
| project TimeGenerated, Caller, OperationNameValue, ResourceGroup
```

**Limites et Coûts :**
- **Ingestion** : ~$2.50/GB (peut varier selon engagement)
- **Rétention** : Incluse jusqu'à 31 jours, puis ~$0.12/GB/mois
- **Requêtes** : Gratuites (incluses)
- **Alertes** : Coût selon fréquence d'évaluation

**2. Storage Account - Archivage Long Terme**

**Use Cases :**
- **Archivage** : Logs pour conformité (1-10 ans)
- **Audit** : Historique complet pour forensic
- **Coût réduit** : Alternative économique à Log Analytics
- **Backup logs** : Logs de sauvegarde et restauration

**Configuration via Azure CLI :**
```bash
# Configurer Diagnostic Settings vers Storage Account
az monitor diagnostic-settings create \
  --name VMDiagToStorage \
  --resource /subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Compute/virtualMachines/myVM \
  --storage-account /subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Storage/storageAccounts/mystorageaccount \
  --logs '[{"category":"Administrative","enabled":true,"retentionPolicy":{"enabled":true,"days":365}}]' \
  --metrics '[{"category":"AllMetrics","enabled":true,"retentionPolicy":{"enabled":true,"days":90}}]'
```

**Structure de Stockage :**
```
mystorageaccount
├── insights-logs-administrative/
│   ├── resourceId=/SUBSCRIPTIONS/{sub-id}/RESOURCEGROUPS/{rg}/PROVIDERS/{provider}/{resource}
│   │   ├── y=2025/m=10/d=27/h=12/m=00/
│   │   │   └── PT1H.json
```

**Format des Logs (JSON) :**
```json
{
  "time": "2025-10-27T12:00:00.000Z",
  "resourceId": "/subscriptions/.../resourceGroups/myRG/providers/Microsoft.Compute/virtualMachines/myVM",
  "category": "Administrative",
  "operationName": "Microsoft.Compute/virtualMachines/write",
  "resultType": "Success",
  "callerIpAddress": "203.0.113.50",
  "identity": {
    "authorization": {
      "action": "Microsoft.Compute/virtualMachines/write",
      "scope": "/subscriptions/.../resourceGroups/myRG"
    }
  }
}
```

**Lifecycle Management :**
```bash
# Créer lifecycle policy pour archiver après 90 jours
az storage account management-policy create \
  --account-name mystorageaccount \
  --resource-group myRG \
  --policy '{
    "rules": [{
      "name": "ArchiveLogs",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["insights-logs-"]
        },
        "actions": {
          "baseBlob": {
            "tierToCool": {"daysAfterModificationGreaterThan": 30},
            "tierToArchive": {"daysAfterModificationGreaterThan": 90}
          }
        }
      }
    }]
  }'
```

**Limites :**
- **Analyse** : Pas de requêtes KQL (nécessite téléchargement)
- **Alertes** : Pas d'alertes en temps réel
- **Coût** : ~$0.02/GB/mois (Hot), ~$0.01/GB/mois (Cool), ~$0.002/GB/mois (Archive)

**3. Event Hub - Streaming en Temps Réel**

**Use Cases :**
- **SIEM Integration** : Splunk, QRadar, ArcSight
- **Custom Processing** : Azure Functions, Stream Analytics
- **Multi-destination** : Plusieurs consommateurs en parallèle
- **Near real-time** : Latence < 1 minute

**Configuration via Azure CLI :**
```bash
# Créer Event Hub Namespace
az eventhubs namespace create \
  --resource-group myRG \
  --name myEventHubNS \
  --location eastus \
  --sku Standard

# Créer Event Hub
az eventhubs eventhub create \
  --resource-group myRG \
  --namespace-name myEventHubNS \
  --name diagnostics-hub \
  --partition-count 4 \
  --message-retention 7

# Configurer Diagnostic Settings vers Event Hub
az monitor diagnostic-settings create \
  --name VMDiagToEventHub \
  --resource /subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Compute/virtualMachines/myVM \
  --event-hub myEventHubNS \
  --event-hub-rule /subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.EventHub/namespaces/myEventHubNS/authorizationRules/RootManageSharedAccessKey \
  --logs '[{"category":"Administrative","enabled":true}]' \
  --metrics '[{"category":"AllMetrics","enabled":true}]'
```

**Consumer Example (Azure Function) :**
```csharp
[FunctionName("ProcessDiagnosticLogs")]
public static async Task Run(
    [EventHubTrigger("diagnostics-hub", Connection = "EventHubConnection")] EventData[] events,
    ILogger log)
{
    foreach (var eventData in events)
    {
        string messageBody = Encoding.UTF8.GetString(eventData.Body.Array);
        var logEntry = JsonConvert.DeserializeObject<DiagnosticLog>(messageBody);
        
        // Custom processing
        if (logEntry.Category == "Security" && logEntry.ResultType == "Failure")
        {
            await SendAlertToSlack(logEntry);
        }
    }
}
```

**Limites et Coûts :**
- **Throughput** : 1 MB/s par partition (Standard), 20 MB/s (Premium)
- **Rétention** : 1-7 jours (Standard), jusqu'à 90 jours (Premium)
- **Coût** : ~$0.028/million events + $11/throughput unit/mois

**4. Partner Solutions - SIEM Tiers**

**Partenaires Disponibles :**
- **Datadog** : APM et infrastructure monitoring
- **Elastic (Elasticsearch)** : Search, analytics, visualization
- **LogRhythm** : SIEM et threat detection
- **Splunk** : Enterprise SIEM
- **Sumo Logic** : Cloud-native SIEM

**Configuration :**
```bash
# Liste des partenaires disponibles
az monitor diagnostic-settings subscription list-categories

# Configuration vers Partner Solution (exemple Datadog)
az monitor diagnostic-settings create \
  --name VMDiagToDatadog \
  --resource /subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Compute/virtualMachines/myVM \
  --marketplace-partner-id /subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Datadog/monitors/myDatadogMonitor \
  --logs '[{"category":"Administrative","enabled":true}]' \
  --metrics '[{"category":"AllMetrics","enabled":true}]'
```

**Avantages :**
- **Corrélation multi-cloud** : Logs Azure + AWS + GCP
- **Threat intelligence** : Détection avancée de menaces
- **Compliance** : Rapports de conformité pré-configurés
- **Expertise** : Support spécialisé SIEM

**Configuration Multi-Destinations**

**⚠️ Point Important pour l'Examen :**
Une ressource peut avoir **plusieurs Diagnostic Settings** envoyant vers **différentes destinations simultanément**.

**Exemple - Architecture Complète :**
```bash
# Diagnostic Setting 1 : Analyse temps réel
az monitor diagnostic-settings create \
  --name RealTimeAnalytics \
  --resource $vmId \
  --workspace $logAnalyticsId \
  --logs '[{"category":"Administrative","enabled":true}]'

# Diagnostic Setting 2 : Archivage long terme
az monitor diagnostic-settings create \
  --name LongTermArchive \
  --resource $vmId \
  --storage-account $storageAccountId \
  --logs '[{"category":"Administrative","enabled":true,"retentionPolicy":{"enabled":true,"days":2555}}]'

# Diagnostic Setting 3 : Streaming vers SIEM
az monitor diagnostic-settings create \
  --name SIEMIntegration \
  --resource $vmId \
  --event-hub $eventHubNSId \
  --logs '[{"category":"Security","enabled":true}]'
```

**Architecture Typique - DevOps :**
```
Azure Resource (VM, Storage, NSG, etc.)
├── Diagnostic Setting 1 → Log Analytics Workspace
│   ├── Requêtes KQL
│   ├── Alertes temps réel
│   └── Dashboards Azure Monitor
│
├── Diagnostic Setting 2 → Storage Account
│   ├── Archivage 7 ans (conformité)
│   ├── Lifecycle policy (Cool → Archive)
│   └── Backup des logs
│
├── Diagnostic Setting 3 → Event Hub
│   ├── Azure Function (processing custom)
│   ├── Stream Analytics
│   └── Splunk/Datadog Integration
│
└── Diagnostic Setting 4 → Partner Solution (Datadog)
    ├── APM
    ├── Infrastructure monitoring
    └── Multi-cloud correlation
```

**Scénarios d'Examen - Diagnostic Settings**

| Besoin | Destination | Raison |
|--------|-------------|--------|
| **Créer alertes sur logs VM** | Log Analytics Workspace | Permet requêtes KQL et alertes |
| **Archiver logs 5 ans pour audit** | Storage Account | Coût faible, rétention illimitée |
| **Envoyer logs vers Splunk** | Event Hub OU Partner Solution | Streaming temps réel ou intégration native |
| **Analyser activité administrative** | Log Analytics Workspace | Requêtes et dashboards |
| **Conformité RGPD (7 ans logs)** | Storage Account | Archivage long terme économique |
| **Multi-cloud SIEM** | Partner Solution (Datadog, Splunk) | Corrélation Azure + AWS + on-prem |

**Catégories de Logs Disponibles par Type de Ressource**

**Virtual Machines :**
- Performance counters (métriques CPU, Memory, Disk, Network)
- Event logs (Windows) / Syslog (Linux)
- IIS logs (si IIS installé)
- Nécessite **VM agent** ou **Azure Monitor Agent**

**Storage Account :**
- StorageRead, StorageWrite, StorageDelete
- Transaction logs (blob, file, queue, table)
- Metrics (availability, latency, capacity)

**Network Security Group (NSG) :**
- NSG Flow Logs (version 1 ou 2)
- Network Watcher requis

**Azure SQL Database :**
- SQLInsights, QueryStoreRuntimeStatistics
- Errors, Timeouts, Deadlocks
- Automatic tuning, Intelligent insights

**App Service :**
- AppServiceHTTPLogs, AppServiceConsoleLogs
- AppServiceAppLogs, AppServiceAuditLogs
- AppServicePlatformLogs

**Best Practices - Diagnostic Settings**

✅ **À FAIRE :**
- **Log Analytics** pour toutes les ressources critiques (analyse + alertes)
- **Storage Account** pour archivage conformité (LRS ou GRS selon criticité)
- **Rétention appropriée** : 90 jours Log Analytics, 7 ans Storage
- **Multi-destinations** : Log Analytics (analyse) + Storage (archive)
- **Catégories sélectives** : N'activer que logs nécessaires (coûts)
- **Lifecycle policies** : Archiver logs anciens (Cool/Archive tier)
- **RBAC strict** : Limiter accès aux logs sensibles

❌ **À ÉVITER :**
- Activer tous les logs sans distinction (coûts élevés)
- Oublier la rétention (défaut = 30 jours Log Analytics)
- Storage Account sans lifecycle policy (coûts croissants)
- Event Hub pour archivage (rétention limitée)
- Multiple Diagnostic Settings avec mêmes logs (duplication coûts)

**PowerShell - Gestion Complète :**
```powershell
# Lister Diagnostic Settings d'une ressource
Get-AzDiagnosticSetting -ResourceId $vmId

# Supprimer un Diagnostic Setting
Remove-AzDiagnosticSetting `
  -ResourceId $vmId `
  -Name "VMDiagnostics"

# Exporter configuration Diagnostic Settings
Get-AzDiagnosticSetting -ResourceId $vmId |
  ConvertTo-Json -Depth 10 |
  Out-File "diagnostic-settings-backup.json"
```

**Troubleshooting Diagnostic Settings**

| Problème | Cause Possible | Solution |
|----------|----------------|----------|
| Logs n'arrivent pas dans Log Analytics | Agent non installé (VM) | Installer Azure Monitor Agent |
| Storage Account ne reçoit pas de logs | Permissions insuffisantes | Vérifier RBAC (Monitoring Contributor) |
| Event Hub sans données | Authorization rule incorrecte | Utiliser RootManageSharedAccessKey |
| Coûts élevés | Tous les logs activés | Sélectionner catégories pertinentes uniquement |
| Rétention trop courte | Défaut 30 jours | Configurer rétention 90-730 jours |

#### Types d'Alertes

**Metric Alerts**
- **Seuils numériques** : CPU > 80%
- **Near real-time** : Évaluation fréquente
- **Multiple dimensions** : Filtrage par propriétés

**Log Alerts**
- **KQL queries** : Requêtes sur logs
- **Example** : `Event | search "error"`
- **Flexibility** : Logique complexe possible

**Activity Log Alerts**
- **Administrative events** : Création/suppression ressources
- **Service Health** : Incidents Azure
- **Resource Health** : État des ressources

#### Requêtes KQL Essentielles
```kusto
// Rechercher dans une table spécifique
Event | search "error"

// Compter par computer
Event | summarize count() by Computer

// Filtrer par niveau
Event | where EventLevelName == "Error"

// Timeline des événements
Event | where TimeGenerated > ago(1h) | sort by TimeGenerated desc
```

### 5.2 Azure Backup

#### Recovery Services Vault

** Règles critiques identifiées :**

**Localisation :**
- Vault et ressources **dans la même région**
- Example : Vault2 (West US) peut sauvegarder Storage1 (West US)
- Share1 sauvegardable car dans Storage1 même région

**Suppression du Vault :**
1. **D'abord** : Arrêter backup de tous les éléments protégés
2. **Ensuite** : Supprimer le vault
3. **Erreur** : Vault ne peut pas être supprimé avec éléments protégés

#### Backup Policies

** Limites par type de ressource :**
- **VMs** : Maximum 100 VMs par policy
- **SQL Databases** : Policy séparée requise
- **File Shares** : Policy séparée requise

**Example :** 100 VMs + 20 SQL + 50 Files = **3 policies minimum**

#### Changement de Vault

**Pour Backup :**
1. Arrêter backup dans vault actuel (RSV1)
2. Configurer backup dans nouveau vault (RSV2)

**Pour Site Recovery :**
- VM → Disaster Recovery → Replication Settings → Nouveau vault

#### Types de Backup
- **Azure VMs** : Snapshots dans région source
- **On-premises** : MARS agent ou DPM/MABS
- **SQL in Azure VM** : Application-consistent backups
- **Azure Files** : Snapshots au niveau share

### 5.3 Azure Site Recovery

#### Supported Scenarios
- **Azure to Azure** : VMs entre régions Azure
- **VMware to Azure** : VMs VMware vers Azure
- **Hyper-V to Azure** : VMs Hyper-V vers Azure
- **Physical to Azure** : Serveurs physiques vers Azure

#### Components
- **Recovery Services Vault** : Orchestration et management
- **Configuration Server** : Pour VMware (on-premises)
- **Process Server** : Réplication data processing
- **Master Target** : Receive replication data

#### Recovery Plans
- **Grouping** : VMs dans groupes logiques
- **Sequencing** : Ordre de démarrage
- **Scripts** : Automation pendant failover
- **Manual actions** : Actions manuelles requises

---

## Tips Pratiques d'Examen - Insights des Questions Réelles

### 4.5 Pièges Fréquents et Solutions

#### VNet Peering - Erreurs de Plages d'Adresses
** Piège identifié :** Confusion entre plages chevauchantes et non-chevauchantes
- **Erreur courante** : Essayer de peerer 192.168.0.0/24 avec 192.168.0.0/16
- **Raison** : /24 est inclus dans /16 → chevauchement détecté par Azure
- **Solution** : Utiliser des plages complètement différentes (10.x.x.x vs 172.x.x.x)
- **Validation** : Azure bloque automatiquement les peerings avec chevauchement

#### NSG - Priorités et Évaluation en Cascade
** Piège identifié :** Oublier l'évaluation en cascade Subnet → NIC
- **Erreur courante** : NSG Subnet Allow + NSG NIC Deny = Trafic bloqué
- **Raison** : Les deux niveaux doivent autoriser le trafic
- **Solution** : Vérifier les NSG aux deux niveaux lors du troubleshooting
- **Optimisation** : Un seul NSG Deny à n'importe quel niveau bloque tout

#### Load Balancer - Session Persistence vs NAT Rules
** Piège identifié :** Confusion entre session persistence et NAT rules
- **Erreur courante** : Utiliser NAT rules pour maintenir les sessions utilisateur
- **Raison** : NAT rules = redirection de trafic, Session persistence = maintien de session
- **Solution** : Client IP + Protocol pour les applications avec état
- **Cas d'usage** : E-commerce, applications avec paniers, sessions utilisateur

#### Diagnostic Réseau - Outils Spécialisés
** Piège identifié :** Utiliser les mauvais outils pour le diagnostic
- **Erreur courante** : `Get-AzVirtualNetworkUsageList` pour diagnostic de ports
- **Raison** : PowerShell Azure ≠ outils de diagnostic réseau
- **Solution** : `netstat -an` pour ports d'écoute, `Test-NetConnection` pour connectivité
- **Règle** : Diagnostic réseau = outils système Windows, pas cmdlets Azure

### 4.6 Matrice de Décision Rapide

| Problème | Diagnostic | Solution | Outil/Commande |
|----------|------------|----------|----------------|
| VNets ne communiquent pas | Vérifier plages d'adresses | VNet Peering | Portail Azure → Peerings |
| Trafic bloqué | Vérifier NSG Subnet + NIC | Ajuster priorités | NSG → Rules → Priority |
| Sessions perdues | Vérifier session persistence | Client IP + Protocol | Load Balancer → Settings |
| Ports d'écoute | Diagnostic réseau | Vérifier services | `netstat -an` |
| Connectivité Internet | Test de connectivité | Vérifier NSG outbound | `Test-NetConnection` |

---

## Points Critiques Basés sur Vos Erreurs

### 1. Log Analytics = Hub Central pour Monitoring
** Erreur courante :** Choisir la VM comme target resource
** Correct :** Log Analytics Workspace pour toutes les alertes de logs
- Windows Event Logs → Log Analytics
- Linux Syslog → Log Analytics
- VM metrics → VM directement

### 2. Règle de Même Région
** Erreur :** Vault et Storage dans régions différentes
** Correct :** Toujours même région pour :
- Recovery Services Vault + Storage Account
- Traffic Analytics components
- Backup sources et destinations

### 3. Disque D: = Temporaire et Volatil
** Erreur :** Stocker des données importantes sur D:
** Correct :** D: pour cache/temp uniquement
- C: = Persistant (OS, apps)
- D: = Temporaire (perdu lors maintenance)
- E:, F: = Persistants (données)

### 4. Storage Account Types et Limitations
** Erreur :** Premium File Shares sur StorageV2
** Correct :** FileStorage accounts uniquement
- StorageV2 = Standard files uniquement
- BlobStorage = Pas de files du tout

### 5. NSG Sharing et Optimisation
** Erreur :** Un NSG par VM
** Correct :** Un NSG partagé si mêmes règles
- 5 VMs = 5 NICs + 1 NSG (optimal)

### 6. Recovery Services Vault Management
** Erreur :** Essayer supprimer vault avec backups actifs
** Correct :** Toujours arrêter backups d'abord
- Stop backup → Delete vault
- Change vault = Stop + Start elsewhere

### 7. App Service Deployment Workflow
** Erreur :** Deploy direct en production
** Correct :** Deploy → Test → Swap
- Staging slot pour tests
- Production swap pour zero downtime

### 8. Root Management Group Access
** Erreur :** Accès direct au root MG
** Correct :** Global Admin + elevation required
- Aucun accès par défaut
- Global Admin doit s'élever

### 9. Azure Bastion vs Remote Desktop
** Erreur :** Exposer les ports RDP/SSH sur Internet
** Correct :** Utiliser Azure Bastion
- Azure Bastion = Accès via navigateur, pas d'exposition de ports
- Remote Desktop = Ports exposés, vulnérable aux attaques
- Azure Monitor et Network Watcher = Pas de connectivité aux VMs

### 10. Private DNS Zone Association
** Erreur :** Essayer d'utiliser directement une Private DNS Zone
** Correct :** Créer un Virtual Network Link
- Virtual Network Link = Associe VNet à Private DNS Zone
- DNS Private Resolver = Proxy pour on-premises ↔ Azure
- Custom DNS Server = Complexe et ne fonctionne pas avec Private DNS Zones

### 11. Network Watcher IP Flow Verify
** Erreur :** Utiliser NSG Flow Logs ou Packet Capture d'abord
** Correct :** IP Flow Verify pour diagnostic rapide
- IP Flow Verify = Identifie le NSG bloquant directement
- NSG Flow Logs = Configuration complexe, analyse manuelle
- Packet Capture = Détails mais pas d'identification NSG

### 12. Load Balancer Troubleshooting
** Erreur :** Modifier session persistence ou timeout pour résoudre connectivité
** Correct :** Vérifier Health Probes, Ports, NSG rules
- Health Probes = Configuration et réponses des VMs
- Port Listening = Applications écoutent sur les bons ports
- NSG Rules = Autoriser trafic au niveau subnet ET NIC
- Session persistence/timeout = Ne résolvent pas les problèmes de base

---

## Checklist Final d'Examen

### Identities and Governance
- [ ] Dynamic group rules syntax : `(user.property -eq "value")`
- [ ] Custom domain DNS records : TXT ou MX
- [ ] Root MG access : Global Admin + elevation
- [ ] RBAC scopes : MG → Subscription → RG → Resource
- [ ] **User Administrator** : Gestion utilisateurs et groupes (moindre privilège)
- [ ] **Global Administrator** : Toutes permissions (à utiliser avec parcimonie)

### Storage
- [ ] FileStorage pour Premium files uniquement
- [ ] Import/Export destinations : Blob + Files (5TB max)
- [ ] Replication types par account type
- [ ] Port 445 pour Azure Files SMB
- [ ] **Data Lake Storage Gen2** : Hierarchical Namespace irréversible
- [ ] **ACLs POSIX** : Permissions granulaires + RBAC (le plus restrictif s'applique)
- [ ] **ABFS protocol** : abfs:// ou abfss:// pour Hadoop/Spark
- [ ] **Storage Blob Data Owner** : Seul rôle pour modifier ACLs
- [ ] **Parquet format** : Format recommandé pour Big Data analytics

### Compute
- [ ] Disque D: temporaire, C: persistant
- [ ] VMSS pour high availability pendant maintenance
- [ ] Template deployment : Resource Group configurable
- [ ] Availability Zones : Managed disks requis

### Networking
- [ ] NSG sharing entre ressources
- [ ] DNS interne : vm-name.internal.cloudapp.net
- [ ] Traffic Analytics : Log Analytics + Storage Account
- [ ] Connection Monitor pour RTT measurements
- [ ] **VNet Peering** : Plages d'adresses non-chevauchantes obligatoires
- [ ] **NSG Priorities** : Plus bas = plus prioritaire (100 < 200)
- [ ] **Session Persistence** : Client IP + Protocol pour sticky sessions
- [ ] **Commandes réseau** : `netstat -an` pour ports d'écoute
- [ ] **Azure Bastion** : Pas d'exposition RDP/SSH, accès via navigateur
- [ ] **Private DNS Zone** : Virtual Network Link pour associer un VNet
- [ ] **DNS Private Resolver** : Proxy DNS entre on-premises et Azure
- [ ] **IP Flow Verify** : Identifier le NSG bloquant la communication
- [ ] **Load Balancer Troubleshooting** : Health probes, ports, NSG rules

### Monitoring & Backup
- [ ] Log Analytics Workspace comme target pour VM alerts
- [ ] Backup policies : 100 VMs max par policy
- [ ] Recovery Services Vault : même région
- [ ] Stop backup avant delete vault
- [ ] KQL syntax : `Table | search "term"`
- [ ] **Azure Monitor** : Maximiser disponibilité et performance des applications
- [ ] **Network Watcher** : Monitoring réseau uniquement (pas connectivité VMs)
- [ ] **NSG vs Monitoring** : NSG = sécurité, pas monitoring

---

##  Ressources d'Étude Recommandées

### Documentation Microsoft
- Azure Architecture Center
- Azure Well-Architected Framework
- Azure Best Practices

### Labs Pratiques
- Microsoft Learn modules
- Azure free account (12 mois)
- Hands-on labs

### Examens Blancs
- MeasureUp practice tests
- Whizlabs Azure exams
- Tutorials Dojo practice tests

**Temps de préparation recommandé :** 40-60 heures d'étude + practice labs
