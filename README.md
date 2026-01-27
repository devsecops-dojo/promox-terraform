# Déploiement VMs Proxmox avec Terraform

Infrastructure as Code pour déployer des VMs sur Proxmox VE pour un Kubernetes homelab.

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Proxmox VE                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  🎯 CLUSTER RANCHER (cluster-rancher)                       │
│  ├─ rancher-1      → 192.168.1.110 → 2C/8 GB/25 G          │
│  ├─ rancher-2      → 192.168.1.111 → 2C/8 GB/25 G          │
│  └─ rancher-3      → 192.168.1.112 → 2C/8 GB/25 G          │
│                                                             │
│  🔧 CLUSTER PAYLOAD (cluster-payload)                       │
│  ├─ payload-master-1 → 192.168.1.113 → 2C/4 GB/25 G        │
│  ├─ payload-master-2 → 192.168.1.114 → 2C/4 GB/25 G        │
│  ├─ payload-master-3 → 192.168.1.115 → 2C/4 GB/25 G        │
│  ├─ payload-worker-1 → 192.168.1.116 → 3C/8 GB/25 G        │
│  ├─ payload-worker-2 → 192.168.1.117 → 3C/8 GB/25 G        │
│  └─ payload-worker-3 → 192.168.1.118 → 3C/8 GB/25 G        │
│                                                             │
│  📦 SERVICES                                                │
│  └─ cicd             → 192.168.1.119 → 2C/8 GB/25 G        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Prérequis

- Terraform >= 1.12.2
- Accès à un serveur Proxmox VE (v7.0+)
- Template Rocky Linux 9 (cloud-init)
- SSH public key configurée

## Démarrage rapide

### 1. Créer le template Rocky Linux 9

Sur le serveur Proxmox :

```bash
scp create-rocky9-template.sh root@<PROXMOX_IP>:/tmp/
ssh root@<PROXMOX_IP>
chmod +x /tmp/create-rocky9-template.sh
/tmp/create-rocky9-template.sh
```

### 2. Configurer les variables

Créez/modifiez `terraform.tfvars` :

```hcl
proxmox_password = "votre-mot-de-passe"
ssh_public_key   = "ssh-rsa AAAAB3..."
proxmox_host     = "192.168.1.100"
proxmox_node     = "pve"
```

### 3. Déployer

```bash
terraform init
terraform plan
terraform apply
```

## Fichiers

| Fichier | Description |
|---------|-------------|10 VMs (Rancher + Payload + CI/CD) |
| `variables.tf` | Variables Terraform |
| `terraform.tfvars` | Valeurs des variables (ne pas committer) |
| `providers.tf` | Configuration du provider Proxmox bpg/proxmox v0.50.0 |
| `outputs.tf` | Outputs : IDs, noms, IPs des VMs |
| `create-rocky9-template.sh` | Script de création du template Rocky 9|
| `template.tf` | Configuration du template |

## Ressources déployées

**Total : 10 VMs**
- 3 VMs Rancher (Control Plane Kubernetes) : 2C / 8 GB RAM / 25 GB disque
- 3 VMs Payload Masters : 2C / 4 GB RAM / 25 GB disque
- 3 VMs Payload Workers : 3C / 8 GB RAM / 25 GB disque
- 1 VM CI/CD : 2C / 8 GB RAM / 25 GB disque

## Commandes utiles

```bash
# Voir l'état
terraform output

# Modifier et appliquer
terraform plan
terraform apply

# Supprimer l'infrastructure
terraform destroy
```

## Dépannage

### Template non trouvé
```bash
# Exécuter sur Proxmox
/tmp/create-rocky9-template.sh
```

### VMs déjà existantes
```bash
# Avec Terraform
terraform destroy

# Ou manuellement sur Proxmox
for i in {110..119}; do qm destroy $i; done
```

### Connexion SSH refusée
1. Vérifier que la VM est démarrée : `qm status <ID>`
2. Vérifier cloud-init via la console Proxmox

## Sécurité

- `terraform.tfvars` contient des secrets - ne pas committer
- `terraform.tfstate` contient l'état - utiliser un backend distant en production
- Le dossier `.terraform/` est généré automatiquement

## Provider

Utilise **bpg/proxmox** : https://registry.terraform.io/providers/bpg/proxmox/latest
