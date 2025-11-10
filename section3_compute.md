## 3. Deploy and Manage Azure Compute Resources (20-25%)

### 3.1 Virtual Machines

#### VM Sizes et Families
- **General Purpose** (B, D, DC series) : Workloads équilibrés
- **Compute Optimized** (F series) : Applications CPU-intensive
- **Memory Optimized** (E, M, G series) : Bases de données, analytics
- **Storage Optimized** (L series) : Big data, SQL NoSQL
- **GPU** (N series) : IA, machine learning, rendering

#### Disques et Stockage

** Erreur fréquente identifiée :** Disques temporaires vs persistants

**Disque C: (OS Disk)**
- **Type** : Persistant (VHD dans Azure Storage)
- **Contenu** : OS, applications installées
- **Persistance** : Conservé lors redémarrages/arrêts
- **Usage** : Applications, données importantes

**Disque D: (Temporary Disk)**
- **Type** : Temporaire (SSD local hyperviseur)
- **Contenu** : Pagefile.sys par défaut
- **Persistance** : **PERDU** lors maintenances/redéploiements
- **Usage** : Cache, fichiers temporaires, TempDB

**Disques de Données (E:, F:, etc.)**
- **Type** : Persistants (Managed Disks)
- **Usage** : Données applicatives, bases de données

#### Types de Disques
- **Standard HDD** : Workloads peu fréquents, coût minimal
- **Standard SSD** : Workloads modérés, balance performance/coût
- **Premium SSD** : Workloads critiques, haute performance
- **Ultra Disk** : Workloads extrêmes, latence ultra-faible

#### Création de VM avec Data Disks

**⚠️ Erreur Courante QCM : Attacher plusieurs data disks dès la création**

**Scénario :** Créer une VM avec 3 disques de données (data disks) attachés dès la création.

**Solution : Azure CLI (Méthode la plus simple)**

```bash
# Créer une VM avec 3 data disks en une seule commande
az vm create \
  --resource-group myRG \
  --name myVM \
  --image UbuntuLTS \
  --size Standard_D2s_v3 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --data-disk-sizes-gb 128 256 512
  # Créera automatiquement 3 disques: 128GB, 256GB et 512GB
```

**Paramètres clés :**
- `--data-disk-sizes-gb` : Liste des tailles de disques séparées par espaces
- Les disques sont créés automatiquement et attachés
- LUN (Logical Unit Number) assignés automatiquement : 0, 1, 2, etc.

**Concepts Importants :**

**LUN (Logical Unit Number) :**
- **Identifiant unique** pour chaque disque attaché à une VM
- **Numérotation** : Commence à 0 (LUN 0, LUN 1, LUN 2, etc.)
- **Maximum** : Dépend de la taille de VM
  - Standard_D2s_v3 : Max 8 data disks (LUN 0-7)
  - Standard_D16s_v3 : Max 32 data disks (LUN 0-31)

**Persistance des Data Disks :**
- ✅ **PERSISTANTS** : Les données sont conservées lors arrêts/redémarrages
- ✅ **Managed Disks** : Gestion automatique par Azure
- ✅ **Snapshots** : Possibilité de créer des snapshots pour backup
- ❌ **Différent du disque temporaire (D:)** qui est volatile

**Scénarios d'Examen :**

| Question | Réponse | Raison |
|----------|---------|--------|
| Créer VM avec 3 disques de données | `--data-disk-sizes-gb 128 256 512` | Azure CLI option la plus simple |
| Maximum de data disks sur Standard_D2s_v3 | 8 disks | Dépend de la taille de VM |
| Les data disks sont-ils persistants ? | Oui | Contrairement au disque temporaire D: |
| Identifier un data disk dans la VM | Par LUN (0, 1, 2...) | Logical Unit Number |

#### Accès Externe aux VMs

**⚠️ Erreur Courante QCM : Minimize Administrative Effort**

**Scénario :** Une VM est accessible uniquement depuis le réseau interne. Vous devez permettre l'accès depuis Internet avec **effort administratif minimal**.

**❌ FAUX :** Configurer un VPN Site-to-Site
**✅ CORRECT :** **Ajouter une adresse IP publique** à la VM

**Comparaison des Solutions :**

| Solution | Complexité | Coût* | Temps | Effort Admin |
|----------|------------|------|-------|--------------|
| **IP Publique** | ✅ Très faible | ~$3/mois | 2 min | ✅ Minimal |
| **Azure Bastion** | Moyenne | ~$150/mois | 15 min | Moyen |
| **VPN Site-to-Site** | ❌ Élevée | ~$25-150/mois | 1-2h | ❌ Élevé |

**\*Coûts indicatifs** : Prix approximatifs pour la région US East. Les tarifs varient selon les régions et changent fréquemment.

**Solution Recommandée - Ajouter une IP Publique :**

```bash
# 1. Créer une IP publique
az network public-ip create \
  --resource-group myRG \
  --name myVM-PublicIP \
  --sku Standard \
  --allocation-method Static

# 2. Associer l'IP publique à la NIC de la VM
az network nic ip-config update \
  --resource-group myRG \
  --nic-name myVM-NIC \
  --name ipconfig1 \
  --public-ip-address myVM-PublicIP
```

**⚠️ Sécurité - NSG Recommandé :**

```bash
# Limiter l'accès SSH/RDP à votre IP uniquement
az network nsg rule create \
  --resource-group myRG \
  --nsg-name myVM-NSG \
  --name Allow-SSH-MyIP \
  --priority 100 \
  --source-address-prefixes 203.0.113.50/32 \
  --destination-port-ranges 22 \
  --access Allow \
  --protocol Tcp
```

**Pourquoi PAS VPN Site-to-Site ?**
- ❌ **Complexe** : Local Network Gateway, Virtual Network Gateway, Connection
- ❌ **Temps** : 1-2 heures de configuration
- ❌ **Coût** : Gateway ~$25-150/mois
- ⚠️ **Use case** : Connectivité réseau entier, pas une seule VM

**Scénarios d'Examen :**
- **"Minimal administrative effort"** → Add Public IP
- **"Secure access without exposing ports"** → Azure Bastion
- **"Connect entire on-premises network"** → VPN Site-to-Site

#### High Availability

**Availability Sets**

**Définition :**
- **Groupement logique** de VMs pour assurer la haute disponibilité
- **Protection** contre pannes matérielles ET maintenances planifiées
- **Placement** : VMs distribuées sur plusieurs racks physiques
- **Scope** : Même datacenter (single region, single datacenter)
- **SLA** : 99.95% (au moins 1 VM disponible)
- **Gratuit** : Aucun coût supplémentaire pour l'availability set lui-même

**Composants Clés :**

**1. Fault Domains (FD) - Domaines de Défaillance**
- **Définition** : Représente un rack physique dans le datacenter
- **Contenu** : Serveurs, switch réseau, source d'alimentation partagés
- **Maximum** : 3 Fault Domains par availability set
- **Protection** : Panne matérielle (hardware failure, power outage, network switch failure)
- **Distribution** : Azure répartit automatiquement vos VMs sur les FDs

**Visualisation des Fault Domains :**
```
Datacenter Azure
├── Rack 1 (FD 0) - Alimentation A, Switch A
│   ├── VM1
│   └── VM4
├── Rack 2 (FD 1) - Alimentation B, Switch B
│   ├── VM2
│   └── VM5
└── Rack 3 (FD 2) - Alimentation C, Switch C
    ├── VM3
    └── VM6
```

**Scénario de panne FD :**
- **Problème** : Rack 1 perd l'alimentation électrique
- **Impact** : Seules VM1 et VM4 sont affectées
- **Résultat** : VM2, VM3, VM5, VM6 continuent de fonctionner
- **Disponibilité** : Au moins 2/3 des VMs restent disponibles

**2. Update Domains (UD) - Domaines de Mise à Jour**
- **Définition** : Groupe logique de VMs redémarrées ensemble lors de maintenances
- **Par défaut** : **5 Update Domains** (si non spécifié à la création)
- **Configurable** : De 1 à **20 Update Domains maximum**
- **Important** : Configuration définie à la création, **non modifiable après**
- **Protection** : Maintenance planifiée Azure (host OS updates, hardware maintenance)
- **Processus** : Azure redémarre un seul UD à la fois
- **Délai** : 30 minutes minimum entre chaque UD redémarré
- **⚠️ Note** : La valeur par défaut de 5 est suffisante pour la plupart des scénarios

**Visualisation des Update Domains :**
```
Update Domain 0: VM1, VM6
Update Domain 1: VM2, VM7
Update Domain 2: VM3, VM8
Update Domain 3: VM4, VM9
Update Domain 4: VM5, VM10

Maintenance planifiée :
Étape 1: Redémarre UD0 → VM1, VM6 offline, autres OK
[Attend 30 min]
Étape 2: Redémarre UD1 → VM2, VM7 offline, autres OK
[Attend 30 min]
Étape 3: Redémarre UD2 → VM3, VM8 offline, autres OK
...
```

**Scénario de maintenance UD :**
- **Événement** : Maintenance planifiée Azure
- **Processus** : Azure redémarre UD par UD (jamais simultanément)
- **Impact** : Maximum 1/5 (20%) des VMs offline à un moment donné
- **Garantie** : Au moins 80% de capacité maintenue

**3. Combinaison FD + UD - Protection Double**

**Matrice de Distribution (Exemple : 6 VMs, 3 FD, 3 UD) :**
```
              FD 0 (Rack 1)  FD 1 (Rack 2)  FD 2 (Rack 3)
UD 0            VM1             VM2             VM3
UD 1            VM4             VM5             VM6
UD 2            VM7             VM8             VM9
```

**Protection simultanée :**
- **Panne FD 1** : Perd VM2, VM5, VM8 → Garde VM1,3,4,6,7,9
- **Maintenance UD 0** : Redémarre VM1, VM2, VM3 → Garde VM4,5,6,7,8,9
- **Combiné** : Jamais plus d'un FD ET un UD affecté en même temps

**Configuration et Contraintes :**

**Création :**
- **Moment** : Doit être configuré AVANT ou PENDANT la création de la VM
- **Modification** : IMPOSSIBLE de changer l'availability set d'une VM existante
- **Solution** : Recréer la VM si changement nécessaire

**Limitations importantes :**
- **Zone géographique** : Limité à un seul datacenter
- **Maximum VMs** : Limite technique de **200 VMs par availability set**
- **Famille de VM** : Toutes les VMs doivent être de tailles compatibles
- **Managed Disks** : Fortement recommandé (Aligned ou non)
- **Incompatibilité** : Ne peut PAS combiner avec Availability Zones

**Configuration PowerShell :**
```powershell
# Créer l'Availability Set
New-AzAvailabilitySet `
  -ResourceGroupName "myRG" `
  -Name "myAvailabilitySet" `
  -Location "East US" `
  -PlatformFaultDomainCount 3 `
  -PlatformUpdateDomainCount 5 `
  -Sku Aligned

# Créer VM dans l'Availability Set
New-AzVM `
  -ResourceGroupName "myRG" `
  -Name "myVM" `
  -AvailabilitySetName "myAvailabilitySet" `
  ...
```

**Configuration Azure CLI :**
```bash
# Créer l'Availability Set
az vm availability-set create \
  --resource-group myRG \
  --name myAvailabilitySet \
  --platform-fault-domain-count 3 \
  --platform-update-domain-count 5

# Créer VM dans l'Availability Set
az vm create \
  --resource-group myRG \
  --name myVM \
  --availability-set myAvailabilitySet \
  ...
```

**Best Practices :**

1. **Minimum 2 VMs par tier** : Pour bénéficier du SLA 99.95%
2. **Utiliser Managed Disks** : Meilleure résilience (Storage Fault Domains)
3. **Load Balancer** : Combiner avec Azure Load Balancer pour distribution trafic
4. **Même fonction** : Regrouper uniquement VMs ayant le même rôle
5. **Planification** : Configurer dès la création, impossible à modifier après

**Cas d'usage typiques :**
- **Web tier** : Serveurs web dans un availability set
- **App tier** : Serveurs applicatifs dans un autre availability set
- **Data tier** : Serveurs de base de données dans un troisième availability set
- **Séparation** : Un availability set différent par tier applicatif

**Comparaison rapide :**
| Caractéristique | Availability Sets | Availability Zones |
|-----------------|-------------------|-------------------|
| **Protection** | Rack failure | Datacenter failure |
| **Fault Domains** | Max 3 | 3 zones |
| **Scope** | Single datacenter | Multiple datacenters |
| **SLA** | 99.95% | 99.99% |
| **Latence** | Minimale | Légèrement plus élevée |
| **Coût** | Gratuit | Frais de transfert inter-zones |

**Availability Zones**
- **Protection** : Datacenters physiquement séparés
- **SLA** : 99.99%
- **Scope** : Région Azure

** Prérequis identifiés pour Availability Zones :**
- **Managed Disks** : Obligatoire
- **Availability Options** : Doit être configuré à la création

#### Accès Externe aux VMs
** Cas d'usage identifié :** Accès externe avec effort administratif minimal
- **Scénario** : VM interne accessible uniquement depuis le réseau interne, besoin d'accès externe
- **Solution optimale** : Ajouter une adresse IP publique à la VM
- **Avantage** : Configuration simple et directe, effort administratif minimal

** Erreur fréquente identifiée :** Complexité inutile pour accès externe simple
- ** Erreur** : Configurer un VPN Site-to-Site pour un accès externe simple
- ** Correct** : Utiliser une adresse IP publique pour minimiser l'effort administratif
- **Raison** : VPN S2S = Configuration complexe (Local Network Gateway, Connection, etc.)
- **Alternative** : IP publique = Configuration simple et directe

### 3.2 Virtual Machine Scale Sets (VMSS)

#### Concepts Clés
- **Scaling automatique** : Basé sur métriques ou planning
- **Load balancing** : Intégré avec Azure Load Balancer
- **Update management** : Rolling updates avec availability

** Cas d'usage identifié :** Maintenance Azure
- Pour garantir **8 VMs minimum** pendant maintenance
- Créer un **VMSS avec 10 instances**
- Azure maintient une partie, les autres restent disponibles

#### Scaling Policies
- **Manual** : Contrôle manuel du nombre d'instances
- **Automatic** : Basé sur CPU, mémoire, métriques custom
- **Scheduled** : Scaling programmé

### 3.3 App Services

#### Service Plans - Tiers Détaillés

**⚠️ Erreur Courante QCM : Capacités de scaling par tier**

**Comparaison Complète des Tiers :**

| Tier | SKUs | Max Instances | Autoscale | Deployment Slots | Custom Domain | SSL | Prix/mois* |
|------|------|---------------|-----------|------------------|---------------|-----|-----------|
| **Free** | F1 | 1 (partagé) | ❌ Non | ❌ Non | ❌ Non | ❌ Non | Gratuit |
| **Shared** | D1 | 1 (partagé) | ❌ Non | ❌ Non | ✅ Oui | ❌ Non | ~$10 |
| **Basic** | B1-B3 | 3 | ❌ Non | ❌ Non | ✅ Oui | ✅ Oui | ~$55-220 |
| **Standard** | S1-S3 | 10 | ✅ Oui | ✅ 5 slots | ✅ Oui | ✅ Oui | ~$70-280 |
| **Premium** | P1v2-P3v2 | 30 | ✅ Oui | ✅ 20 slots | ✅ Oui | ✅ Oui | ~$150-600 |
| **PremiumV3** | P1v3-P3v3 | 30 | ✅ Oui | ✅ 20 slots | ✅ Oui | ✅ Oui | ~$200-800 |
| **Isolated** | I1-I3 | 100 | ✅ Oui | ✅ 20 slots | ✅ Oui | ✅ Oui | ~$650+ |

**\*Prix indicatifs** : Prix approximatifs pour la région US East. Les tarifs varient selon les régions Azure et changent fréquemment. Consultez la [page officielle de tarification Azure App Service](https://azure.microsoft.com/pricing/details/app-service/) pour les prix actuels de votre région.

**Caractéristiques par Tier :**

**Free / Shared (Development) :**
- **Infrastructure** : Partagée avec autres clients
- **Limitations** : Quotas CPU (60 min/jour pour Free)
- **Downtime** : Possible si app inactive > 20 min
- **Use case** : Développement, POC uniquement
- ❌ **Production** : Non recommandé

**Basic (Small Production) :**
- **Infrastructure** : VMs dédiées
- **Scaling** : Manuel uniquement (1-3 instances)
- **Limitations** : Pas d'autoscale, pas de slots
- **Use case** : Petites apps production, budget limité
- **SLA** : 99.95%

**Standard (Production) :**
- **Infrastructure** : VMs dédiées
- **Scaling** : Manuel + Autoscale (1-10 instances)
- **Features** : Deployment slots (5), Traffic Manager
- **Use case** : Apps production standard
- **SLA** : 99.95%

**Premium (High Performance) :**
- **Infrastructure** : VMs dédiées plus puissantes
- **Scaling** : Manuel + Autoscale (1-30 instances)
- **Features** : Deployment slots (20), VNet integration
- **Use case** : Apps critiques, haute performance
- **SLA** : 99.95%

**Isolated (Enterprise) :**
- **Infrastructure** : App Service Environment (ASE) dédié
- **Isolation** : Réseau isolé, pas de partage
- **Scaling** : Jusqu'à 100 instances
- **Use case** : Conformité, sécurité maximale
- **SLA** : 99.95%

#### App Service Plan Scaling - Détaillé

**⚠️ Erreur Courante QCM : Scale Up vs Scale Out**

**Types de Scaling Disponibles :**

**1. Scale Up (Vertical Scaling) - Changer de Tier/SKU**

**Définition :** Augmenter les ressources (CPU, RAM, Storage) de chaque instance

**Quand utiliser :**
- Application nécessite plus de CPU/RAM
- Atteinte des limites de l'instance actuelle
- Besoin de nouvelles fonctionnalités (ex: deployment slots)

**Exemple - Azure CLI :**
```bash
# Upgrader de B1 vers S1
az appservice plan update \
  --name myAppServicePlan \
  --resource-group myRG \
  --sku S1

# Upgrader vers Premium
az appservice plan update \
  --name myAppServicePlan \
  --resource-group myRG \
  --sku P1V2
```

**Impact :**
- ⏱️ **Downtime** : Bref redémarrage (quelques secondes)
- 💰 **Coût** : Augmente selon le nouveau tier
- 🎯 **Performance** : Chaque instance plus puissante

**2. Scale Out (Horizontal Scaling) - Ajouter des Instances**

**Définition :** Augmenter le nombre d'instances (VMs) qui exécutent l'application

**Quand utiliser :**
- Trafic élevé, besoin de plus de capacité
- Haute disponibilité requise
- Distribution de charge

**Exemple - Azure CLI :**
```bash
# Scale Out manuel - Ajouter 5 instances
az appservice plan update \
  --name myAppServicePlan \
  --resource-group myRG \
  --number-of-workers 5
```

**Impact :**
- ⏱️ **Downtime** : ✅ Aucun
- 💰 **Coût** : Proportionnel au nombre d'instances
- 🎯 **Performance** : Capacité totale multipliée

**3. Autoscale (Automatic Horizontal Scaling)**

**Disponible à partir de :** Standard (S1) et supérieur

**Configuration Autoscale :**
```bash
# Créer une règle d'autoscale
az monitor autoscale create \
  --resource-group myRG \
  --resource myAppServicePlan \
  --resource-type Microsoft.Web/serverFarms \
  --name myAutoscaleRule \
  --min-count 2 \
  --max-count 10 \
  --count 2

# Règle basée sur CPU (Scale Out)
az monitor autoscale rule create \
  --resource-group myRG \
  --autoscale-name myAutoscaleRule \
  --condition "CpuPercentage > 75 avg 5m" \
  --scale out 1

# Règle basée sur CPU (Scale In)
az monitor autoscale rule create \
  --resource-group myRG \
  --autoscale-name myAutoscaleRule \
  --condition "CpuPercentage < 25 avg 5m" \
  --scale in 1
```

**Métriques disponibles pour Autoscale :**
- **CPU Percentage** : % utilisation CPU
- **Memory Percentage** : % utilisation RAM
- **Disk Queue Length** : File d'attente disque
- **Http Queue Length** : File d'attente requêtes HTTP
- **Data In/Out** : Bande passante réseau
- **Custom metrics** : Application Insights

**Configuration via Portal :**
```
App Service Plan → Scale out (App Service plan) → Autoscale
→ Add a rule → Metric (CPU Percentage > 70)
→ Operation (Increase count by 1)
→ Cool down (5 minutes)
```

**Best Practices Autoscale :**
- ✅ **Minimum 2 instances** pour haute disponibilité
- ✅ **Cool down period** : 5-10 minutes (éviter flapping)
- ✅ **Margins** : Scale out à 70%, scale in à 30%
- ⚠️ **Limites** : Définir max instances pour contrôler coûts

**Matrice de Décision - Scaling :**

| Symptôme | Solution | Commande |
|----------|----------|----------|
| **CPU/RAM saturé** sur instances | Scale Up | Change SKU (B1→S1) |
| **Trafic élevé**, instances saturées | Scale Out | Augmenter instances |
| **Trafic variable** (pics) | Autoscale | Règles basées métriques |
| **Besoin de deployment slots** | Scale Up | Upgrade vers S1+ |
| **Latence élevée** | Scale Out | Plus d'instances |
| **Coûts trop élevés** | Scale Down/In | Réduire tier ou instances |

**Limitations par Tier :**

| Tier | Scale Out Manuel | Autoscale | Max Instances |
|------|-----------------|-----------|---------------|
| Free/Shared | ❌ Non | ❌ Non | 1 (partagé) |
| Basic | ✅ Oui | ❌ Non | 3 |
| Standard | ✅ Oui | ✅ Oui | 10 |
| Premium | ✅ Oui | ✅ Oui | 30 |
| Isolated | ✅ Oui | ✅ Oui | 100 |

**Scénarios d'Examen :**

| Question | Réponse | Raison |
|----------|---------|--------|
| App nécessite plus de RAM | **Scale Up** | Augmenter ressources par instance |
| Trafic web élevé, pic de charge | **Scale Out** | Ajouter instances pour capacité |
| Besoin deployment slots | **Scale Up to Standard** | Slots disponibles à partir de S1 |
| Trafic variable jour/nuit | **Autoscale** | Ajustement automatique |
| Minimiser downtime pendant scaling | **Scale Out** | Pas de redémarrage |

**PowerShell - Gestion Complète :**
```powershell
# Scale Up
Set-AzAppServicePlan `
  -ResourceGroupName "myRG" `
  -Name "myAppServicePlan" `
  -Tier "Standard" `
  -WorkerSize "Medium"

# Scale Out
Set-AzAppServicePlan `
  -ResourceGroupName "myRG" `
  -Name "myAppServicePlan" `
  -NumberofWorkers 5

# Get current scale
Get-AzAppServicePlan `
  -ResourceGroupName "myRG" `
  -Name "myAppServicePlan" | 
  Select-Object Name, Sku, NumberOfWorkers
```

#### Runtime Stacks et OS
** Point clé identifié :** Un App Service = Un runtime
- Chaque App Service configuré pour **un seul runtime**
- Python OU Java OU C# - pas de mix possible
- Un Plan App Service peut héberger plusieurs App Services du même OS

#### Deployment Slots

** Workflow optimal identifié :**
1. **Deploy** → Déployer l'update sur staging slot
2. **Test** → Tester l'application sur staging
3. **Swap** → Basculer staging vers production

**Avantages du Slot Swapping :**
- **Zero downtime** : Aucune interruption
- **Warm-up automatique** : Instances préchauffées
- **Rollback instant** : Re-swap pour annuler
- **Configuration preservation** : Settings spécifiques aux slots

#### Authentication et Authorization

**⚠️ Erreur Courante QCM : Comprendre Anonymous Access**

**Anonymous Access n'est PAS une méthode d'authentification** - c'est l'**ABSENCE** d'authentification.

**Méthodes d'Authentification Disponibles :**
- ✅ **Microsoft Entra ID (Azure AD)** : Authentification Azure AD
- ✅ **Microsoft Account** : Comptes personnels Microsoft
- ✅ **Facebook** : Authentification via Facebook
- ✅ **Google** : Authentification via Google
- ✅ **Twitter** : Authentification via Twitter
- ❌ **Anonymous Access** : Pas d'authentification (accès public par défaut)

**Configuration - Désactiver l'Accès Anonyme :**

Par défaut, App Service permet l'accès anonyme (sans authentification). Pour sécuriser votre application :

```bash
# Configurer l'authentification avec Azure AD
az webapp auth update \
  --resource-group myRG \
  --name myWebApp \
  --enabled true \
  --action LoginWithAzureActiveDirectory \
  --aad-client-id {client-id}
```

**Via Azure Portal :**
```
App Service → Authentication/Authorization
→ App Service Authentication: On
→ Action when request is not authenticated: Log in with Azure Active Directory
→ Authentication Providers: Configure providers
```

**Actions Disponibles :**
- **Allow Anonymous requests (no action)** : Accès public autorisé
- **Log in with [Provider]** : Redirection vers authentification
- **Return 401 Unauthorized** : Rejeter les requêtes non authentifiées

### 3.4 Azure Container Instances (ACI)

**⚠️ Concept Clé pour AZ-104 : ACI est le moyen le plus simple et rapide d'exécuter des containers dans Azure sans gérer de serveurs**

**Définition :**
- **Azure Container Instances (ACI)** : Service serverless pour exécuter des containers Docker dans Azure
- **Pas d'infrastructure** : Aucune VM, orchestrateur, ou cluster à gérer
- **Démarrage rapide** : Containers disponibles en secondes
- **Facturation** : Pay-per-second (facturé à la seconde d'exécution)
- **Isolation** : Chaque container groupe est isolé dans son propre sandbox

**Caractéristiques Principales :**

**1. Serverless Containers**
- ✅ **Pas de gestion d'infrastructure** : Azure gère tout
- ✅ **Pas de VM à provisionner** : Containers s'exécutent directement sur l'infrastructure Azure
- ✅ **Pas d'orchestrateur requis** : Pas besoin de Kubernetes, Docker Swarm, etc.
- ✅ **Démarrage instantané** : Containers disponibles en quelques secondes

**2. Facturation à la Seconde**
- **Facturation** : Pay-per-second (facturé à la seconde d'exécution)
- **Avantage** : Coût optimal pour workloads intermittents
- **Exemple** : Container qui tourne 2h30 = facturé pour exactement 9000 secondes
- **Comparaison** : VMs = facturées à l'heure (minimum 1h même si arrêtée après 5 min)

**3. Support Multi-OS**
- **Linux containers** : Toutes les images Linux standard
- **Windows containers** : Windows Server Core, Nano Server
- **Images** : Docker Hub, Azure Container Registry (ACR), autres registries

**4. Isolation et Sécurité**
- **Isolation** : Chaque container groupe est isolé
- **Network isolation** : VNet integration possible (delegated subnet)
- **Managed Identity** : Support des identités managées Azure AD
- **Secrets** : Support Azure Key Vault pour secrets

**Architecture ACI :**

```
┌─────────────────────────────────────────┐
│      Azure Container Instances         │
├─────────────────────────────────────────┤
│  Container Group 1                     │
│  ├── Container A (nginx)                │
│  ├── Container B (app)                  │
│  └── Shared Volume (Azure Files)        │
├─────────────────────────────────────────┤
│  Container Group 2                      │
│  └── Container C (batch job)            │
└─────────────────────────────────────────┘
```

**Container Groups :**
- **Définition** : Collection de containers qui partagent ressources et réseau
- **Ressources partagées** : CPU, RAM, réseau, volumes
- **Lifecycle** : Tous les containers démarrent/arrêtent ensemble
- **Use case** : Sidecar pattern (app + logging container)

**Création d'un Container Instance :**

**Via Azure CLI :**
```bash
# Créer un container simple (Linux)
az container create \
  --resource-group myRG \
  --name mycontainer \
  --image mcr.microsoft.com/azuredocs/aci-helloworld:latest \
  --dns-name-label myapp \
  --ports 80 \
  --cpu 1 \
  --memory 1.5

# Créer avec variables d'environnement
az container create \
  --resource-group myRG \
  --name mycontainer \
  --image myregistry.azurecr.io/myapp:latest \
  --registry-login-server myregistry.azurecr.io \
  --registry-username myregistry \
  --registry-password $ACR_PASSWORD \
  --environment-variables \
    DATABASE_URL='https://mydb.azure.com' \
    API_KEY='secret123' \
  --cpu 2 \
  --memory 4

# Créer avec command override
az container create \
  --resource-group myRG \
  --name mycontainer \
  --image alpine:latest \
  --command-line "ping -c 10 8.8.8.8" \
  --restart-policy Never
```

**Via Azure Portal :**
```
Container Instances → Create → Configure:
- Container name
- Image source (Docker Hub, ACR, etc.)
- OS type (Linux/Windows)
- Size (CPU cores, Memory)
- Networking (Public IP, DNS name)
- Environment variables
- Restart policy
```

**Configuration Avancée :**

**1. Container Groups Multi-Containers :**
```bash
# Créer un container group avec plusieurs containers
az container create \
  --resource-group myRG \
  --name mycontainergroup \
  --image mcr.microsoft.com/azuredocs/aci-helloworld:latest \
  --cpu 2 \
  --memory 3 \
  --ip-address Public \
  --ports 80 \
  --container-name webapp \
  --registry-login-server myregistry.azurecr.io \
  --registry-username myregistry \
  --registry-password $ACR_PASSWORD

# Ajouter un second container au groupe (via YAML)
az container create \
  --resource-group myRG \
  --name mycontainergroup \
  --yaml container-group.yaml
```

**Fichier container-group.yaml :**
```yaml
apiVersion: 2018-10-01
location: eastus
name: mycontainergroup
properties:
  containers:
  - name: webapp
    properties:
      image: myregistry.azurecr.io/webapp:latest
      resources:
        requests:
          cpu: 1
          memoryInGb: 1.5
      ports:
      - port: 80
        protocol: TCP
  - name: sidecar
    properties:
      image: myregistry.azurecr.io/logging:latest
      resources:
        requests:
          cpu: 0.5
          memoryInGb: 0.5
  osType: Linux
  restartPolicy: Always
  ipAddress:
    type: Public
    ports:
    - protocol: tcp
      port: 80
    dnsNameLabel: myapp
```

**2. Volumes et Stockage :**

**Types de Volumes Supportés :**
- **Azure Files Share** : Partages SMB montés (persistants)
- **Git Repo** : Clone un repo Git dans le container
- **Empty Directory** : Volume temporaire (perdu à l'arrêt)
- **Secret Volume** : Secrets depuis Azure Key Vault

```bash
# Créer avec Azure Files volume
az container create \
  --resource-group myRG \
  --name mycontainer \
  --image myapp:latest \
  --azure-file-volume-share-name myshare \
  --azure-file-volume-account-name mystorageaccount \
  --azure-file-volume-account-key $STORAGE_KEY \
  --azure-file-volume-mount-path /mnt/azure \
  --cpu 1 \
  --memory 1.5

# Créer avec Git repo volume
az container create \
  --resource-group myRG \
  --name mycontainer \
  --image alpine/git:latest \
  --gitrepo-url https://github.com/Azure-Samples/aci-helloworld.git \
  --gitrepo-mount-path /mnt/repo \
  --command-line "ls /mnt/repo" \
  --restart-policy Never
```

**3. Restart Policies :**

| Policy | Comportement | Use Case |
|--------|-------------|----------|
| **Always** (Default) | Redémarre automatiquement si arrêté | Applications long-running |
| **OnFailure** | Redémarre seulement si erreur (exit code ≠ 0) | Jobs avec retry |
| **Never** | Ne redémarre jamais | Jobs batch, one-time tasks |

```bash
# Container avec restart policy Never (job batch)
az container create \
  --resource-group myRG \
  --name batchjob \
  --image myapp:latest \
  --restart-policy Never \
  --command-line "python process_data.py"
```

**4. Networking :**

**Options de Networking :**
- **Public IP** : Accès Internet public (avec DNS name label optionnel)
- **VNet Integration** : Container dans un subnet délégué (ACI subnet)
- **Private IP** : IP privée dans VNet uniquement

```bash
# Container avec IP publique et DNS name
az container create \
  --resource-group myRG \
  --name mycontainer \
  --image nginx:latest \
  --dns-name-label myapp \
  --ports 80 \
  --ip-address Public

# URL accessible : http://myapp.eastus.azurecontainer.io

# Container avec VNet integration
az container create \
  --resource-group myRG \
  --name mycontainer \
  --image myapp:latest \
  --vnet myVNet \
  --subnet aci-subnet \
  --ip-address Private
```

**⚠️ Prérequis VNet Integration :**
- Subnet délégué à `Microsoft.ContainerInstance/containerGroups`
- Subnet avec au moins 32 adresses IP (/27 minimum)
- NSG configuré pour autoriser le trafic

**5. Managed Identities :**

```bash
# Créer container avec System-Assigned Managed Identity
az container create \
  --resource-group myRG \
  --name mycontainer \
  --image myapp:latest \
  --assign-identity \
  --cpu 1 \
  --memory 1.5

# Assigner RBAC role à l'identité
az role assignment create \
  --assignee $(az container show --name mycontainer --resource-group myRG --query identity.principalId -o tsv) \
  --role "Storage Blob Data Reader" \
  --scope /subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Storage/storageAccounts/mystorageaccount
```

**Limites et Quotas :**

| Ressource | Limite | Notes |
|-----------|--------|-------|
| **CPU par container** | 1-4 cores | Dépend de la région |
| **RAM par container** | 0.5-16 GB | Dépend de la région |
| **Containers par groupe** | 60 | Maximum |
| **Volumes par groupe** | 20 | Azure Files, Git, Empty, Secret |
| **Ports par groupe** | 5 | Ports TCP/UDP exposés |
| **Container groups par région** | 50 | Par défaut (augmentable) |
| **Taille image** | 15 GB | Maximum |

**Use Cases Détaillés :**

**1. Burst Workloads et Scaling Événementiel**
```
Scénario : Site e-commerce avec pic de trafic Black Friday
Solution : ACI pour gérer le trafic supplémentaire
Avantage : Démarrage en secondes, facturation à la seconde
```

**2. CI/CD Build Agents**
```
Scénario : Agents de build Azure DevOps
Solution : ACI comme agents auto-hébergés
Avantage : Pas de VM à maintenir, scaling automatique
```

**3. Jobs Batch et Data Processing**
```
Scénario : Traitement de données quotidien
Solution : Container ACI avec restart policy Never
Avantage : Coût optimal (payé seulement pendant exécution)
```

**4. Development/Testing Environnements**
```
Scénario : Environnements temporaires pour tests
Solution : ACI avec images de dev
Avantage : Création/destruction rapide, coût minimal
```

**5. Microservices Simple (sans orchestrateur)**
```
Scénario : Application simple avec 2-3 microservices
Solution : Container groups avec plusieurs containers
Avantage : Pas besoin de Kubernetes pour cas simples
```

**6. Scheduled Tasks**
```
Scénario : Tâches planifiées (cron jobs)
Solution : ACI + Azure Logic Apps / Event Grid
Avantage : Serverless, pas de VM à maintenir
```

**Monitoring et Logs :**

```bash
# Afficher les logs d'un container
az container logs \
  --resource-group myRG \
  --name mycontainer

# Suivre les logs en temps réel (tail -f)
az container attach \
  --resource-group myRG \
  --name mycontainer

# Afficher l'état et métriques
az container show \
  --resource-group myRG \
  --name mycontainer \
  --query "{Status:containers[0].instanceView.currentState.state,CPU:containers[0].resources.requests.cpu,Memory:containers[0].resources.requests.memoryInGb}"

# Lister tous les containers
az container list \
  --resource-group myRG \
  --output table
```

**Gestion du Lifecycle :**

```bash
# Démarrer un container arrêté
az container start \
  --resource-group myRG \
  --name mycontainer

# Arrêter un container
az container stop \
  --resource-group myRG \
  --name mycontainer

# Redémarrer un container
az container restart \
  --resource-group myRG \
  --name mycontainer

# Supprimer un container
az container delete \
  --resource-group myRG \
  --name mycontainer \
  --yes
```

**Comparaison ACI vs Autres Services :**

| Critère | ACI | VMs | App Service | AKS |
|---------|-----|-----|-------------|-----|
| **Démarrage** | ⚡ Secondes | ⏱️ Minutes | ⚡ Secondes | ⏱️ Minutes |
| **Gestion infrastructure** | ✅ Aucune | ❌ Complète | ✅ Aucune | ⚠️ Partielle |
| **Orchestration** | ❌ Non | ❌ Non | ❌ Non | ✅ Oui (K8s) |
| **Scaling** | ⚠️ Manuel | ⚠️ Manuel | ✅ Auto | ✅ Auto |
| **Coût (workload intermittent)** | ✅ Optimal | ❌ Élevé | ⚠️ Moyen | ❌ Élevé |
| **Multi-containers** | ✅ Oui (groups) | ✅ Oui | ❌ Non | ✅ Oui |
| **VNet Integration** | ✅ Oui | ✅ Oui | ✅ Oui | ✅ Oui |
| **Use Case** | Jobs, burst | Custom OS | Web apps | Production K8s |

**Scénarios d'Examen AZ-104 :**

**Question Type 1 : Workload Intermittent**
```
Scénario : Application qui tourne 2h/jour, besoin de démarrage rapide
Question : Quelle solution avec coût minimal et effort admin minimal ?

Réponse : Azure Container Instances ✅

Justification :
- Facturation à la seconde (optimal pour intermittent)
- Pas d'infrastructure à gérer
- Démarrage en secondes
- Pas besoin d'orchestrateur pour cas simple
```

**Question Type 2 : CI/CD Build Agents**
```
Scénario : Besoin d'agents de build pour Azure DevOps, pas de maintenance
Question : Quelle solution serverless ?

Réponse : ACI avec auto-scaling ✅

Justification :
- Pas de VM à maintenir
- Agents créés à la demande
- Coût optimal (payé seulement pendant builds)
```

**Question Type 3 : ACI vs AKS**
```
Scénario : Application simple avec 2 containers, pas de scaling complexe
Question : ACI ou AKS ?

Réponse : ACI ✅ (si pas besoin orchestration avancée)

Justification :
- ACI = Plus simple, moins de coût
- AKS = Overkill pour cas simple
- AKS = Nécessaire si besoin auto-scaling, service mesh, etc.
```

**Best Practices ACI :**

✅ **À FAIRE :**
- Utiliser **Azure Container Registry (ACR)** pour images privées
- Configurer **restart policy** appropriée (Always/OnFailure/Never)
- Utiliser **Managed Identities** pour accès sécurisé aux ressources
- Monitorer avec **Azure Monitor** et logs
- Utiliser **VNet integration** pour workloads sensibles
- Optimiser **taille images** (images légères = démarrage plus rapide)

❌ **À ÉVITER :**
- Utiliser ACI pour applications long-running 24/7 (coût élevé vs VMs)
- Hardcoder secrets dans images (utiliser Key Vault)
- Oublier de configurer restart policy (Always par défaut peut être inapproprié)
- Utiliser ACI pour workloads nécessitant orchestration complexe (préférer AKS)

### 3.5 Infrastructure as Code (IaC) - ARM, Bicep, Terraform

#### ARM Templates (Azure Resource Manager Templates)

**⚠️ Fondamental pour l'Examen AZ-104**

**Définition :**
ARM Templates sont des fichiers JSON déclaratifs qui définissent l'infrastructure Azure de manière programmatique (Infrastructure as Code).

**Avantages :**
- ✅ **Déclaratif** : Décrivez "quoi" déployer, pas "comment"
- ✅ **Idempotent** : Même résultat à chaque exécution
- ✅ **Orchestration** : Déploie ressources dans le bon ordre
- ✅ **Validation** : Test avant déploiement
- ✅ **Modularité** : Templates réutilisables
- ✅ **Versioning** : Intégration Git/DevOps

**Structure Complète ARM Template :**

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "metadata": {
    "description": "Déploiement VM Windows avec disques et réseau"
  },
  "parameters": {
    "vmName": {
      "type": "string",
      "defaultValue": "myVM",
      "metadata": {
        "description": "Nom de la machine virtuelle"
      }
    },
    "vmSize": {
      "type": "string",
      "defaultValue": "Standard_D2s_v3",
      "allowedValues": [
        "Standard_B2s",
        "Standard_D2s_v3",
        "Standard_D4s_v3"
      ],
      "metadata": {
        "description": "Taille de la VM"
      }
    },
    "adminUsername": {
      "type": "string",
      "metadata": {
        "description": "Nom d'utilisateur administrateur"
      }
    },
    "adminPassword": {
      "type": "securestring",
      "metadata": {
        "description": "Mot de passe administrateur"
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Localisation des ressources"
      }
    }
  },
  "variables": {
    "vnetName": "[concat(parameters('vmName'), '-vnet')]",
    "subnetName": "default",
    "nicName": "[concat(parameters('vmName'), '-nic')]",
    "publicIPName": "[concat(parameters('vmName'), '-pip')]",
    "nsgName": "[concat(parameters('vmName'), '-nsg')]",
    "osDiskName": "[concat(parameters('vmName'), '-osdisk')]",
    "addressPrefix": "10.0.0.0/16",
    "subnetPrefix": "10.0.1.0/24"
  },
  "resources": [
    {
      "type": "Microsoft.Network/virtualNetworks",
      "apiVersion": "2023-04-01",
      "name": "[variables('vnetName')]",
      "location": "[parameters('location')]",
      "properties": {
        "addressSpace": {
          "addressPrefixes": [
            "[variables('addressPrefix')]"
          ]
        },
        "subnets": [
          {
            "name": "[variables('subnetName')]",
            "properties": {
              "addressPrefix": "[variables('subnetPrefix')]"
            }
          }
        ]
      }
    },
    {
      "type": "Microsoft.Network/publicIPAddresses",
      "apiVersion": "2023-04-01",
      "name": "[variables('publicIPName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "Standard"
      },
      "properties": {
        "publicIPAllocationMethod": "Static"
      }
    },
    {
      "type": "Microsoft.Network/networkSecurityGroups",
      "apiVersion": "2023-04-01",
      "name": "[variables('nsgName')]",
      "location": "[parameters('location')]",
      "properties": {
        "securityRules": [
          {
            "name": "RDP",
            "properties": {
              "priority": 100,
              "protocol": "Tcp",
              "access": "Allow",
              "direction": "Inbound",
              "sourceAddressPrefix": "*",
              "sourcePortRange": "*",
              "destinationAddressPrefix": "*",
              "destinationPortRange": "3389"
            }
          }
        ]
      }
    },
    {
      "type": "Microsoft.Network/networkInterfaces",
      "apiVersion": "2023-04-01",
      "name": "[variables('nicName')]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[resourceId('Microsoft.Network/virtualNetworks', variables('vnetName'))]",
        "[resourceId('Microsoft.Network/publicIPAddresses', variables('publicIPName'))]",
        "[resourceId('Microsoft.Network/networkSecurityGroups', variables('nsgName'))]"
      ],
      "properties": {
        "ipConfigurations": [
          {
            "name": "ipconfig1",
            "properties": {
              "subnet": {
                "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('vnetName'), variables('subnetName'))]"
              },
              "publicIPAddress": {
                "id": "[resourceId('Microsoft.Network/publicIPAddresses', variables('publicIPName'))]"
              }
            }
          }
        ],
        "networkSecurityGroup": {
          "id": "[resourceId('Microsoft.Network/networkSecurityGroups', variables('nsgName'))]"
        }
      }
    },
    {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2023-03-01",
      "name": "[parameters('vmName')]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
      ],
      "properties": {
        "hardwareProfile": {
          "vmSize": "[parameters('vmSize')]"
        },
        "osProfile": {
          "computerName": "[parameters('vmName')]",
          "adminUsername": "[parameters('adminUsername')]",
          "adminPassword": "[parameters('adminPassword')]"
        },
        "storageProfile": {
          "imageReference": {
            "publisher": "MicrosoftWindowsServer",
            "offer": "WindowsServer",
            "sku": "2022-Datacenter",
            "version": "latest"
          },
          "osDisk": {
            "name": "[variables('osDiskName')]",
            "createOption": "FromImage",
            "managedDisk": {
              "storageAccountType": "Premium_LRS"
            }
          },
          "dataDisks": [
            {
              "lun": 0,
              "name": "[concat(parameters('vmName'), '-datadisk1')]",
              "createOption": "Empty",
              "diskSizeGB": 128,
              "managedDisk": {
                "storageAccountType": "Premium_LRS"
              }
            }
          ]
        },
        "networkProfile": {
          "networkInterfaces": [
            {
              "id": "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
            }
          ]
        }
      }
    }
  ],
  "outputs": {
    "vmName": {
      "type": "string",
      "value": "[parameters('vmName')]"
    },
    "publicIP": {
      "type": "string",
      "value": "[reference(resourceId('Microsoft.Network/publicIPAddresses', variables('publicIPName'))).ipAddress]"
    },
    "resourceId": {
      "type": "string",
      "value": "[resourceId('Microsoft.Compute/virtualMachines', parameters('vmName'))]"
    }
  }
}
```

**Sections du Template ARM :**

**1. `$schema` et `contentVersion` :**
- **$schema** : Définit le schéma de validation
- **contentVersion** : Version du template (pour tracking)

**2. `parameters` - Valeurs Configurables :**
```json
"parameters": {
  "vmName": {
    "type": "string",
    "defaultValue": "myVM",
    "minLength": 1,
    "maxLength": 15,
    "metadata": {
      "description": "Nom de la VM"
    }
  },
  "adminPassword": {
    "type": "securestring"
  },
  "vmSize": {
    "type": "string",
    "allowedValues": ["Standard_B2s", "Standard_D2s_v3"]
  }
}
```

**Types de paramètres :**
- `string` : Texte
- `securestring` : Mot de passe (non affiché dans logs)
- `int` : Nombre entier
- `bool` : Boolean
- `object` : Objet JSON
- `array` : Tableau
- `secureObject` : Objet sécurisé

**3. `variables` - Valeurs Calculées :**
```json
"variables": {
  "nicName": "[concat(parameters('vmName'), '-nic')]",
  "subnetRef": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('vnetName'), variables('subnetName'))]",
  "isProduction": "[equals(parameters('environment'), 'prod')]"
}
```

**Fonctions ARM utiles :**
- `concat()` : Concaténation de chaînes
- `resourceId()` : Obtenir l'ID d'une ressource
- `reference()` : Obtenir les propriétés d'une ressource
- `parameters()` : Accéder aux paramètres
- `variables()` : Accéder aux variables
- `resourceGroup()` : Info sur le resource group
- `subscription()` : Info sur la subscription
- `uniqueString()` : Générer chaîne unique
- `equals()`, `greater()`, `less()` : Comparaisons
- `if()` : Condition ternaire

**4. `resources` - Ressources Azure :**
```json
"resources": [
  {
    "type": "Microsoft.Compute/virtualMachines",
    "apiVersion": "2023-03-01",
    "name": "[parameters('vmName')]",
    "location": "[parameters('location')]",
    "dependsOn": [
      "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
    ],
    "properties": {
      // Configuration de la ressource
    }
  }
]
```

**`dependsOn` - Ordre de Déploiement :**
- Spécifie les dépendances entre ressources
- Azure déploie les ressources dans le bon ordre
- Utilise `resourceId()` pour référencer

**5. `outputs` - Valeurs Retournées :**
```json
"outputs": {
  "publicIP": {
    "type": "string",
    "value": "[reference(resourceId('Microsoft.Network/publicIPAddresses', variables('publicIPName'))).ipAddress]"
  },
  "vmId": {
    "type": "string",
    "value": "[resourceId('Microsoft.Compute/virtualMachines', parameters('vmName'))]"
  }
}
```

**Déploiement ARM Template :**

**Via Azure CLI :**
```bash
# Déployer template avec paramètres inline
az deployment group create \
  --resource-group myRG \
  --template-file azuredeploy.json \
  --parameters vmName=myVM adminUsername=azureuser adminPassword='P@ssw0rd123!'

# Déployer avec fichier de paramètres
az deployment group create \
  --resource-group myRG \
  --template-file azuredeploy.json \
  --parameters @azuredeploy.parameters.json

# Valider avant déploiement (What-If)
az deployment group validate \
  --resource-group myRG \
  --template-file azuredeploy.json \
  --parameters @azuredeploy.parameters.json

# What-If - Prévisualiser changements
az deployment group what-if \
  --resource-group myRG \
  --template-file azuredeploy.json \
  --parameters @azuredeploy.parameters.json
```

**Via PowerShell :**
```powershell
# Déployer template
New-AzResourceGroupDeployment `
  -ResourceGroupName "myRG" `
  -TemplateFile "azuredeploy.json" `
  -TemplateParameterFile "azuredeploy.parameters.json" `
  -Verbose

# Test deployment (validation)
Test-AzResourceGroupDeployment `
  -ResourceGroupName "myRG" `
  -TemplateFile "azuredeploy.json" `
  -TemplateParameterFile "azuredeploy.parameters.json"

# What-If
New-AzResourceGroupDeployment `
  -ResourceGroupName "myRG" `
  -TemplateFile "azuredeploy.json" `
  -WhatIf
```

**Fichier de Paramètres (azuredeploy.parameters.json) :**
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "value": "myVM"
    },
    "vmSize": {
      "value": "Standard_D2s_v3"
    },
    "adminUsername": {
      "value": "azureuser"
    },
    "adminPassword": {
      "reference": {
        "keyVault": {
          "id": "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.KeyVault/vaults/myKeyVault"
        },
        "secretName": "vmAdminPassword"
      }
    }
  }
}
```

**Linked Templates - Modularité :**
```json
{
  "type": "Microsoft.Resources/deployments",
  "apiVersion": "2021-04-01",
  "name": "linkedDeployment",
  "properties": {
    "mode": "Incremental",
    "templateLink": {
      "uri": "https://mystorageaccount.blob.core.windows.net/templates/network.json",
      "contentVersion": "1.0.0.0"
    },
    "parameters": {
      "vnetName": {
        "value": "[variables('vnetName')]"
      }
    }
  }
}
```

**Deployment Modes :**

| Mode | Comportement | Use Case |
|------|--------------|----------|
| **Incremental** (Default) | Ajoute/modifie ressources, ne supprime pas | Ajout progressif de ressources |
| **Complete** | Supprime ressources non dans template | Déploiement complet contrôlé |

**Best Practices ARM Templates :**

✅ **À FAIRE :**
- Utiliser parameters files pour différents environnements
- Stocker passwords dans Azure Key Vault
- Utiliser `dependsOn` explicitement
- Valider avec `az deployment group validate`
- Tester avec `what-if` avant production
- Versionner templates dans Git
- Utiliser linked templates pour modularité
- Documenter avec `metadata.description`

❌ **À ÉVITER :**
- Hardcoder passwords dans templates
- Templates monolithiques (> 1000 lignes)
- Complete mode sans tester (risque suppression)
- Ignorer les erreurs de validation

#### Bicep - Alternative Moderne aux ARM Templates

**⚠️ Important pour l'Examen AZ-104**

**Définition :**
Bicep est un langage déclaratif pour déployer des ressources Azure. C'est une **abstraction simplifiée** d'ARM Templates JSON.

**Avantages Bicep vs ARM JSON :**
- ✅ **Syntaxe simplifiée** : Moins verbose (50% moins de code)
- ✅ **Type safety** : Validation au moment de l'écriture
- ✅ **Intellisense** : Support IDE (VS Code extension)
- ✅ **Modules** : Réutilisation facile
- ✅ **No state file** : Contrairement à Terraform
- ✅ **Transpilation** : Bicep → ARM JSON automatique

**Même VM en Bicep (comparaison) :**

```bicep
// Paramètres
@description('Nom de la VM')
@minLength(1)
@maxLength(15)
param vmName string = 'myVM'

@description('Taille de la VM')
@allowed([
  'Standard_B2s'
  'Standard_D2s_v3'
  'Standard_D4s_v3'
])
param vmSize string = 'Standard_D2s_v3'

@description('Nom d\'utilisateur administrateur')
param adminUsername string

@secure()
@description('Mot de passe administrateur')
param adminPassword string

param location string = resourceGroup().location

// Variables
var vnetName = '${vmName}-vnet'
var subnetName = 'default'
var nicName = '${vmName}-nic'
var publicIPName = '${vmName}-pip'
var nsgName = '${vmName}-nsg'
var osDiskName = '${vmName}-osdisk'
var addressPrefix = '10.0.0.0/16'
var subnetPrefix = '10.0.1.0/24'

// Ressources
resource vnet 'Microsoft.Network/virtualNetworks@2023-04-01' = {
  name: vnetName
  location: location
  properties: {
    addressSpace: {
      addressPrefixes: [
        addressPrefix
      ]
    }
    subnets: [
      {
        name: subnetName
        properties: {
          addressPrefix: subnetPrefix
        }
      }
    ]
  }
}

resource publicIP 'Microsoft.Network/publicIPAddresses@2023-04-01' = {
  name: publicIPName
  location: location
  sku: {
    name: 'Standard'
  }
  properties: {
    publicIPAllocationMethod: 'Static'
  }
}

resource nsg 'Microsoft.Network/networkSecurityGroups@2023-04-01' = {
  name: nsgName
  location: location
  properties: {
    securityRules: [
      {
        name: 'RDP'
        properties: {
          priority: 100
          protocol: 'Tcp'
          access: 'Allow'
          direction: 'Inbound'
          sourceAddressPrefix: '*'
          sourcePortRange: '*'
          destinationAddressPrefix: '*'
          destinationPortRange: '3389'
        }
      }
    ]
  }
}

resource nic 'Microsoft.Network/networkInterfaces@2023-04-01' = {
  name: nicName
  location: location
  properties: {
    ipConfigurations: [
      {
        name: 'ipconfig1'
        properties: {
          subnet: {
            id: vnet.properties.subnets[0].id
          }
          publicIPAddress: {
            id: publicIP.id
          }
        }
      }
    ]
    networkSecurityGroup: {
      id: nsg.id
    }
  }
}

resource vm 'Microsoft.Compute/virtualMachines@2023-03-01' = {
  name: vmName
  location: location
  properties: {
    hardwareProfile: {
      vmSize: vmSize
    }
    osProfile: {
      computerName: vmName
      adminUsername: adminUsername
      adminPassword: adminPassword
    }
    storageProfile: {
      imageReference: {
        publisher: 'MicrosoftWindowsServer'
        offer: 'WindowsServer'
        sku: '2022-Datacenter'
        version: 'latest'
      }
      osDisk: {
        name: osDiskName
        createOption: 'FromImage'
        managedDisk: {
          storageAccountType: 'Premium_LRS'
        }
      }
      dataDisks: [
        {
          lun: 0
          name: '${vmName}-datadisk1'
          createOption: 'Empty'
          diskSizeGB: 128
          managedDisk: {
            storageAccountType: 'Premium_LRS'
          }
        }
      ]
    }
    networkProfile: {
      networkInterfaces: [
        {
          id: nic.id
        }
      ]
    }
  }
}

// Outputs
output vmName string = vmName
output publicIP string = publicIP.properties.ipAddress
output vmId string = vm.id
```

**Différences Clés Bicep vs ARM JSON :**

| Feature | ARM JSON | Bicep |
|---------|----------|-------|
| **Syntaxe** | Verbeux, brackets | Concis, naturel |
| **Variables** | `[variables('name')]` | `varName` |
| **Parameters** | `[parameters('name')]` | `paramName` |
| **String concat** | `[concat('a', 'b')]` | `'${a}${b}'` |
| **Resource ID** | `[resourceId('...')]` | `resource.id` |
| **Properties** | `[reference('...').prop]` | `resource.properties.prop` |
| **Dependencies** | `dependsOn` explicite | Automatique (référence symbolique) |

**Bicep Modules - Réutilisation :**

**Module : storage.bicep**
```bicep
param storageAccountName string
param location string = resourceGroup().location

resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: storageAccountName
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
}

output storageAccountId string = storageAccount.id
output primaryEndpoint string = storageAccount.properties.primaryEndpoints.blob
```

**Main template : main.bicep**
```bicep
module storage 'storage.bicep' = {
  name: 'storageDeployment'
  params: {
    storageAccountName: 'mystorageaccount'
    location: 'eastus'
  }
}

output storageId string = storage.outputs.storageAccountId
```

**Déploiement Bicep :**

```bash
# Installer Bicep CLI (si pas déjà fait)
az bicep install

# Upgrader Bicep
az bicep upgrade

# Déployer Bicep template
az deployment group create \
  --resource-group myRG \
  --template-file main.bicep \
  --parameters vmName=myVM adminUsername=azureuser adminPassword='P@ssw0rd123!'

# Build Bicep vers ARM JSON (pour inspection)
az bicep build --file main.bicep

# Decompile ARM JSON vers Bicep
az bicep decompile --file azuredeploy.json

# Valider Bicep
az deployment group validate \
  --resource-group myRG \
  --template-file main.bicep

# What-If avec Bicep
az deployment group what-if \
  --resource-group myRG \
  --template-file main.bicep
```

**Bicep Loops - Itération :**
```bicep
param vmCount int = 3

resource vms 'Microsoft.Compute/virtualMachines@2023-03-01' = [for i in range(0, vmCount): {
  name: 'vm-${i}'
  location: resourceGroup().location
  properties: {
    hardwareProfile: {
      vmSize: 'Standard_B2s'
    }
    // ... autres propriétés
  }
}]
```

**Bicep Conditions :**
```bicep
param deployPublicIP bool = true

resource publicIP 'Microsoft.Network/publicIPAddresses@2023-04-01' = if (deployPublicIP) {
  name: 'myPublicIP'
  location: resourceGroup().location
  properties: {
    publicIPAllocationMethod: 'Static'
  }
}
```

#### Terraform pour Azure

**⚠️ Moins Important pour AZ-104 mais Bon à Connaître**

**Définition :**
Terraform est un outil IaC **multi-cloud** de HashiCorp. Utilise HCL (HashiCorp Configuration Language).

**Terraform vs ARM/Bicep :**

| Feature | ARM/Bicep | Terraform |
|---------|-----------|-----------|
| **Scope** | Azure uniquement | Multi-cloud (Azure, AWS, GCP) |
| **Language** | JSON / Bicep | HCL |
| **State** | Pas de state file | State file requis |
| **Provider** | Natif Azure | AzureRM provider |
| **Modules** | Bicep modules | Terraform modules |
| **What-If** | `az deployment what-if` | `terraform plan` |
| **Validation** | Intégrée Azure | Locale |

**Même VM en Terraform :**

```hcl
# Provider configuration
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {}
}

# Variables
variable "vm_name" {
  description = "Nom de la VM"
  type        = string
  default     = "myVM"
}

variable "vm_size" {
  description = "Taille de la VM"
  type        = string
  default     = "Standard_D2s_v3"
}

variable "admin_username" {
  description = "Username administrateur"
  type        = string
}

variable "admin_password" {
  description = "Password administrateur"
  type        = string
  sensitive   = true
}

variable "location" {
  description = "Localisation Azure"
  type        = string
  default     = "East US"
}

# Resource Group
resource "azurerm_resource_group" "main" {
  name     = "${var.vm_name}-rg"
  location = var.location
}

# Virtual Network
resource "azurerm_virtual_network" "main" {
  name                = "${var.vm_name}-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}

# Subnet
resource "azurerm_subnet" "main" {
  name                 = "default"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.1.0/24"]
}

# Public IP
resource "azurerm_public_ip" "main" {
  name                = "${var.vm_name}-pip"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  allocation_method   = "Static"
  sku                 = "Standard"
}

# Network Security Group
resource "azurerm_network_security_group" "main" {
  name                = "${var.vm_name}-nsg"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  security_rule {
    name                       = "RDP"
    priority                   = 100
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "3389"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }
}

# Network Interface
resource "azurerm_network_interface" "main" {
  name                = "${var.vm_name}-nic"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  ip_configuration {
    name                          = "ipconfig1"
    subnet_id                     = azurerm_subnet.main.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.main.id
  }
}

# Associate NSG with NIC
resource "azurerm_network_interface_security_group_association" "main" {
  network_interface_id      = azurerm_network_interface.main.id
  network_security_group_id = azurerm_network_security_group.main.id
}

# Virtual Machine
resource "azurerm_windows_virtual_machine" "main" {
  name                = var.vm_name
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  size                = var.vm_size
  admin_username      = var.admin_username
  admin_password      = var.admin_password

  network_interface_ids = [
    azurerm_network_interface.main.id,
  ]

  os_disk {
    name                 = "${var.vm_name}-osdisk"
    caching              = "ReadWrite"
    storage_account_type = "Premium_LRS"
  }

  source_image_reference {
    publisher = "MicrosoftWindowsServer"
    offer     = "WindowsServer"
    sku       = "2022-Datacenter"
    version   = "latest"
  }
}

# Data Disk
resource "azurerm_managed_disk" "data" {
  name                 = "${var.vm_name}-datadisk1"
  location             = azurerm_resource_group.main.location
  resource_group_name  = azurerm_resource_group.main.name
  storage_account_type = "Premium_LRS"
  create_option        = "Empty"
  disk_size_gb         = 128
}

resource "azurerm_virtual_machine_data_disk_attachment" "data" {
  managed_disk_id    = azurerm_managed_disk.data.id
  virtual_machine_id = azurerm_windows_virtual_machine.main.id
  lun                = 0
  caching            = "ReadWrite"
}

# Outputs
output "vm_name" {
  value = azurerm_windows_virtual_machine.main.name
}

output "public_ip" {
  value = azurerm_public_ip.main.ip_address
}

output "vm_id" {
  value = azurerm_windows_virtual_machine.main.id
}
```

**Commandes Terraform :**

```bash
# Initialiser Terraform (télécharge providers)
terraform init

# Valider syntaxe
terraform validate

# Formater code
terraform fmt

# Prévisualiser changements (What-If)
terraform plan

# Appliquer changements
terraform apply

# Appliquer sans confirmation
terraform apply -auto-approve

# Détruire infrastructure
terraform destroy

# Afficher outputs
terraform output

# Afficher state
terraform show

# Import ressource existante
terraform import azurerm_resource_group.main /subscriptions/{sub-id}/resourceGroups/myRG
```

**Terraform State - Gestion :**

Le state file (`terraform.tfstate`) contient l'état actuel de l'infrastructure.

**🚨 AVERTISSEMENT DE SÉCURITÉ CRITIQUE 🚨**

Le fichier `terraform.tfstate` contient des **informations sensibles en clair** :
- ❌ **Mots de passe** : Admin passwords, database credentials
- ❌ **Clés d'accès** : Storage account keys, API keys
- ❌ **Secrets** : Certificats, tokens, connection strings
- ❌ **Données privées** : Private IPs, configuration détaillée

**⚠️ INTERDICTIONS ABSOLUES :**
- ❌ **NE JAMAIS** commiter `terraform.tfstate` dans Git
- ❌ **NE JAMAIS** partager le state file sans chiffrement
- ❌ **NE JAMAIS** stocker le state file en local en production
- ❌ **NE JAMAIS** exposer le state file publiquement

**Local State (⚠️ Development SEULEMENT) :**
```hcl
# State stocké localement (terraform.tfstate)
# ⚠️ ATTENTION : Contient des secrets en clair !
# ❌ Ne JAMAIS commiter dans Git !
# ✅ Ajouter à .gitignore :
#    terraform.tfstate
#    terraform.tfstate.backup
#    *.tfstate
#    *.tfstate.*
```

**Remote State (✅ RECOMMANDÉ pour Production) :**
```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "tfstate-rg"
    storage_account_name = "tfstatestorage"
    container_name       = "tfstate"
    key                  = "terraform.tfstate"
    # ✅ Utiliser avec Azure Storage chiffré
    # ✅ Activer State Locking
    # ✅ Configurer RBAC pour accès restreint
  }
}
```

**Best Practices - Sécurité State File :**

✅ **À FAIRE :**
- Utiliser **Remote State** avec Azure Storage
- Activer **State Locking** pour éviter les conflits
- Chiffrer le backend storage (**SSE activé**)
- Configurer **RBAC** pour accès restreint au state
- Utiliser **Azure Key Vault** pour les secrets
- Activer **Versioning** sur le storage account
- Sauvegarder régulièrement le state

❌ **À ÉVITER :**
- Commiter le state file dans Git
- Partager le state file par email/chat
- Laisser le state file en local
- Utiliser des secrets en dur dans les variables

**Terraform Modules :**
```hcl
# Module pour créer VMs
module "vm" {
  source = "./modules/vm"
  
  vm_name        = "myVM"
  vm_size        = "Standard_D2s_v3"
  admin_username = "azureuser"
  admin_password = var.admin_password
  location       = "East US"
}

output "vm_public_ip" {
  value = module.vm.public_ip
}
```

#### Comparaison Complète : ARM vs Bicep vs Terraform

| Critère | ARM Templates | Bicep | Terraform |
|---------|---------------|-------|-----------|
| **Format** | JSON | Bicep (DSL) | HCL |
| **Cloud** | Azure only | Azure only | Multi-cloud |
| **Verbosité** | Élevée | Faible | Moyenne |
| **Courbe apprentissage** | Élevée | Faible | Moyenne |
| **State management** | ❌ Aucun | ❌ Aucun | ✅ State file |
| **IDE Support** | Basique | ✅ Excellent (VS Code) | ✅ Excellent |
| **Modules** | Linked templates | ✅ Natif | ✅ Natif |
| **Community** | Microsoft | Microsoft | HashiCorp + Community |
| **Validation** | Azure | Azure | Locale |
| **What-If** | ✅ Oui | ✅ Oui | ✅ Plan |
| **Maturité** | +++++ | +++ | ++++ |
| **AZ-104 Exam** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ |

**Quand Utiliser Chaque Outil :**

| Scénario | Recommandation |
|----------|----------------|
| **Azure uniquement, nouveau projet** | **Bicep** |
| **Azure uniquement, legacy** | ARM Templates |
| **Multi-cloud (Azure + AWS + GCP)** | **Terraform** |
| **Équipe déjà Terraform** | Terraform |
| **Besoin validation native Azure** | ARM/Bicep |
| **Examen AZ-104** | ARM Templates + Bicep |

#### Scénarios d'Examen - IaC

**⚠️ Questions Typiques AZ-104**

| Question | Réponse | Justification |
|----------|---------|---------------|
| **Déployer 100 VMs identiques** | ARM Template avec copy loop | Automatisation, consistance |
| **Resource Group configurable au déploiement** | Parameter dans template | Seul paramètre runtime disponible |
| **Réutiliser template pour Dev/Staging/Prod** | Parameters files séparés | Même template, paramètres différents |
| **Valider template sans déployer** | `az deployment group validate` ou `what-if` | Test avant production |
| **Syntaxe simplifiée vs ARM JSON** | Bicep | 50% moins de code |
| **Multi-cloud Azure + AWS** | Terraform | Support multi-cloud |
| **Export configuration ressources existantes** | `az group export --name myRG` | Générer template depuis existant |

**Export Template depuis Ressources Existantes :**

```bash
# Exporter toutes ressources d'un Resource Group
az group export \
  --name myRG \
  --output json > exported-template.json

# Exporter template d'un déploiement
az deployment group export \
  --name myDeployment \
  --resource-group myRG \
  --output json > deployment-template.json
```

```powershell
# Export via PowerShell
Export-AzResourceGroup `
  -ResourceGroupName "myRG" `
  -Path "exported-template.json" `
  -Force
```

### 3.6 Azure Kubernetes Service (AKS) - Basics

**⚠️ Concept Clé pour AZ-104 : AKS est le service managé Kubernetes d'Azure pour orchestrer des containers en production**

**Définition :**
- **Azure Kubernetes Service (AKS)** : Service managé Kubernetes pour déployer, gérer et scaler des applications containerisées
- **Kubernetes (K8s)** : Orchestrateur open-source pour automatiser le déploiement, scaling et gestion de containers
- **Managed Service** : Azure gère les masters (control plane), vous gérez les nodes (worker nodes)
- **Use Case** : Applications production nécessitant orchestration, auto-scaling, service discovery, load balancing

**Architecture AKS :**

```
┌─────────────────────────────────────────────────────┐
│         Azure Kubernetes Service (AKS)              │
├─────────────────────────────────────────────────────┤
│  Control Plane (Managed by Azure)                   │
│  ├── API Server                                     │
│  ├── etcd (state store)                             │
│  ├── Scheduler                                      │
│  └── Controller Manager                             │
├─────────────────────────────────────────────────────┤
│  Node Pool (Worker Nodes - Managed by You)          │
│  ├── Node 1 (VM)                                    │
│  │   ├── kubelet                                    │
│  │   ├── kube-proxy                                  │
│  │   └── Pods (containers)                          │
│  ├── Node 2 (VM)                                    │
│  │   └── Pods (containers)                          │
│  └── Node 3 (VM)                                    │
│      └── Pods (containers)                          │
└─────────────────────────────────────────────────────┘
```

**Composants Clés Kubernetes :**

**1. Control Plane (Managed by Azure)**
- **API Server** : Point d'entrée pour toutes les requêtes (kubectl, dashboard)
- **etcd** : Base de données distribuée pour état du cluster
- **Scheduler** : Détermine sur quel node placer les pods
- **Controller Manager** : Gère les controllers (replicas, deployments, etc.)
- **Coût** : Gratuit (Azure gère et facture uniquement les nodes)

**2. Nodes (Worker Nodes)**
- **kubelet** : Agent sur chaque node qui communique avec l'API server
- **kube-proxy** : Gère le networking et load balancing
- **Pods** : Plus petite unité déployable (1 ou plusieurs containers)
- **Coût** : Vous payez pour les VMs des nodes

**3. Concepts Kubernetes Essentiels :**

**Pods :**
- **Définition** : Plus petite unité déployable dans Kubernetes
- **Contenu** : 1 ou plusieurs containers (généralement 1)
- **Lifecycle** : Éphémère (peuvent être recréés)
- **Ressources partagées** : Pod partage réseau, storage, IP

**Deployments :**
- **Définition** : Gère le cycle de vie des pods (création, mise à jour, rollback)
- **Replicas** : Nombre de copies de pods à maintenir
- **Rolling Updates** : Mise à jour progressive sans downtime
- **Rollback** : Retour à version précédente en cas de problème

**Services :**
- **Définition** : Abstraction réseau pour exposer pods
- **Types** : ClusterIP (interne), LoadBalancer (externe), NodePort
- **Load Balancing** : Distribution du trafic entre pods

**Namespaces :**
- **Définition** : Isolation logique des ressources
- **Use Case** : Séparer dev/staging/prod, équipes
- **Default** : `default`, `kube-system`, `kube-public`

**Création d'un Cluster AKS :**

**Via Azure CLI :**
```bash
# Créer un cluster AKS basique
az aks create \
  --resource-group myRG \
  --name myAKSCluster \
  --node-count 3 \
  --node-vm-size Standard_D2s_v3 \
  --generate-ssh-keys \
  --location eastus

# Créer avec node pool spécifique
az aks create \
  --resource-group myRG \
  --name myAKSCluster \
  --node-count 3 \
  --node-vm-size Standard_D2s_v3 \
  --enable-managed-identity \
  --network-plugin azure \
  --network-policy azure \
  --location eastus

# Créer avec Azure AD integration (RBAC)
az aks create \
  --resource-group myRG \
  --name myAKSCluster \
  --node-count 3 \
  --enable-aad \
  --aad-admin-group-object-ids <group-id> \
  --location eastus
```

**Via Azure Portal :**
```
Kubernetes services → Create → Configure:
- Cluster name
- Resource group
- Region
- Kubernetes version
- Node pool (size, count)
- Authentication (Service Principal or Managed Identity)
- Networking (CNI or kubenet)
```

**Configuration kubectl :**

```bash
# Installer kubectl (si pas déjà fait)
az aks install-cli

# Se connecter au cluster AKS
az aks get-credentials \
  --resource-group myRG \
  --name myAKSCluster

# Vérifier la connexion
kubectl get nodes

# Afficher les pods
kubectl get pods --all-namespaces

# Afficher les services
kubectl get services
```

**Déploiement d'une Application :**

**1. Créer un Deployment :**

**Fichier deployment.yaml :**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 256Mi
```

**Déployer :**
```bash
# Créer le deployment
kubectl apply -f deployment.yaml

# Vérifier le deployment
kubectl get deployments
kubectl get pods

# Afficher les détails
kubectl describe deployment nginx-deployment
```

**2. Exposer avec un Service :**

**Fichier service.yaml :**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

**Déployer :**
```bash
# Créer le service
kubectl apply -f service.yaml

# Obtenir l'IP externe
kubectl get service nginx-service

# Accéder à l'application via l'IP externe
```

**Types de Services :**

| Type | Description | Use Case |
|------|-------------|----------|
| **ClusterIP** (Default) | IP interne dans le cluster | Communication interne |
| **LoadBalancer** | IP publique Azure Load Balancer | Exposer application Internet |
| **NodePort** | Port exposé sur chaque node | Accès direct via node IP |
| **ExternalName** | CNAME vers service externe | Intégration services externes |

**Node Pools :**

**Définition :**
- **Node Pool** : Groupe de nodes avec même configuration (VM size, OS, etc.)
- **Multiple Pools** : Possibilité d'avoir plusieurs node pools (ex: CPU-intensive, GPU, Windows)

**Gestion des Node Pools :**
```bash
# Lister les node pools
az aks nodepool list \
  --resource-group myRG \
  --cluster-name myAKSCluster

# Ajouter un nouveau node pool
az aks nodepool add \
  --resource-group myRG \
  --cluster-name myAKSCluster \
  --name gpunodepool \
  --node-count 2 \
  --node-vm-size Standard_NC6s_v3 \
  --enable-cluster-autoscaler \
  --min-count 1 \
  --max-count 5

# Mettre à jour un node pool (scaling)
az aks nodepool scale \
  --resource-group myRG \
  --cluster-name myAKSCluster \
  --name nodepool1 \
  --node-count 5

# Supprimer un node pool
az aks nodepool delete \
  --resource-group myRG \
  --cluster-name myAKSCluster \
  --name gpunodepool
```

**Auto-Scaling :**

**Cluster Autoscaler :**
- **Fonction** : Ajuste automatiquement le nombre de nodes selon la demande
- **Trigger** : Pods en attente (pending) = besoin de plus de nodes
- **Scale Down** : Nodes sous-utilisés = suppression automatique

```bash
# Activer Cluster Autoscaler
az aks update \
  --resource-group myRG \
  --name myAKSCluster \
  --enable-cluster-autoscaler \
  --min-count 1 \
  --max-count 10

# Configurer sur un node pool spécifique
az aks nodepool update \
  --resource-group myRG \
  --cluster-name myAKSCluster \
  --name nodepool1 \
  --enable-cluster-autoscaler \
  --min-count 2 \
  --max-count 10
```

**Horizontal Pod Autoscaler (HPA) :**
- **Fonction** : Ajuste le nombre de replicas de pods selon métriques (CPU, mémoire)
- **Scope** : Au niveau deployment, pas au niveau cluster

```bash
# Créer un HPA
kubectl autoscale deployment nginx-deployment \
  --cpu-percent=70 \
  --min=2 \
  --max=10

# Vérifier le HPA
kubectl get hpa
```

**Networking AKS :**

**Deux Modes de Networking :**

**1. kubenet (Basic)**
- **Réseau** : Azure gère les routes
- **Limitation** : 400 nodes maximum par cluster
- **Use Case** : Clusters simples, développement

**2. Azure CNI (Advanced)**
- **Réseau** : Pods obtiennent des IPs du subnet VNet
- **Avantage** : Intégration native avec VNet Azure
- **Limitation** : Besoin de planifier l'espace IP
- **Use Case** : Production, intégration VNet

```bash
# Créer cluster avec Azure CNI
az aks create \
  --resource-group myRG \
  --name myAKSCluster \
  --network-plugin azure \
  --vnet-subnet-id /subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Network/virtualNetworks/myVNet/subnets/aks-subnet \
  --node-count 3
```

**Sécurité AKS :**

**1. Authentication et Authorization :**

**Azure AD Integration (RBAC) :**
```bash
# Créer cluster avec Azure AD
az aks create \
  --resource-group myRG \
  --name myAKSCluster \
  --enable-aad \
  --aad-admin-group-object-ids <group-id> \
  --node-count 3
```

**RBAC Kubernetes :**
- **Roles** : Permissions dans un namespace
- **ClusterRoles** : Permissions cluster-wide
- **RoleBindings** : Associe roles à utilisateurs/groups

**2. Secrets Management :**

**Kubernetes Secrets :**
```bash
# Créer un secret
kubectl create secret generic mysecret \
  --from-literal=username=admin \
  --from-literal=password=secret123

# Utiliser dans un pod
# (référencé dans deployment.yaml)
```

**Azure Key Vault Integration :**
- **Azure Key Vault Provider for Secrets Store CSI Driver**
- **Use Case** : Secrets depuis Key Vault dans pods

**3. Pod Security Policies / Pod Security Standards :**
- **Restrictions** : Limiter capabilities des pods
- **Use Case** : Sécurité renforcée, conformité

**Monitoring et Logging :**

**Azure Monitor pour Containers :**
```bash
# Activer Azure Monitor
az aks enable-addons \
  --resource-group myRG \
  --name myAKSCluster \
  --addons monitoring
```

**Logs :**
- **Container Insights** : Métriques et logs des pods
- **kube-audit** : Logs d'audit du cluster
- **kube-apiserver** : Logs de l'API server

**Commandes Utiles :**

```bash
# Afficher les nodes
kubectl get nodes

# Afficher les pods
kubectl get pods
kubectl get pods --all-namespaces

# Afficher les deployments
kubectl get deployments

# Afficher les services
kubectl get services

# Afficher les logs d'un pod
kubectl logs <pod-name>

# Exécuter une commande dans un pod
kubectl exec -it <pod-name> -- /bin/bash

# Décrire une ressource
kubectl describe pod <pod-name>

# Supprimer une ressource
kubectl delete deployment nginx-deployment
kubectl delete service nginx-service

# Mettre à jour un deployment
kubectl set image deployment/nginx-deployment nginx=nginx:1.22

# Rollback un deployment
kubectl rollout undo deployment/nginx-deployment
```

**Comparaison AKS vs ACI vs VMs :**

| Critère | AKS | ACI | VMs |
|---------|-----|-----|-----|
| **Orchestration** | ✅ Kubernetes complet | ❌ Non | ❌ Non |
| **Auto-scaling** | ✅ Pods + Nodes | ⚠️ Manuel | ⚠️ Manuel |
| **Service Discovery** | ✅ Natif | ❌ Non | ❌ Non |
| **Load Balancing** | ✅ Natif | ⚠️ Manuel | ⚠️ Load Balancer séparé |
| **Complexité** | ❌ Élevée | ✅ Faible | ⚠️ Moyenne |
| **Coût** | ⚠️ Nodes + Control Plane (gratuit) | ✅ Pay-per-second | ⚠️ Par heure |
| **Use Case** | Production, microservices | Jobs, burst, simple | Custom OS, legacy |
| **Démarrage** | ⏱️ Minutes (cluster) | ⚡ Secondes | ⏱️ Minutes |

**Scénarios d'Examen AZ-104 :**

**Question Type 1 : Quand Utiliser AKS ?**
```
Scénario : Application microservices avec 10+ services, besoin auto-scaling
Question : Quelle solution Azure ?

Réponse : Azure Kubernetes Service (AKS) ✅

Justification :
- Orchestration Kubernetes nécessaire
- Auto-scaling pods et nodes
- Service discovery natif
- Load balancing intégré
```

**Question Type 2 : AKS vs ACI**
```
Scénario : Application simple avec 2 containers, pas de scaling complexe
Question : AKS ou ACI ?

Réponse : ACI ✅ (si pas besoin orchestration)

Justification :
- ACI = Plus simple, moins de coût
- AKS = Overkill pour cas simple
- AKS = Nécessaire si besoin orchestration avancée
```

**Question Type 3 : Control Plane AKS**
```
Question : Qui gère le control plane AKS ?

Réponse : Azure ✅

Justification :
- Control plane = Managed by Azure (gratuit)
- Vous gérez uniquement les worker nodes
- Azure gère : API Server, etcd, Scheduler, Controller Manager
```

**Limites et Quotas AKS :**

| Ressource | Limite | Notes |
|-----------|--------|-------|
| **Clusters par subscription** | 50 | Par défaut |
| **Nodes par cluster (kubenet)** | 400 | Maximum |
| **Nodes par cluster (Azure CNI)** | 1000 | Maximum |
| **Pods par node** | 30-250 | Dépend de la taille VM |
| **Node pools par cluster** | 100 | Maximum |
| **Services par cluster** | 5000 | Maximum |

**Best Practices AKS :**

✅ **À FAIRE :**
- Utiliser **Managed Identity** au lieu de Service Principal
- Activer **Azure AD integration** pour RBAC
- Utiliser **Azure CNI** pour production (intégration VNet)
- Configurer **Cluster Autoscaler** pour optimiser coûts
- Activer **Azure Monitor** pour monitoring
- Utiliser **Azure Container Registry (ACR)** pour images
- Implémenter **Resource Quotas** et **Limit Ranges**
- Utiliser **Namespaces** pour isolation
- Configurer **Network Policies** pour sécurité réseau
- Utiliser **Secrets** ou **Key Vault** pour credentials

❌ **À ÉVITER :**
- Utiliser AKS pour applications simples (préférer ACI ou App Service)
- Oublier de configurer auto-scaling (coûts élevés)
- Exposer services sensibles publiquement sans authentification
- Hardcoder secrets dans images ou YAML
- Ignorer les mises à jour de sécurité Kubernetes
- Utiliser kubenet pour clusters > 50 nodes (préférer Azure CNI)

**⚠️ Points Clés pour l'Examen :**
- ✅ **Control Plane = Managed by Azure** (gratuit)
- ✅ **Worker Nodes = Votre responsabilité** (coût)
- ✅ **AKS = Orchestration complète** (vs ACI = simple)
- ✅ **Auto-scaling** : Cluster Autoscaler (nodes) + HPA (pods)
- ✅ **Networking** : kubenet (simple) vs Azure CNI (avancé)
- ✅ **Azure AD Integration** : RBAC pour sécurité
- ✅ **Use Case** : Production, microservices, orchestration complexe

---
