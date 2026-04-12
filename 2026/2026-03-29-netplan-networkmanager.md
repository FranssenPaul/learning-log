# Netplan & NetworkManager sur Ubuntu
*learning-log — réseau Linux*

---

## Le problème conceptuel de départ

Sur Ubuntu desktop, quand tu ouvres les paramètres réseau et que tu ne vois rien — pas de WiFi, pas d'Ethernet, juste "VPN" et "Proxy" — c'est rarement une panne matérielle. C'est presque toujours un **problème de couche de gestion** : quelque chose a pris le contrôle du réseau sans que NetworkManager le sache.

Pour comprendre pourquoi, il faut saisir l'architecture en couches du réseau sous Linux.

---

## Les trois niveaux à distinguer

### 1. Le matériel et le kernel

Le kernel Linux détecte les interfaces physiques (carte réseau, puce WiFi) et leur donne un nom : `eth0`, `eno1`, `wlan0`, `enp3s0`... Ces noms ont évolué vers un système "prévisible" basé sur la topologie matérielle — c'est pourquoi on voit `eno1` (onboard) ou `enp0s25` (PCI bus 0, slot 25).

À ce niveau, l'interface existe mais elle n'a ni adresse IP, ni route, ni DNS. Elle est comme une prise murale : présente, mais non connectée.

### 2. Le gestionnaire réseau

C'est ici qu'intervient la séparation fondamentale. Linux propose **deux systèmes concurrents** pour configurer les interfaces :

- **`systemd-networkd`** — minimaliste, orienté serveur, piloté par des fichiers de config statiques
- **`NetworkManager`** — riche, orienté desktop, avec GUI GNOME, gestion WiFi, VPN, profils utilisateur

Ces deux systèmes ne se partagent pas le travail : **un seul prend le contrôle d'une interface donnée**. Si les deux essaient de gérer la même interface, c'est le chaos.

> 💡 **Analogie** : imagine deux chefs de cuisine dans la même cuisine. Soit tu définis clairement qui gère quoi, soit les plats ne sortent jamais. NetworkManager et networkd, c'est pareil.

### 3. Netplan — le configurateur de configurateur

Ubuntu a introduit **Netplan** comme couche d'abstraction au-dessus des deux. Netplan lit des fichiers YAML et *génère* la configuration pour l'un ou l'autre backend. C'est lui qui décide **qui sera chef de cuisine**.

```
Fichiers YAML (/etc/netplan/*.yaml)
           ↓
         Netplan
        ↙       ↘
NetworkManager   systemd-networkd
        ↓               ↓
   Interfaces réseau (kernel)
```

---

## Le paramètre clé : `renderer`

Dans chaque fichier YAML netplan, un seul paramètre détermine tout :

```yaml
network:
  version: 2
  renderer: NetworkManager   # ← donne le contrôle à NM
```

ou :

```yaml
network:
  version: 2
  renderer: networkd         # ← donne le contrôle à systemd-networkd
```

**Un desktop Ubuntu standard** utilise `renderer: NetworkManager` — ce qui permet à GNOME d'afficher les interfaces, de gérer le WiFi, de basculer entre réseaux.

**Un serveur ou un système embarqué** (comme un nœud POS sur LAN) utilise `renderer: networkd` avec une IP statique définie directement dans le YAML — plus léger, aucune dépendance à une session utilisateur.

---

## Ce qui s'est passé concrètement

La machine `sirocco` avait ce fichier netplan :

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eno1:
      dhcp4: no
      addresses: [192.168.129.25/23]
      routes:
        - to: default
          via: 192.168.128.1
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
```

C'était une config **serveur POSEN** : IP statique, pas de DHCP, piloté par `networkd`. Résultat : NetworkManager ne voyait pas `eno1` (marquée `unmanaged`), GNOME affichait une page réseau vide, et le message d'erreur disait `NetworkManager needs to be running` — alors qu'il tournait très bien, il n'avait simplement rien à gérer.

---

## La résolution : rendre le contrôle à NetworkManager

Remplacer le contenu du fichier YAML par la config minimale desktop :

```yaml
network:
  version: 2
  renderer: NetworkManager
```

Puis appliquer :

```bash
sudo netplan apply
sudo systemctl restart NetworkManager
```

Netplan régénère les configs backend, NetworkManager reprend `eno1`, DHCP s'active, les interfaces réapparaissent dans GNOME.

---

## Les fichiers qui comptent

| Chemin | Rôle |
|--------|------|
| `/etc/netplan/*.yaml` | Source de vérité — c'est ici qu'on touche |
| `/run/systemd/network/` | Config générée par netplan pour networkd |
| `/run/NetworkManager/` | Config générée par netplan pour NM |
| `/etc/NetworkManager/NetworkManager.conf` | Config globale de NM (rarement à modifier) |

> ⚠️ S'il existe **plusieurs fichiers** dans `/etc/netplan/`, ils sont fusionnés par ordre alphabétique. Un `99-custom.yaml` peut écraser un `01-network-manager-all.yaml`. Toujours vérifier avec `sudo ls /etc/netplan/`.

---

## Deux usages, deux philosophies

| Contexte | Renderer | Pourquoi |
|----------|----------|---------|
| Desktop / laptop | `NetworkManager` | GUI, WiFi, profils, hotspots |
| Serveur / POS / LAN fixe | `networkd` | Léger, déterministe, sans GUI |
| Raspberry Pi / IoT | `networkd` | Même raison |
| VM cloud (cloud-init) | `networkd` | Config automatique au boot |

---

## Commandes de diagnostic essentielles

```bash
# Voir les interfaces et leur état physique
ip link show

# Voir les IPs attribuées
ip addr show

# Voir ce que NetworkManager connaît
nmcli device status

# État du service NetworkManager
sudo systemctl status NetworkManager

# Appliquer les changements netplan
sudo netplan apply

# Tester la connectivité
ping 8.8.8.8
```

---

## Ce qu'il faut retenir

Quand le réseau disparaît sous Ubuntu, la première question n'est pas *"est-ce que le câble est branché ?"* mais *"qui est censé gérer cette interface ?"*. NetworkManager, networkd, et netplan forment une chaîne — une rupture dans cette chaîne suffit à rendre une interface invisible à GNOME, même si elle est physiquement active (`UP, LOWER_UP` dans `ip link show`).

La commande `ip link show` dit ce que le **kernel** voit. `nmcli device status` dit ce que **NetworkManager** voit. Si l'un voit et l'autre pas, le problème est dans la couche de gestion — et la réponse est dans `/etc/netplan/`.
