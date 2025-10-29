Voici le `README.md` mis à jour avec le lien vers votre dépôt GitHub.

-----

# Nova Sentinel - Phase 3: Infrastructure as Code

Ce projet contient le code d'infrastructure (Infrastructure as Code) pour le déploiement et la configuration de la plateforme d'audit "Nova Sentinel".

L'objectif est d'industrialiser l'environnement pour le rendre **reproductible, sécurisé et automatisé**. Ce dépôt gère à la fois la création de l'infrastructure (Terraform) et sa configuration (Ansible).

## 🚀 Technologies utilisées

Ce projet s'articule autour des outils du "fil rouge" Nova Sentinel :

  * **Terraform** : Pour la création (provisioning) des serveurs sur VMware vSphere.
  * **Ansible** : Pour la configuration (configuration management) des serveurs.
  * **HashiCorp Vault** : Pour la gestion centralisée et sécurisée des secrets (mots de passe, clés API).
  * **Python Sentinel** : L'agent d'audit personnalisé qui s'exécute sur les nœuds applicatifs.
  * **Git** : Pour le versioning et la traçabilité de l'infrastructure et de la configuration.

## 🏗️ Architecture Cible

L'infrastructure est pilotée depuis un **Bastion (Control Node)**. C'est la seule machine gérée manuellement, et elle héberge tous les outils de déploiement (Terraform, Ansible, Git, Vault CLI).

Le Bastion déploie automatiquement l'architecture suivante sur VMware :

```text
                               +-----------------------------+
                               |   BASTION NODE (Control Node) |
                               | - Terraform, Ansible, Git   |
                               | - Code Nova Sentinel        |
                               +--------------+--------------+
                                              |
                     +------------------------+------------------------+
                     |                        |                        |
                     v                        v                        v
+------------------+----+       +------------------+----+       +------------------+----+
|    App Node 1        |       |    App Node 2        |       |  Monitoring Node     |
| - Sentinel Agent     | | - Sentinel Agent     | | - Vault + Logs     |
+----------------------+       +----------------------+       +----------------------+
```

  * **App Nodes** : Exécutent l'agent d'audit Python Sentinel.
  * **Monitoring Node** : Centralise les logs d'audit et héberge le service Vault pour la gestion des secrets.

## 📂 Structure du Projet

```
nova-sentinel-infra/
├── terraform/                # Code d'infrastructure Terraform
│   ├── main.tf             # Ressources principales (VMs)
│   ├── variables.tf        # Variables d'entrée
│   ├── outputs.tf          # IPs de sortie pour Ansible
│   └── terraform.tfvars    # (Fichier local, ignoré par Git) Valeurs des variables
├── ansible/                  # Code de configuration Ansible
│   ├── playbooks/          # Playbooks de déploiement
│   └── inventory/          # Inventaire (généré par Terraform)
├── vault/                    # Scripts et politiques Vault
│   ├── init_vault.sh       # Script d'initialisation des secrets
│   └── policies/           # Politiques d'accès Vault
├── python_agent/             # Code source de l'agent
│   └── sentinel_agent.py   # Agent d'audit Python
├── .gitignore                # Exclut les fichiers sensibles (.tfstate, .tfvars, vault-token)
└── infra.md                  # Documentation de l'infrastructure
```

## 🛠️ Prérequis

Avant de commencer, assurez-vous que votre machine **Bastion (Control Node)** dispose de :

1.  `git`
2.  `terraform`
3.  `ansible`
4.  `vault` (CLI)
5.  Accès réseau et identifiants pour votre hyperviseur VMware vSphere.

## 🚀 Déploiement

Le déploiement complet (création infrastructure + configuration) se fait via une seule commande.

1.  **Cloner le dépôt sur le Bastion :**

    ```bash
    git clone https://github.com/7mnm/nova-sentinel-infra.git
    cd nova-sentinel-infra
    ```

2.  **Configurer les variables Terraform :**
    Créez un fichier `terraform.tfvars` à partir de l'exemple (à créer si non existant) et renseignez vos identifiants VMware vSphere.

    ```bash
    # (Créez un terraform.tfvars.example si vous n'en avez pas)
    cp terraform/terraform.tfvars.example terraform/terraform.tfvars
    nano terraform/terraform.tfvars
    ```

    *Ce fichier est ignoré par `.gitignore` pour ne pas exposer de secrets*.

3.  **Lancer le déploiement complet :**
    Naviguez dans le dossier Terraform et appliquez la configuration.

    ```bash
    cd terraform/

    # 1. Initialiser Terraform (télécharge le provider vSphere)
    terraform init

    # 2. (Optionnel) Valider le plan d'exécution
    terraform plan

    # 3. Appliquer la création et la configuration
    # Terraform va créer les VMs puis lancer Ansible automatiquement
    terraform apply
    ```

## ⚙️ Workflow d'automatisation

L'exécution de `terraform apply` déclenche le pipeline suivant :

1.  **Terraform** contacte VMware vSphere et provisionne les 3 machines virtuelles (2 App Nodes, 1 Monitoring Node).
2.  Une fois les machines créées, **Terraform** récupère leurs adresses IP.
3.  Terraform génère dynamiquement un fichier d'inventaire pour **Ansible** avec ces nouvelles IPs.
4.  Terraform déclenche automatiquement le playbook principal d'Ansible (`provisioner "local-exec"`).
5.  **Ansible** lit l'inventaire, se connecte aux machines et exécute les rôles :
      * Il installe et configure **Vault** sur le `Monitoring Node`.
      * Il récupère dynamiquement les secrets (ex: clés d'agent) depuis Vault.
      * Il déploie et configure l'agent **Python Sentinel** sur les `App Nodes`.
6.  L'infrastructure est prête et l'audit est actif.

## 🔒 Gestion des Secrets

La sécurité des secrets est gérée par **HashiCorp Vault**.

  * **Aucun mot de passe** n'est stocké en clair dans le dépôt Git (ni dans les playbooks, ni dans les variables Terraform).
  * Vault est déployé par Ansible sur le `Monitoring Node`.
  * Les secrets (clés SSH, tokens d'agent, etc.) sont définis dans Vault.
  * Les playbooks Ansible utilisent des *lookups* pour récupérer ces secrets dynamiquement au moment de l'exécution.

## 🧹 Nettoyage

Pour détruire l'ensemble de l'infrastructure créée par Terraform, exécutez la commande suivante depuis le dossier `terraform/` :

```bash
terraform destroy
```
