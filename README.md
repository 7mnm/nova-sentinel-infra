Voici une proposition de fichier `README.md` complet pour votre projet, basé sur les informations du document TP que vous avez fourni.

-----

# Nova Sentinel - Phase 3: Infrastructure as Code

[cite\_start]Ce projet contient le code d'infrastructure (Infrastructure as Code) pour le déploiement et la configuration de la plateforme d'audit "Nova Sentinel"[cite: 1, 3].

[cite\_start]L'objectif est d'industrialiser l'environnement pour le rendre **reproductible, sécurisé et automatisé**[cite: 11]. [cite\_start]Ce dépôt gère à la fois la création de l'infrastructure (Terraform) et sa configuration (Ansible)[cite: 3].

## 🚀 Technologies utilisées

[cite\_start]Ce projet s'articule autour des outils du "fil rouge" Nova Sentinel[cite: 4]:

  * [cite\_start]**Terraform** : Pour la création (provisioning) des serveurs sur VMware vSphere[cite: 5, 15].
  * [cite\_start]**Ansible** : Pour la configuration (configuration management) des serveurs[cite: 6].
  * [cite\_start]**HashiCorp Vault** : Pour la gestion centralisée et sécurisée des secrets (mots de passe, clés API)[cite: 8, 115].
  * [cite\_start]**Python Sentinel** : L'agent d'audit personnalisé qui s'exécute sur les nœuds applicatifs[cite: 7].
  * [cite\_start]**Git** : Pour le versioning et la traçabilité de l'infrastructure et de la configuration[cite: 3].

## 🏗️ Architecture Cible

[cite\_start]L'infrastructure est pilotée depuis un **Bastion (Control Node)**[cite: 12]. [cite\_start]C'est la seule machine gérée manuellement, et elle héberge tous les outils de déploiement (Terraform, Ansible, Git, Vault CLI)[cite: 13, 14].

[cite\_start]Le Bastion déploie automatiquement l'architecture suivante sur VMware[cite: 15]:

```text
                               +-----------------------------+
                               |   BASTION NODE (Control Node) |
                               | - [cite_start]Terraform, Ansible, Git   | [cite: 18]
                               | - Code Nova Sentinel        |
                               +--------------+--------------+
                                              |
                     +------------------------+------------------------+
                     |                        |                        |
                     v                        v                        v
+------------------+----+       +------------------+----+       +------------------+----+
|    App Node 1        |       |    App Node 2        |       |  Monitoring Node     |
| - [cite_start]Sentinel Agent     | [cite: 17] | - [cite_start]Sentinel Agent     | [cite: 20] | - [cite_start]Vault + Logs     | [cite: 21]
+----------------------+       +----------------------+       +----------------------+
```

  * [cite\_start]**App Nodes** : Exécutent l'agent d'audit Python Sentinel[cite: 63].
  * [cite\_start]**Monitoring Node** : Centralise les logs d'audit et héberge le service Vault pour la gestion des secrets[cite: 64].

## 📂 Structure du Projet

```
nova-sentinel-infra/
[cite_start]├── terraform/                # Code d'infrastructure Terraform [cite: 23, 24]
[cite_start]│   ├── main.tf             # Ressources principales (VMs) [cite: 25]
[cite_start]│   ├── variables.tf        # Variables d'entrée [cite: 26]
[cite_start]│   ├── outputs.tf          # IPs de sortie pour Ansible [cite: 27]
[cite_start]│   └── terraform.tfvars    # (Fichier local, ignoré par Git) Valeurs des variables [cite: 28, 142]
[cite_start]├── ansible/                  # Code de configuration Ansible [cite: 29]
[cite_start]│   ├── playbooks/          # Playbooks de déploiement [cite: 31]
[cite_start]│   └── inventory/          # Inventaire (généré par Terraform) [cite: 32, 105]
[cite_start]├── vault/                    # Scripts et politiques Vault [cite: 33, 34]
[cite_start]│   ├── init_vault.sh       # Script d'initialisation des secrets [cite: 35]
[cite_start]│   └── policies/           # Politiques d'accès Vault [cite: 36]
[cite_start]├── python_agent/             # Code source de l'agent [cite: 37]
[cite_start]│   └── sentinel_agent.py   # Agent d'audit Python [cite: 39]
[cite_start]├── .gitignore                # Exclut les fichiers sensibles (.tfstate, .tfvars, vault-token) [cite: 142]
[cite_start]└── infra.md                  # Documentation de l'infrastructure [cite: 40]
```

## 🛠️ Prérequis

Avant de commencer, assurez-vous que votre machine **Bastion (Control Node)** dispose de :

1.  [cite\_start]`git` [cite: 14]
2.  [cite\_start]`terraform` [cite: 14]
3.  [cite\_start]`ansible` [cite: 14]
4.  [cite\_start]`vault` (CLI) [cite: 14]
5.  [cite\_start]Accès réseau et identifiants pour votre hyperviseur VMware vSphere[cite: 50].

## 🚀 Déploiement

[cite\_start]Le déploiement complet (création infrastructure + configuration) se fait via une seule commande[cite: 108].

1.  **Cloner le dépôt sur le Bastion :**

    ```bash
    git clone <votre_url_repo> nova-sentinel-infra
    cd nova-sentinel-infra
    ```

2.  **Configurer les variables Terraform :**
    Créez un fichier `terraform.tfvars` à partir de l'exemple (à créer si non existant) et renseignez vos identifiants VMware vSphere.

    ```bash
    # (Créez un terraform.tfvars.example si vous n'en avez pas)
    cp terraform/terraform.tfvars.example terraform/terraform.tfvars
    nano terraform/terraform.tfvars
    ```

    [cite\_start]*Ce fichier est ignoré par `.gitignore` pour ne pas exposer de secrets*[cite: 142].

3.  **Lancer le déploiement complet :**
    Naviguez dans le dossier Terraform et appliquez la configuration.

    ```bash
    cd terraform/

    # 1. Initialiser Terraform (télécharge le provider vSphere)
    terraform init

    # 2. (Optionnel) Valider le plan d'exécution
    terraform plan

    # 3. Appliquer la création et la configuration
    # [cite_start]Terraform va créer les VMs puis lancer Ansible automatiquement [cite: 106]
    terraform apply
    ```

## ⚙️ Workflow d'automatisation

[cite\_start]L'exécution de `terraform apply` déclenche le pipeline suivant[cite: 157]:

1.  [cite\_start]**Terraform** contacte VMware vSphere et provisionne les 3 machines virtuelles (2 App Nodes, 1 Monitoring Node)[cite: 93].
2.  [cite\_start]Une fois les machines créées, **Terraform** récupère leurs adresses IP[cite: 65].
3.  [cite\_start]Terraform génère dynamiquement un fichier d'inventaire pour **Ansible** avec ces nouvelles IPs[cite: 105].
4.  [cite\_start]Terraform déclenche automatiquement le playbook principal d'Ansible (`provisioner "local-exec"`)[cite: 106].
5.  **Ansible** lit l'inventaire, se connecte aux machines et exécute les rôles :
      * [cite\_start]Il installe et configure **Vault** sur le `Monitoring Node`[cite: 117].
      * [cite\_start]Il récupère dynamiquement les secrets (ex: clés d'agent) depuis Vault[cite: 120].
      * [cite\_start]Il déploie et configure l'agent **Python Sentinel** sur les `App Nodes`[cite: 128].
6.  L'infrastructure est prête et l'audit est actif.

## 🔒 Gestion des Secrets

[cite\_start]La sécurité des secrets est gérée par **HashiCorp Vault**[cite: 115].

  * [cite\_start]**Aucun mot de passe** n'est stocké en clair dans le dépôt Git (ni dans les playbooks, ni dans les variables Terraform)[cite: 121].
  * [cite\_start]Vault est déployé par Ansible sur le `Monitoring Node`[cite: 117].
  * [cite\_start]Les secrets (clés SSH, tokens d'agent, etc.) sont définis dans Vault[cite: 118].
  * [cite\_start]Les playbooks Ansible utilisent des *lookups* pour récupérer ces secrets dynamiquement au moment de l'exécution[cite: 120].

## 🧹 Nettoyage

Pour détruire l'ensemble de l'infrastructure créée par Terraform, exécutez la commande suivante depuis le dossier `terraform/` :

```bash
terraform destroy
```
