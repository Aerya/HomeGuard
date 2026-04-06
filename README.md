# WireGuard VPN — Docker + iOS / Android

Serveur WireGuard en Docker pour utiliser son propre serveur DNS (AdGuard Home, Pi-hole…) depuis un smartphone, même en dehors du réseau local.

## Principe

```
Smartphone (4G / WiFi externe)
    │  tunnel WireGuard chiffré
    ▼
Serveur WireGuard (chez soi)
    │  requête DNS interne
    ▼
Serveur DNS local (AdGuard Home, Pi-hole…)
```

- Hors du réseau : le VPN s'active, tout le DNS passe par le serveur maison
- Sur le WiFi maison : le VPN se désactive automatiquement, le DNS local est joignable directement

## Prérequis

- Docker + Docker Compose sur le serveur
- Un nom de domaine ou une IP publique pointant vers la box (DynDNS recommandé si IP variable)
- Port UDP `51820` ouvert et forwardé vers le serveur sur la box
- Le DHCP du réseau local doit pousser l'IP du serveur DNS aux clients (pour que le smartphone l'utilise directement quand il est à la maison)

## Configuration

Éditer `docker-compose.yml` et remplacer les deux valeurs suivantes :

| Variable | Description | Exemple |
|---|---|---|
| `SERVERURL` | Domaine ou IP publique du serveur VPN | `vpn.mondomaine.com` |
| `PEERDNS` | DNS principal + fallback | `192.168.1.1,9.9.9.9` |

## Démarrage

```bash
docker compose up -d
```

## Ajouter un peer (appareil)

Les peers sont définis dans la variable `PEERS` du `docker-compose.yml` (noms séparés par des virgules) :

```yaml
- PEERS=iphone,laptop,tablet
```

Après modification, relancer le conteneur :

```bash
docker compose up -d --force-recreate
```

## Récupérer la config d'un peer (QR code)

```bash
docker exec -it wireguard /app/show-peer iphone
```

Un QR code ASCII s'affiche dans le terminal — à scanner depuis l'app WireGuard sur le smartphone.

## Configuration iOS (iPhone / iPad)

1. Installer **WireGuard** depuis l'[App Store](https://apps.apple.com/fr/app/wireguard/id1441195209)
2. Ouvrir l'app → **`+`** (en haut à droite) → **Créer depuis un QR code**
3. Scanner le QR code affiché dans le terminal
4. Donner un nom (ex : `Maison`) → **Ajouter la configuration VPN** → Autoriser

### Activer le VPN automatiquement (sauf sur le WiFi maison)

Dans l'app WireGuard, appuyer sur le tunnel puis **On-Demand Activation** :

1. Activer **Wi-Fi** et **Cellular**
2. **Add rule** → **SSID** → saisir le nom exact du WiFi maison → action **Disconnect**
3. Sauvegarder

Le VPN s'activera automatiquement sur tous les réseaux sauf le WiFi maison. Dès que vous rentrez, il se coupe tout seul.

## Configuration Android

1. Installer **WireGuard** depuis le [Play Store](https://play.google.com/store/apps/details?id=com.wireguard.android)
2. **`+`** → **Scanner depuis QR code**
3. Scanner le QR code affiché dans le terminal
4. Donner un nom → Autoriser l'ajout du VPN

### Activer le VPN automatiquement (sauf sur le WiFi maison)

L'app WireGuard Android ne gère pas nativement les règles par SSID. Deux options :

- **Manuellement** : activer/désactiver le tunnel dans l'app selon le réseau
- **Automatiquement** avec [Tasker](https://play.google.com/store/apps/details?id=net.dinglisch.android.taskerm) (payant) ou [MacroDroid](https://play.google.com/store/apps/details?id=com.arlosoft.macrodroid) (gratuit) : créer un profil qui désactive le VPN quand le SSID maison est détecté, et le réactive sinon
