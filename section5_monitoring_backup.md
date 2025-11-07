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

**⚠️ Note Importante :** Les Partner Solutions (Datadog, Elastic, Splunk) s'intègrent généralement via **Event Hub** comme intermédiaire ou via des ressources Azure natives spécifiques (ex: `Microsoft.Datadog/monitors`), plutôt que directement via Diagnostic Settings standard.

**Partenaires Disponibles :**
- **Datadog** : APM et infrastructure monitoring
- **Elastic (Elasticsearch)** : Search, analytics, visualization
- **LogRhythm** : SIEM et threat detection
- **Splunk** : Enterprise SIEM
- **Sumo Logic** : Cloud-native SIEM

**Configuration Recommandée - Via Event Hub :**
```bash
# Créer Event Hub pour intégration SIEM
az eventhubs namespace create \
  --resource-group myRG \
  --name mySIEMEventHub \
  --location eastus \
  --sku Standard

az eventhubs eventhub create \
  --resource-group myRG \
  --namespace-name mySIEMEventHub \
  --name diagnostics-to-siem

# Configurer Diagnostic Settings vers Event Hub
az monitor diagnostic-settings create \
  --name VMDiagToSIEM \
  --resource /subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Compute/virtualMachines/myVM \
  --event-hub mySIEMEventHub \
  --event-hub-rule /subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.EventHub/namespaces/mySIEMEventHub/authorizationRules/RootManageSharedAccessKey \
  --logs '[{"category":"Administrative","enabled":true}]' \
  --metrics '[{"category":"AllMetrics","enabled":true}]'

# Configurer le SIEM pour consommer depuis Event Hub
```

**Configuration Alternative - Ressources Natives (Datadog Example) :**
```bash
# Créer ressource Datadog dans Azure
az datadog monitor create \
  --resource-group myRG \
  --name myDatadogMonitor \
  --location eastus \
  --sku-name "Linked"

# L'intégration se configure ensuite dans le portail Datadog
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
- **Destinations disponibles** :
  - **Storage Account** : Archivage des logs (méthode traditionnelle)
  - **Log Analytics Workspace** : Envoi direct sans Storage Account (depuis 2023)
  - **Storage Account + Log Analytics** : Combinaison pour archivage + analyse

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

#### Action Groups - Notifications et Actions Automatiques

**⚠️ Concept Important pour l'Examen**

Les **Action Groups** définissent les actions à exécuter lorsqu'une alerte est déclenchée.

**Types d'Actions Disponibles :**

| Type | Description | Use Case |
|------|-------------|----------|
| **Email/SMS/Push/Voice** | Notifications aux personnes | Alertes critiques équipe DevOps |
| **Azure Function** | Exécution code serverless | Processing custom, enrichissement données |
| **Logic App** | Workflows complexes | Intégration ServiceNow, Jira, Teams |
| **Webhook** | Appel HTTP endpoint | Intégration systèmes externes |
| **Automation Runbook** | Scripts PowerShell/Python | Remediation automatique (restart VM) |
| **ITSM** | Incident management | ServiceNow, BMC Remedy |
| **Event Hub** | Streaming événements | Archivage, processing en masse |
| **Secure Webhook** | Webhook avec authentification Azure AD | Sécurité renforcée |

**Configuration via Azure CLI :**
```bash
# Créer Action Group avec email et runbook
az monitor action-group create \
  --name CriticalAlerts \
  --resource-group myRG \
  --short-name CritAlerts \
  --email-receiver admin email=admin@contoso.com \
  --automation-runbook name=StopVM \
    runbookName=Stop-AzureVM \
    webhookResourceId=/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Automation/automationAccounts/myAutomation/webhooks/stopvm \
    serviceUri=https://s1events.azure-automation.net/webhooks?token=...
```

**Configuration via PowerShell :**
```powershell
# Email Receiver
$emailReceiver = New-AzActionGroupReceiver `
  -Name "AdminEmail" `
  -EmailReceiver `
  -EmailAddress "admin@contoso.com"

# Runbook Receiver pour auto-remediation
$runbookReceiver = New-AzActionGroupReceiver `
  -Name "StopVMRunbook" `
  -AutomationRunbookReceiver `
  -AutomationAccountId "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Automation/automationAccounts/myAutomation" `
  -RunbookName "Stop-AzureVM" `
  -WebhookResourceId "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Automation/automationAccounts/myAutomation/webhooks/stopvm" `
  -IsGlobalRunbook $false `
  -ServiceUri "https://s1events.azure-automation.net/webhooks?token=..." `
  -UseCommonAlertSchema $true

# Créer Action Group
Set-AzActionGroup `
  -Name "CriticalAlerts" `
  -ResourceGroupName "myRG" `
  -ShortName "CritAlerts" `
  -Receiver $emailReceiver, $runbookReceiver
```

**Common Alert Schema :**
```json
{
  "schemaId": "azureMonitorCommonAlertSchema",
  "data": {
    "essentials": {
      "alertId": "/subscriptions/{sub-id}/providers/Microsoft.AlertsManagement/alerts/{alert-id}",
      "alertRule": "CPU > 80%",
      "severity": "Sev2",
      "signalType": "Metric",
      "monitorCondition": "Fired",
      "monitoringService": "Platform",
      "alertTargetIDs": ["/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Compute/virtualMachines/myVM"],
      "originAlertId": "{guid}",
      "firedDateTime": "2025-10-30T10:00:00.000Z",
      "description": "CPU usage exceeded 80%"
    },
    "alertContext": {
      "properties": null,
      "conditionType": "SingleResourceMultipleMetricCriteria",
      "condition": {
        "allOf": [
          {
            "metricName": "Percentage CPU",
            "metricValue": 85.5,
            "threshold": 80,
            "operator": "GreaterThan",
            "timeAggregation": "Average"
          }
        ]
      }
    }
  }
}
```

**Best Practices - Action Groups :**
✅ **À FAIRE :**
- Un Action Group par criticité (Critical, Warning, Info)
- Common Alert Schema pour uniformité
- Runbooks pour auto-remediation (restart services, scale out)
- Email pour notifications humaines
- ITSM pour tracking incidents
- Tester régulièrement les Action Groups

❌ **À ÉVITER :**
- Trop de notifications (alert fatigue)
- Actions destructives sans confirmation
- Webhooks non sécurisés pour données sensibles

#### Azure Monitor Agents - Collecte de Données VM

**⚠️ Important pour l'Examen AZ-104**

**Types d'Agents Disponibles :**

| Agent | Plateformes | Use Case | Status |
|-------|-------------|----------|--------|
| **Azure Monitor Agent (AMA)** | Windows, Linux | Agent unifié recommandé | ✅ Recommandé |
| **Log Analytics Agent (MMA/OMS)** | Windows, Linux | Legacy, toujours supporté | ⚠️ Deprecated 2024 |
| **Diagnostics Extension (WAD/LAD)** | Windows, Linux | Métriques vers Storage Account | Legacy |
| **Telegraf Agent** | Linux | Métriques custom | Spécialisé |
| **Dependency Agent** | Windows, Linux | Service Map, VM Insights | Complémentaire |

**Azure Monitor Agent (AMA) - Configuration**

**⚠️ Agent Recommandé pour Nouvelles Implémentations**

**Installation via Azure CLI :**
```bash
# Installer AMA sur VM Windows
az vm extension set \
  --resource-group myRG \
  --vm-name myWindowsVM \
  --name AzureMonitorWindowsAgent \
  --publisher Microsoft.Azure.Monitor \
  --enable-auto-upgrade true

# Installer AMA sur VM Linux
az vm extension set \
  --resource-group myRG \
  --vm-name myLinuxVM \
  --name AzureMonitorLinuxAgent \
  --publisher Microsoft.Azure.Monitor \
  --enable-auto-upgrade true
```

**Installation via PowerShell :**
```powershell
# Windows VM
Set-AzVMExtension `
  -ResourceGroupName "myRG" `
  -VMName "myWindowsVM" `
  -Name "AzureMonitorWindowsAgent" `
  -Publisher "Microsoft.Azure.Monitor" `
  -ExtensionType "AzureMonitorWindowsAgent" `
  -TypeHandlerVersion "1.0" `
  -Location "East US" `
  -EnableAutomaticUpgrade $true

# Linux VM
Set-AzVMExtension `
  -ResourceGroupName "myRG" `
  -VMName "myLinuxVM" `
  -Name "AzureMonitorLinuxAgent" `
  -Publisher "Microsoft.Azure.Monitor" `
  -ExtensionType "AzureMonitorLinuxAgent" `
  -TypeHandlerVersion "1.0" `
  -Location "East US" `
  -EnableAutomaticUpgrade $true
```

**Data Collection Rules (DCR) - Configuration de Collecte**

**⚠️ DCR obligatoires avec Azure Monitor Agent**

```bash
# Créer Data Collection Rule
az monitor data-collection rule create \
  --name myDCR \
  --resource-group myRG \
  --location eastus \
  --rule-file dcr-config.json

# Associer DCR à une VM
az monitor data-collection rule association create \
  --name myDCRAssociation \
  --rule-id /subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Insights/dataCollectionRules/myDCR \
  --resource /subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Compute/virtualMachines/myVM
```

**Exemple DCR JSON :**
```json
{
  "location": "eastus",
  "properties": {
    "dataSources": {
      "performanceCounters": [
        {
          "name": "perfCounterDataSource",
          "streams": ["Microsoft-Perf"],
          "samplingFrequencyInSeconds": 60,
          "counterSpecifiers": [
            "\\Processor(_Total)\\% Processor Time",
            "\\Memory\\Available Bytes",
            "\\LogicalDisk(_Total)\\% Free Space"
          ]
        }
      ],
      "windowsEventLogs": [
        {
          "name": "eventLogsDataSource",
          "streams": ["Microsoft-Event"],
          "xPathQueries": [
            "Application!*[System[(Level=1 or Level=2 or Level=3)]]",
            "System!*[System[(Level=1 or Level=2 or Level=3)]]"
          ]
        }
      ],
      "syslog": [
        {
          "name": "syslogDataSource",
          "streams": ["Microsoft-Syslog"],
          "facilityNames": ["auth", "authpriv", "cron", "daemon", "kern"],
          "logLevels": ["Debug", "Info", "Notice", "Warning", "Error", "Critical", "Alert", "Emergency"]
        }
      ]
    },
    "destinations": {
      "logAnalytics": [
        {
          "workspaceResourceId": "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.OperationalInsights/workspaces/myWorkspace",
          "name": "centralWorkspace"
        }
      ]
    },
    "dataFlows": [
      {
        "streams": ["Microsoft-Perf", "Microsoft-Event", "Microsoft-Syslog"],
        "destinations": ["centralWorkspace"]
      }
    ]
  }
}
```

**Avantages Azure Monitor Agent vs Legacy :**
- ✅ **Data Collection Rules (DCR)** : Configuration centralisée et réutilisable
- ✅ **Multi-homing amélioré** : Plusieurs workspaces facilement
- ✅ **Filtering** : Filtrage données avant envoi (économies)
- ✅ **Security** : Managed Identity support
- ✅ **Performance** : Meilleure efficacité réseau

**Migration MMA → AMA :**
```powershell
# Vérifier si MMA installé
Get-AzVMExtension -ResourceGroupName "myRG" -VMName "myVM" |
  Where-Object {$_.ExtensionType -eq "MicrosoftMonitoringAgent"}

# Installer AMA
Set-AzVMExtension `
  -ResourceGroupName "myRG" `
  -VMName "myVM" `
  -Name "AzureMonitorWindowsAgent" `
  -Publisher "Microsoft.Azure.Monitor" `
  -ExtensionType "AzureMonitorWindowsAgent" `
  -TypeHandlerVersion "1.0"

# Créer et associer DCR (voir exemple ci-dessus)

# Désinstaller MMA après validation
Remove-AzVMExtension `
  -ResourceGroupName "myRG" `
  -VMName "myVM" `
  -Name "MicrosoftMonitoringAgent" `
  -Force
```

#### VM Insights - Monitoring Avancé des VMs

**⚠️ Fonctionnalité Importante pour l'Examen**

VM Insights fournit un monitoring complet des VMs avec métriques de performance, dépendances d'applications et health monitoring.

**Composants Requis :**
1. **Azure Monitor Agent (AMA)** - Recommandé
   - **Avec AMA moderne** : Dependency Agent intégré (pas d'extension séparée nécessaire)
   - **Avec Log Analytics Agent (Legacy)** : Dependency Agent requis comme extension séparée
2. **Log Analytics Workspace**
3. **Data Collection Rule (DCR)** - Si utilisation d'AMA

**⚠️ Note Importante :** Avec Azure Monitor Agent moderne, les fonctionnalités de dépendances (Service Map) sont intégrées directement. L'extension Dependency Agent séparée n'est requise que si vous utilisez encore le Legacy Log Analytics Agent (MMA/OMS).

**Activation via Portal :**
1. VM → **Monitoring** → **Insights**
2. **Enable**
3. Choisir **Log Analytics Workspace**
4. Configuration automatique des agents

**Activation via PowerShell :**
```powershell
# Installer VM Insights avec Azure Monitor Agent
Install-Module -Name Az.MonitoringSolutions

# Enable VM Insights
Set-AzVMExtension `
  -ResourceGroupName "myRG" `
  -VMName "myVM" `
  -Name "DependencyAgentWindows" `
  -Publisher "Microsoft.Azure.Monitoring.DependencyAgent" `
  -ExtensionType "DependencyAgentWindows" `
  -TypeHandlerVersion "9.10"

# Configurer Log Analytics Workspace
$workspace = Get-AzOperationalInsightsWorkspace `
  -ResourceGroupName "myRG" `
  -Name "myWorkspace"

# Activer VM Insights solution
Set-AzOperationalInsightsIntelligencePack `
  -ResourceGroupName "myRG" `
  -WorkspaceName "myWorkspace" `
  -IntelligencePackName "VMInsights" `
  -Enabled $true
```

**Métriques Collectées :**
- **Performance** : CPU, Memory, Disk, Network
- **Processes** : Processus running et connexions
- **Dependencies** : Connexions réseau entre VMs et services externes
- **Logical Disk** : Space, IOPS, throughput
- **Network Adapter** : Bytes sent/received, packets

**Requêtes KQL - VM Insights :**
```kusto
// Top 10 VMs par utilisation CPU
InsightsMetrics
| where Namespace == "Processor" and Name == "UtilizationPercentage"
| summarize avg(Val) by Computer
| top 10 by avg_Val desc

// Connexions réseau actives
VMConnection
| where TimeGenerated > ago(1h)
| summarize count() by Computer, RemoteIp, Direction
| order by count_ desc

// Processus consommant le plus de mémoire
VMProcess
| where TimeGenerated > ago(1h)
| summarize avg(WorkingSetMemory) by ProcessName, Computer
| top 10 by avg_WorkingSetMemory desc

// Services Map - Dépendances
VMBoundPort
| where TimeGenerated > ago(24h)
| where Machine == "myVM"
| summarize count() by Port, ProcessName
| order by count_ desc
```

**Service Map - Visualisation des Dépendances :**
- **Inbound connections** : Clients se connectant à la VM
- **Outbound connections** : Services externes utilisés
- **Ports** : Ports d'écoute et connexions actives
- **Failed connections** : Tentatives de connexion échouées

**Use Cases VM Insights :**
- 🔍 **Troubleshooting performance** : Identifier bottlenecks CPU/Memory/Disk
- 🗺️ **Application mapping** : Comprendre architecture et dépendances
- 📊 **Capacity planning** : Dimensionnement VMs
- 🚨 **Alerting** : Alertes sur seuils performance
- 🔒 **Security** : Détecter connexions non autorisées

#### Application Insights - APM pour Applications

**⚠️ Pour l'Examen : Application Insights fait partie d'Azure Monitor**

**Types d'Applications Supportées :**
- .NET, .NET Core, Java, Node.js, Python, JavaScript
- App Service, Azure Functions, VMs, On-premises, Kubernetes

**Installation - App Service :**
```bash
# Activer Application Insights sur App Service
az webapp config appsettings set \
  --resource-group myRG \
  --name myWebApp \
  --settings APPINSIGHTS_INSTRUMENTATIONKEY={instrumentation-key}

# Ou via connection string (recommandé)
az webapp config appsettings set \
  --resource-group myRG \
  --name myWebApp \
  --settings APPLICATIONINSIGHTS_CONNECTION_STRING="InstrumentationKey={key};IngestionEndpoint=https://eastus-8.in.applicationinsights.azure.com/"
```

**Installation - .NET Application :**
```bash
# NuGet package
dotnet add package Microsoft.ApplicationInsights.AspNetCore
```

**Configuration - appsettings.json :**
```json
{
  "ApplicationInsights": {
    "ConnectionString": "InstrumentationKey={key};IngestionEndpoint=https://eastus-8.in.applicationinsights.azure.com/"
  },
  "Logging": {
    "ApplicationInsights": {
      "LogLevel": {
        "Default": "Information",
        "Microsoft": "Warning"
      }
    }
  }
}
```

**Données Collectées :**
- **Requests** : HTTP requests, response times, URLs
- **Dependencies** : SQL, HTTP, Redis, Azure services
- **Exceptions** : Stack traces, error messages
- **Page Views** : Client-side navigation
- **Custom Events** : Business metrics, user actions
- **Metrics** : Performance counters, custom metrics
- **Traces** : Logs applicatifs

**Requêtes KQL - Application Insights :**
```kusto
// Requests avec temps de réponse > 1s
requests
| where timestamp > ago(1h)
| where duration > 1000
| project timestamp, name, url, duration, resultCode
| order by duration desc

// Top 10 exceptions
exceptions
| where timestamp > ago(24h)
| summarize count() by type, outerMessage
| top 10 by count_ desc

// Dépendances SQL lentes
dependencies
| where timestamp > ago(1h)
| where type == "SQL"
| where duration > 500
| project timestamp, name, data, duration, success
| order by duration desc

// Availability (tests synthétiques)
availabilityResults
| where timestamp > ago(24h)
| summarize SuccessRate = 100.0 * countif(success == true) / count() by name
| order by SuccessRate asc
```

**Smart Detection - Alertes Automatiques :**
- **Failure anomalies** : Augmentation soudaine des échecs
- **Performance anomalies** : Dégradation temps de réponse
- **Memory leak detection** : Fuite mémoire détectée
- **Slow page load time** : Chargement page lent
- **Exception volume** : Volume d'exceptions anormal

**Application Map :**
```
[Client Browser]
    ↓
[App Service: myWebApp] → [SQL Database: myDB]
    ↓                        ↓
[Redis Cache]            [Slow queries detected]
    ↓
[Azure Storage]
```

### 5.2 Azure Backup

#### Recovery Services Vault - Fondamentaux

**⚠️ Règles Critiques pour l'Examen AZ-104**

**Localisation :**
- Vault et ressources **dans la même région obligatoire**
- Example : Vault2 (West US) peut sauvegarder Storage1 (West US)
- Share1 sauvegardable car dans Storage1 même région
- **Cross-region restore** : Possible si GRS activé sur vault

**Création via Azure CLI :**
```bash
# Créer Recovery Services Vault
az backup vault create \
  --resource-group myRG \
  --name myVault \
  --location eastus

# Configurer redundancy (LRS ou GRS)
az backup vault backup-properties set \
  --resource-group myRG \
  --name myVault \
  --backup-storage-redundancy GeoRedundant
```

**Création via PowerShell :**
```powershell
# Créer Resource Group
New-AzResourceGroup -Name "myRG" -Location "East US"

# Créer Recovery Services Vault
New-AzRecoveryServicesVault `
  -ResourceGroupName "myRG" `
  -Name "myVault" `
  -Location "East US"

# Obtenir le vault context
$vault = Get-AzRecoveryServicesVault `
  -ResourceGroupName "myRG" `
  -Name "myVault"

# Configurer Storage Redundancy
Set-AzRecoveryServicesBackupProperty `
  -Vault $vault `
  -BackupStorageRedundancy GeoRedundant
```

**Storage Redundancy Options :**

| Type | Copies | Cross-Region | Use Case | Coût |
|------|--------|--------------|----------|------|
| **LRS** | 3 copies (même datacenter) | Non | Dev/Test, données non critiques | Faible |
| **ZRS** | 3 copies (3 zones) | Non | Production, haute disponibilité | Moyen |
| **GRS** | 6 copies (primaire + secondaire) | Oui | Données critiques, DR | Élevé |

**⚠️ Important - Changement de Storage Redundancy :**
- **Recommandation** : Configurer AVANT le premier backup pour éviter les complications
- **Depuis 2023** : Microsoft permet le changement APRÈS le premier backup avec limitations :
  - Fenêtres de maintenance spécifiques requises
  - Peut nécessiter l'arrêt temporaire des backups
  - Certaines restrictions selon le type de workload
  - Processus plus complexe et risqué
- **Best Practice** : Toujours planifier la redundancy correcte dès la création du vault

**Suppression du Vault - Procédure Correcte :**

```powershell
# 1. Lister tous les éléments protégés
$vault = Get-AzRecoveryServicesVault -ResourceGroupName "myRG" -Name "myVault"
Set-AzRecoveryServicesVaultContext -Vault $vault

$containers = Get-AzRecoveryServicesBackupContainer -ContainerType AzureVM -VaultId $vault.ID

# 2. Arrêter backup et supprimer données pour chaque VM
foreach ($container in $containers) {
    $backupItems = Get-AzRecoveryServicesBackupItem `
        -Container $container `
        -WorkloadType AzureVM `
        -VaultId $vault.ID
    
    foreach ($item in $backupItems) {
        # Désactiver soft delete si activé
        Disable-AzRecoveryServicesBackupProtection `
            -Item $item `
            -VaultId $vault.ID `
            -RemoveRecoveryPoints `
            -Force
    }
}

# 3. Désactiver soft delete au niveau vault
Set-AzRecoveryServicesVaultProperty `
    -VaultId $vault.ID `
    -SoftDeleteFeatureState Disable

# 4. Supprimer le vault
Remove-AzRecoveryServicesVault -Vault $vault -Force
```

**Erreurs Courantes :**
❌ **Vault ne peut pas être supprimé avec éléments protégés**
✅ Arrêter backup + supprimer recovery points d'abord

❌ **Soft delete empêche suppression**
✅ Désactiver soft delete au niveau vault

#### Soft Delete - Protection Contre Suppression Accidentelle

**⚠️ Activé par Défaut depuis 2020**

**Fonctionnement :**
- Backup supprimé → **Retained 14 jours** en soft delete state
- Restauration possible pendant 14 jours
- Aucun coût supplémentaire pendant soft delete
- Après 14 jours → Suppression définitive

**Configuration via PowerShell :**
```powershell
# Activer soft delete
Set-AzRecoveryServicesVaultProperty `
    -VaultId $vault.ID `
    -SoftDeleteFeatureState Enable

# Désactiver soft delete (déconseillé)
Set-AzRecoveryServicesVaultProperty `
    -VaultId $vault.ID `
    -SoftDeleteFeatureState Disable

# Enhanced soft delete (always-on, ne peut pas être désactivé)
Set-AzRecoveryServicesVaultProperty `
    -VaultId $vault.ID `
    -SoftDeleteFeatureState AlwaysOn
```

**Restaurer depuis Soft Delete :**
```powershell
# Undelete backup item
$softDeletedItems = Get-AzRecoveryServicesBackupItem `
    -BackupManagementType AzureVM `
    -WorkloadType AzureVM `
    -VaultId $vault.ID `
    -DeleteState ToBeDeleted

Undo-AzRecoveryServicesBackupItemDeletion `
    -Item $softDeletedItems[0] `
    -VaultId $vault.ID `
    -Force
```

**Security PIN :**
- Protection supplémentaire pour opérations critiques
- Requis pour : Disable backup, Change passphrase, Unregister server

```powershell
# Générer Security PIN (valide 5 minutes)
Get-AzRecoveryServicesVaultSettingsFile `
    -Vault $vault `
    -Path "C:\Temp\"

# Le PIN sera affiché dans le portail
```

#### Backup Policies - Stratégies de Sauvegarde

**⚠️ Limites par Type de Ressource :**
- **VMs Azure** : Maximum 100 VMs par policy
- **SQL in Azure VM** : Policy séparée requise
- **SAP HANA in Azure VM** : Policy séparée requise
- **Azure Files** : Policy séparée requise

**Example :** 100 VMs + 20 SQL + 50 Files = **3 policies minimum**

**Créer Backup Policy pour Azure VMs :**

```powershell
# Policy avec backup quotidien + rétention
$schedulePolicy = Get-AzRecoveryServicesBackupSchedulePolicyObject -WorkloadType AzureVM
$schedulePolicy.ScheduleRunFrequency = "Daily"
$schedulePolicy.ScheduleRunTimes[0] = "2025-10-30T02:00:00Z"

$retentionPolicy = Get-AzRecoveryServicesBackupRetentionPolicyObject -WorkloadType AzureVM
$retentionPolicy.DailySchedule.DurationCountInDays = 30
$retentionPolicy.WeeklySchedule.DurationCountInWeeks = 12
$retentionPolicy.MonthlySchedule.DurationCountInMonths = 12
$retentionPolicy.YearlySchedule.DurationCountInYears = 5

New-AzRecoveryServicesBackupProtectionPolicy `
    -Name "DailyBackupPolicy" `
    -WorkloadType AzureVM `
    -RetentionPolicy $retentionPolicy `
    -SchedulePolicy $schedulePolicy `
    -VaultId $vault.ID
```

**Types de Backup Policy :**

**1. Standard Policy (Default) :**
- Backup quotidien à heure fixe
- Snapshots locaux (Instant Restore)
- Transfer vers vault après 2-5 jours
- Rétention : 30 jours à 10 ans

**2. Enhanced Policy :**
- Multiple backups par jour (jusqu'à 4x/jour)
- Granularité horaire
- Use case : Bases de données critiques

**Configuration Policy via CLI :**
```bash
# Créer policy avec rétention 90 jours
az backup policy create \
  --resource-group myRG \
  --vault-name myVault \
  --name DailyBackup90Days \
  --policy '{
    "schedulePolicy": {
      "schedulePolicyType": "SimpleSchedulePolicy",
      "scheduleRunFrequency": "Daily",
      "scheduleRunTimes": ["2025-10-30T02:00:00.000Z"]
    },
    "retentionPolicy": {
      "retentionPolicyType": "LongTermRetentionPolicy",
      "dailySchedule": {
        "retentionTimes": ["2025-10-30T02:00:00.000Z"],
        "retentionDuration": {
          "count": 90,
          "durationType": "Days"
        }
      }
    }
  }'
```

**Rétention Granulaire :**

| Fréquence | Rétention Max | Use Case |
|-----------|---------------|----------|
| **Daily** | 9999 jours (27 ans) | Logs quotidiens |
| **Weekly** | 5163 semaines (~99 ans) | Point de restauration hebdomadaire |
| **Monthly** | 1188 mois (~99 ans) | Archivage mensuel |
| **Yearly** | 99 ans | Conformité long terme |

**Example - Policy Complexe :**
```
Daily   : 30 jours (dernier backup de chaque jour)
Weekly  : 12 semaines (backup du dimanche)
Monthly : 60 mois (backup du premier dimanche du mois)
Yearly  : 10 ans (backup du 1er janvier)
```

#### Azure VM Backup - Configuration Complète

**⚠️ Processus Important pour l'Examen**

**Architecture Backup VM :**
```
Azure VM
├── Snapshot Tier (Instant Restore)
│   ├── Stockage : Managed Disks snapshots
│   ├── Localisation : Même région que VM
│   ├── Rétention : 1-5 jours (configurable)
│   └── Restore : < 5 minutes
│
└── Vault-Standard Tier
    ├── Stockage : Recovery Services Vault
    ├── Localisation : Région vault
    ├── Rétention : 30 jours à 99 ans
    └── Restore : 30 minutes - 6 heures
```

**Activer Backup sur VM Existante :**

```powershell
# Sélectionner vault et policy
$vault = Get-AzRecoveryServicesVault -ResourceGroupName "myRG" -Name "myVault"
Set-AzRecoveryServicesVaultContext -Vault $vault

$policy = Get-AzRecoveryServicesBackupProtectionPolicy `
    -Name "DailyBackupPolicy" `
    -VaultId $vault.ID

# Activer backup sur VM
Enable-AzRecoveryServicesBackupProtection `
    -ResourceGroupName "myRG" `
    -Name "myVM" `
    -Policy $policy `
    -VaultId $vault.ID
```

```bash
# Via Azure CLI
az backup protection enable-for-vm \
  --resource-group myRG \
  --vault-name myVault \
  --vm myVM \
  --policy-name DailyBackupPolicy
```

**Backup On-Demand (Immediate) :**
```powershell
# Trigger backup manuel
$backupContainer = Get-AzRecoveryServicesBackupContainer `
    -ContainerType AzureVM `
    -FriendlyName "myVM" `
    -VaultId $vault.ID

$backupItem = Get-AzRecoveryServicesBackupItem `
    -Container $backupContainer `
    -WorkloadType AzureVM `
    -VaultId $vault.ID

Backup-AzRecoveryServicesBackupItem `
    -Item $backupItem `
    -VaultId $vault.ID `
    -ExpiryDateTimeUTC (Get-Date).AddDays(30)
```

**Instant Restore Feature :**
- Snapshots stockés localement (managed disk snapshots)
- Restore ultra-rapide : 5-10 minutes
- Rétention configurable : 1-5 jours
- Coût : Snapshot storage (environ $0.05/GB/mois)

**Configuration Instant Restore :**
```powershell
# Modifier rétention instant restore (1-5 jours)
$policy = Get-AzRecoveryServicesBackupProtectionPolicy `
    -Name "DailyBackupPolicy" `
    -VaultId $vault.ID

$policy.SnapshotRetentionInDays = 5

Set-AzRecoveryServicesBackupProtectionPolicy `
    -Policy $policy `
    -VaultId $vault.ID
```

**Types de Restore VM :**

**1. Create New VM (Restore as new) :**
```powershell
# Restaurer comme nouvelle VM
$rp = Get-AzRecoveryServicesBackupRecoveryPoint `
    -Item $backupItem `
    -VaultId $vault.ID `
    | Select-Object -First 1

$restoreConfig = Get-AzRecoveryServicesBackupWorkloadRecoveryConfig `
    -RecoveryPoint $rp `
    -VaultId $vault.ID `
    -TargetResourceGroupName "myRestoredRG"

Restore-AzRecoveryServicesBackupItem `
    -WLRecoveryConfig $restoreConfig `
    -VaultId $vault.ID
```

**2. Restore Disks Only :**
```powershell
# Restaurer uniquement les disques
$restoreConfig = Get-AzRecoveryServicesBackupWorkloadRecoveryConfig `
    -RecoveryPoint $rp `
    -RestoreAsUnmanagedDisks `
    -VaultId $vault.ID `
    -TargetStorageAccountName "myrestoresa"

Restore-AzRecoveryServicesBackupItem `
    -WLRecoveryConfig $restoreConfig `
    -VaultId $vault.ID
```

**3. Replace Existing Disks :**
```powershell
# Remplacer disques VM existante
$restoreConfig = Get-AzRecoveryServicesBackupWorkloadRecoveryConfig `
    -RecoveryPoint $rp `
    -RestoreOnlyOSDisk `
    -VaultId $vault.ID

Restore-AzRecoveryServicesBackupItem `
    -WLRecoveryConfig $restoreConfig `
    -VaultId $vault.ID
```

**4. File-Level Recovery (ILR - Item Level Recovery) :**
```powershell
# Monter recovery point comme disque virtuel
$rp = Get-AzRecoveryServicesBackupRecoveryPoint `
    -Item $backupItem `
    -VaultId $vault.ID `
    | Select-Object -First 1

# Générer script de montage
Get-AzRecoveryServicesBackupRPMountScript `
    -RecoveryPoint $rp `
    -VaultId $vault.ID

# Le script PowerShell téléchargé montera le recovery point
# Copier fichiers nécessaires manuellement
# Puis démonter :

Disable-AzRecoveryServicesBackupRPMountScript `
    -RecoveryPoint $rp `
    -VaultId $vault.ID
```

#### Changement de Vault

**Pour Backup :**
```powershell
# 1. Arrêter backup dans vault actuel (RSV1)
$vault1 = Get-AzRecoveryServicesVault -Name "RSV1"
Set-AzRecoveryServicesVaultContext -Vault $vault1

$backupItem = Get-AzRecoveryServicesBackupItem `
    -BackupManagementType AzureVM `
    -WorkloadType AzureVM `
    -Name "myVM" `
    -VaultId $vault1.ID

Disable-AzRecoveryServicesBackupProtection `
    -Item $backupItem `
    -VaultId $vault1.ID `
    -RemoveRecoveryPoints `
    -Force

# 2. Configurer backup dans nouveau vault (RSV2)
$vault2 = Get-AzRecoveryServicesVault -Name "RSV2"
Set-AzRecoveryServicesVaultContext -Vault $vault2

$policy2 = Get-AzRecoveryServicesBackupProtectionPolicy `
    -Name "DailyBackupPolicy" `
    -VaultId $vault2.ID

Enable-AzRecoveryServicesBackupProtection `
    -ResourceGroupName "myRG" `
    -Name "myVM" `
    -Policy $policy2 `
    -VaultId $vault2.ID
```

**⚠️ Important :** Changement de vault = Perte des recovery points existants

**Pour Site Recovery :**
- VM → Disaster Recovery → Replication Settings → Nouveau vault
- Moins destructif que Backup (replication continuous)

#### Types de Backup Détaillés

**1. Azure VMs - Application-Consistent Backups**

**Consistency Levels :**

| Type | Windows | Linux | Use Case |
|------|---------|-------|----------|
| **Application-consistent** | VSS enabled | Pre/Post scripts | Production databases |
| **File-system consistent** | VSS failed | Default | File servers |
| **Crash-consistent** | VM powered off | VM powered off | Graceful shutdown non requis |

**VSS (Volume Shadow Copy Service) - Windows :**
```
1. VM Backup triggered
2. Azure Backup calls VSS Writer
3. Applications (SQL, Exchange) flush data to disk
4. Snapshot created (application-consistent)
5. Applications resume normal operations
```

**Pre/Post Scripts - Linux :**
```bash
# /etc/azure/pre-script.sh
#!/bin/bash
# Flush MySQL buffers avant snapshot
mysql -u root -p${MYSQL_PASSWORD} -e "FLUSH TABLES WITH READ LOCK;"

# /etc/azure/post-script.sh
#!/bin/bash
# Unlock MySQL après snapshot
mysql -u root -p${MYSQL_PASSWORD} -e "UNLOCK TABLES;"
```

**Configuration Scripts :**
```json
{
  "ScriptExecutionPolicy": "Custom",
  "PreScriptLocation": "/etc/azure/pre-script.sh",
  "PostScriptLocation": "/etc/azure/post-script.sh",
  "PreScriptTimeout": "300",
  "PostScriptTimeout": "300"
}
```

**2. On-Premises Backup - MARS Agent**

**MARS (Microsoft Azure Recovery Services) Agent :**
- Backup fichiers/dossiers Windows vers Azure
- Pas de backup système complet (OS)
- Backup granulaire au niveau fichier

**Installation MARS Agent :**
```powershell
# Télécharger MARS agent
$url = "https://aka.ms/azurebackup_agent"
$output = "C:\Temp\MARSAgentInstaller.exe"
Invoke-WebRequest -Uri $url -OutFile $output

# Installer
Start-Process -FilePath $output -ArgumentList "/q /nu" -Wait

# Télécharger vault credentials depuis portal
# Azure Backup → Recovery Services Vault → Backup Infrastructure → Download Vault Credentials

# Enregistrer serveur avec vault
& "C:\Program Files\Microsoft Azure Recovery Services Agent\bin\wabadmin" `
    Start RegisterWithVault `
    -VaultCredentialsFilePath "C:\Temp\VaultCreds.vaultcredentials" `
    -EncryptionPassphrase "MyStrongPassphrase123!"
```

**Configurer Backup Schedule :**
```powershell
# Via MARS agent GUI ou PowerShell
$policy = New-OBPolicy

# Ajouter fichiers/dossiers
$fileSpec = New-OBFileSpec -FileSpec "C:\ImportantData" -Recurse

Add-OBFileSpec -Policy $policy -FileSpec $fileSpec

# Schedule (quotidien 2AM)
$schedule = New-OBSchedule -DaysOfWeek Monday,Tuesday,Wednesday,Thursday,Friday,Saturday,Sunday -TimesOfDay 02:00

Set-OBSchedule -Policy $policy -Schedule $schedule

# Rétention (30 jours)
$retention = New-OBRetentionPolicy -RetentionDays 30
Set-OBRetentionPolicy -Policy $policy -RetentionPolicy $retention

# Appliquer policy
Set-OBPolicy -Policy $policy -Confirm:$false
```

**3. SQL Server in Azure VM - Database Backup**

**⚠️ Backup Spécifique SQL**

**Features :**
- **Full, Differential, Log backups**
- **Point-in-time restore** (15 minutes granularity)
- **Auto-protection** : Nouvelles databases automatiquement sauvegardées
- **Compression** : Backups compressés

**Activer SQL Backup :**
```powershell
# Découvrir SQL instances
Register-AzRecoveryServicesBackupContainer `
    -ResourceGroupName "myRG" `
    -VaultId $vault.ID `
    -ContainerType AzureVMAppContainer `
    -FriendlyName "myVM"

# Lister SQL databases
Get-AzRecoveryServicesBackupProtectableItem `
    -Container $container `
    -WorkloadType MSSQL `
    -VaultId $vault.ID

# Activer backup
Enable-AzRecoveryServicesBackupProtection `
    -ResourceGroupName "myRG" `
    -Name "myVM;mssqlserver;myDB" `
    -Policy $sqlPolicy `
    -VaultId $vault.ID
```

**SQL Backup Policy Example :**
```
Full Backup    : Weekly (Sunday 2AM)
Differential   : Daily (except Sunday, 2AM)
Log Backup     : Every 15 minutes
Rétention Full : 12 weeks
Rétention Diff : 30 jours
Rétention Log  : 7 jours
```

**4. Azure Files - Share Snapshots**

**⚠️ Snapshots Natifs Azure Files**

**Caractéristiques :**
- Snapshots incrémentiels (seuls blocs modifiés)
- Stockés dans même Storage Account
- Pas de transfert vers vault
- Rétention : 200 snapshots max par share

**Activer Backup Azure Files :**
```bash
az backup protection enable-for-azurefileshare \
  --resource-group myRG \
  --vault-name myVault \
  --storage-account myStorageAccount \
  --azure-file-share myFileShare \
  --policy-name DailyFileShareBackup
```

```powershell
Enable-AzRecoveryServicesBackupProtection `
    -StorageAccountName "mystorageaccount" `
    -Name "myfileshare" `
    -Policy $policy `
    -VaultId $vault.ID
```

**Restore Azure Files :**
```powershell
# Restore entier share
$rp = Get-AzRecoveryServicesBackupRecoveryPoint `
    -Item $backupItem `
    -VaultId $vault.ID `
    | Select-Object -First 1

Restore-AzRecoveryServicesBackupItem `
    -RecoveryPoint $rp `
    -TargetStorageAccountName "myrestoredsa" `
    -TargetFileShareName "myrestoredshare" `
    -VaultId $vault.ID

# Restore fichiers individuels
$rp = Get-AzRecoveryServicesBackupRecoveryPoint `
    -Item $backupItem `
    -VaultId $vault.ID

# Portal : Browse files → Select files → Restore
```

#### Backup Reports et Monitoring

**⚠️ Azure Backup Reports utilise Azure Monitor Workbooks**

**Configuration Backup Reports :**

```powershell
# Configurer Diagnostic Settings pour envoyer backup data vers Log Analytics
$vault = Get-AzRecoveryServicesVault -Name "myVault"

Set-AzDiagnosticSetting `
    -ResourceId $vault.ID `
    -Name "BackupReports" `
    -WorkspaceId "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.OperationalInsights/workspaces/myWorkspace" `
    -Enabled $true `
    -Category @("AzureBackupReport", "CoreAzureBackup", "AddonAzureBackupJobs", "AddonAzureBackupAlerts", "AddonAzureBackupPolicy", "AddonAzureBackupStorage", "AddonAzureBackupProtectedInstance")
```

**Via Azure CLI :**
```bash
az monitor diagnostic-settings create \
  --name BackupReports \
  --resource /subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.RecoveryServices/vaults/myVault \
  --workspace /subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.OperationalInsights/workspaces/myWorkspace \
  --logs '[
    {"category": "AzureBackupReport", "enabled": true},
    {"category": "CoreAzureBackup", "enabled": true},
    {"category": "AddonAzureBackupJobs", "enabled": true},
    {"category": "AddonAzureBackupAlerts", "enabled": true}
  ]'
```

**Requêtes KQL - Backup Monitoring :**

```kusto
// Jobs de backup échoués dernières 24h
AddonAzureBackupJobs
| where TimeGenerated > ago(24h)
| where JobStatus == "Failed"
| project TimeGenerated, BackupItemFriendlyName, JobOperation, ErrorDetails
| order by TimeGenerated desc

// Taille totale des backups par VM
AddonAzureBackupStorage
| where TimeGenerated > ago(1d)
| summarize TotalBackupSizeGB = sum(StorageConsumedInMBs)/1024 by BackupItemFriendlyName
| order by TotalBackupSizeGB desc

// Compliance backup (VMs sans backup configuré)
CoreAzureBackup
| where TimeGenerated > ago(7d)
| where OperationName == "BackupItem"
| summarize LastBackup = max(TimeGenerated) by BackupItemFriendlyName
| where LastBackup < ago(2d)
| project BackupItemFriendlyName, DaysSinceLastBackup = datetime_diff('day', now(), LastBackup)

// Alertes backup critiques
AddonAzureBackupAlerts
| where TimeGenerated > ago(7d)
| where AlertSeverity == "Critical"
| summarize count() by AlertCode, BackupItemFriendlyName
| order by count_ desc
```

**Métriques Importantes pour Monitoring :**

| Métrique | Description | Seuil Critique |
|----------|-------------|----------------|
| **Backup Health** | % VMs avec backup réussi | < 95% |
| **Restore Testing** | Dernière restore test | > 30 jours |
| **Storage Growth** | Croissance mensuelle stockage | > 20%/mois |
| **Job Duration** | Durée moyenne backup jobs | > 4 heures |
| **Failed Jobs** | Jobs échoués dernières 24h | > 0 |

**Alertes Backup Recommandées :**

```powershell
# Alerte pour backup jobs échoués
$actionGroup = Get-AzActionGroup -ResourceGroupName "myRG" -Name "BackupAlerts"

$condition = New-AzScheduledQueryRuleCondition `
    -Query "AddonAzureBackupJobs | where JobStatus == 'Failed'" `
    -TimeAggregation "Count" `
    -Operator "GreaterThan" `
    -Threshold 0

New-AzScheduledQueryRule `
    -ResourceGroupName "myRG" `
    -Location "eastus" `
    -Name "BackupJobsFailed" `
    -DisplayName "Backup Jobs Failed Alert" `
    -Description "Alert when backup jobs fail" `
    -Severity 2 `
    -Enabled $true `
    -EvaluationFrequency "PT5M" `
    -WindowSize "PT5M" `
    -TargetResourceId "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.OperationalInsights/workspaces/myWorkspace" `
    -Condition $condition `
    -ActionGroupId $actionGroup.Id
```

#### Scénarios d'Examen - Azure Backup

**⚠️ Questions Typiques AZ-104**

| Scénario | Solution | Justification |
|----------|----------|---------------|
| **Vault dans West US, VM dans East US** | ❌ Impossible de configurer backup | Vault et ressources doivent être dans même région |
| **100 VMs + 20 SQL databases à sauvegarder** | 2 policies minimum (1 VM, 1 SQL) | Policies séparées par workload type |
| **Besoin restore VM < 5 minutes** | Instant Restore activé (rétention 1-5 jours) | Snapshots locaux pour restore rapide |
| **Soft delete activé, backup supprimé** | Recovery possible 14 jours | Soft delete = protection 14 jours |
| **Changer LRS vers GRS** | Impossible si backups existants | Storage redundancy modifiable AVANT premier backup |
| **Backup SQL avec RPO 15 min** | SQL in Azure VM backup (log backups 15 min) | Point-in-time restore granularité 15 min |
| **Restore uniquement certains fichiers VM** | ILR (Item Level Recovery) | Montage recovery point comme disque virtuel |
| **On-premises Windows files vers Azure** | MARS Agent | Backup granulaire fichiers/dossiers |
| **Supprimer Recovery Services Vault** | Stop backup tous items + disable soft delete | Vault ne peut être supprimé avec items protégés |
| **DR vers autre région Azure** | GRS redundancy + Cross-region restore | Backup répliqué vers région secondaire |

**Troubleshooting Backup - Erreurs Courantes :**

| Erreur | Cause | Solution |
|--------|-------|---------|
| **UserErrorVmProvisioningStateFailed** | VM deallocated/stopped | Démarrer la VM |
| **ExtensionSnapshotFailedNoNetwork** | VM sans accès Internet | Vérifier NSG outbound rules |
| **UserErrorVaultMaximumResourcesReached** | Limite 1000 items par vault | Créer nouveau vault ou nettoyer ancien backup |
| **BackupOperationFailed** | Permissions insuffisantes | VM Contributor + Backup Contributor sur vault |
| **ExtensionInstallationFailure** | Extension conflict | Désinstaller extensions anciennes (MMA si AMA installé) |
| **GuestAgentStatusUnavailable** | VM Agent non running | Installer/réparer Azure VM Agent |
| **UserErrorBackupNotEnabledOnVm** | Backup jamais configuré | Enable backup via Recovery Services Vault |

**Best Practices - Azure Backup :**

✅ **À FAIRE :**
- **GRS redundancy** pour données critiques (DR)
- **Soft delete always-on** pour protection maximale
- **Instant Restore 5 jours** pour recovery rapide
- **Test régulièrement les restores** (au moins trimestriel)
- **Backup reports** vers Log Analytics pour monitoring
- **Action Groups** pour alertes backup échoués
- **Tags** sur backup items pour organisation/coûts
- **Rétention appropriée** : Conformité vs coûts
- **Application-consistent backups** : VSS (Windows) ou Pre/Post scripts (Linux)

❌ **À ÉVITER :**
- Changer storage redundancy après premier backup
- Oublier de tester les restores (backup = inutile si restore impossible)
- Désactiver soft delete (protection accidentelle)
- Backup sans monitoring/alertes
- Utiliser Legacy agents (MMA) au lieu d'Azure Monitor Agent
- Stocker tous recovery points à vie (coûts prohibitifs)
- Un vault par VM (complexité management)

**Calcul Coûts Backup - Estimation :**

```
Azure VM Backup Coût = Protected Instance + Storage

Protected Instance :
- VM < 50 GB : $10/mois
- VM 50-500 GB : $20/mois  
- VM > 500 GB : $30/mois

Storage :
- Snapshot tier : ~$0.05/GB/mois (instant restore 1-5 jours)
- Vault-Standard tier : ~$0.05/GB/mois (rétention long terme)
- GRS : 2x coût LRS

Example :
- 10 VMs @ 100 GB chacune
- Protected instances : 10 x $20 = $200/mois
- Storage (30 jours rétention, ~10% change rate) :
  - Snapshots (5 jours) : 10 VMs x 100 GB x 0.05 x 5 jours / 30 jours = $8.33
  - Vault (30 jours, incrémental) : 10 VMs x 100 GB x 0.10 x 30 jours x $0.05 = $150
- Total : $200 + $8 + $150 = ~$358/mois
```

**RPO/RTO Backup - Service Level Agreements :**

| Backup Type | RPO (Recovery Point Objective) | RTO (Recovery Time Objective) |
|-------------|--------------------------------|------------------------------|
| **Azure VM (Standard)** | 24 heures (backup quotidien) | 30 min - 6 heures (vault tier) |
| **Azure VM (Instant Restore)** | 24 heures | < 5 minutes (snapshot tier) |
| **SQL in Azure VM (Log backups)** | 15 minutes | 30 min - 2 heures |
| **Azure Files** | 1-24 heures (configurable) | 5-30 minutes (snapshots) |
| **MARS Agent** | 24 heures | 1-4 heures (selon taille) |

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
- [ ] **Action Groups** : Notifications et actions automatiques (email, runbook, webhook, Logic App)
- [ ] **Common Alert Schema** : Uniformité des alertes entre services
- [ ] **Azure Monitor Agent (AMA)** : Agent recommandé (remplace MMA/OMS)
- [ ] **Data Collection Rules (DCR)** : Configuration centralisée pour AMA
- [ ] **VM Insights** : Performance monitoring + Service Map (Dependency Agent requis)
- [ ] **Application Insights** : APM pour applications (part of Azure Monitor)
- [ ] **Soft Delete** : Activé par défaut, rétention 14 jours
- [ ] **Storage Redundancy** : LRS/ZRS/GRS, changeable AVANT premier backup uniquement
- [ ] **Instant Restore** : Snapshots locaux, restore < 5 minutes, rétention 1-5 jours
- [ ] **VSS** : Application-consistent backups pour Windows VMs
- [ ] **MARS Agent** : Backup fichiers/dossiers on-premises vers Azure
- [ ] **SQL in Azure VM** : Full/Diff/Log backups, point-in-time restore (15 min granularity)
- [ ] **Azure Files Backup** : Share snapshots, 200 max par share
- [ ] **Cross-region restore** : Possible si GRS activé sur vault
- [ ] **ILR (Item Level Recovery)** : Restore fichiers individuels depuis VM backup

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
