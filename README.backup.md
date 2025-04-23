# Guide d'installation d'OpenCore Legacy Patcher

Ce guide explique la procédure complète d'installation d'OpenCore Legacy Patcher (OCLP) depuis le code source jusqu'à l'installation sur le disque interne et l'application des patchs post-installation.

## Table des matières

- [Prérequis](#prérequis)
- [Clonage du dépôt](#clonage-du-dépôt)
- [Configuration de l'environnement](#configuration-de-lenvironnement)
- [Mise à jour du dépôt](#mise-à-jour-du-dépôt)
- [Génération de l'EFI OpenCore](#génération-de-lefi-opencore)
- [Installation d'OpenCore sur le disque interne](#installation-dopencore-sur-le-disque-interne)
- [Application des patchs post-installation](#application-des-patchs-post-installation)
- [Script d'automatisation](#script-dautomatisation)
- [Dépannage](#dépannage)

## Prérequis

- macOS 10.13 ou ultérieur
- Python 3.6 ou plus récent (idéalement Python 3.11)
- Git
- Droits d'administrateur sur votre machine
- Sauvegarde complète de vos données

## Clonage du dépôt

Clonez le dépôt OpenCore Legacy Patcher sur votre Mac :

```bash
mkdir -p ~/Projets
cd ~/Projets
git clone https://github.com/dortania/OpenCore-Legacy-Patcher
cd OpenCore-Legacy-Patcher
```

## Configuration de l'environnement

### Installation de Python

Pour des résultats optimaux, utilisez Python depuis python.org, et non la version fournie avec macOS :

1. Téléchargez Python depuis [python.org](https://www.python.org/downloads/)
2. Installez-le en suivant les instructions
3. Vérifiez l'installation :

```bash
python3 --version
# Devrait afficher Python 3.11 ou plus récent
```

### Installation des dépendances

Installez les dépendances requises pour OCLP :

```bash
pip3 install -r requirements.txt
```

## Mise à jour du dépôt

Pour vous assurer d'avoir la dernière version d'OCLP (nightly build) :

```bash
git pull origin main
git status
# Devrait afficher "Your branch is up to date with 'origin/main'."
```

## Génération de l'EFI OpenCore

Générez un dossier EFI OpenCore spécifique à votre modèle de Mac. Par exemple, pour un iMac11,1 (iMac 27" fin 2009) :

```bash
python3 OpenCore-Patcher-GUI.command --build --model iMac11,1 --verbose
```

Le terminal affichera toutes les étapes de la génération. À la fin, vous verrez un message indiquant l'emplacement du dossier EFI généré :

```
Your OpenCore EFI for iMac11,1 has been built at:
    /Users/votre_utilisateur/Projets/OpenCore-Legacy-Patcher/Build-Folder/OpenCore-Build
```

## Installation d'OpenCore sur le disque interne

Vous pouvez installer OpenCore de deux façons :

### Méthode 1 : Via l'interface graphique

1. Lancez OpenCore Patcher :
   ```bash
   python3 OpenCore-Patcher-GUI.command
   ```
2. Cliquez sur "Build and Install OpenCore"
3. Sélectionnez votre modèle de Mac
4. Sélectionnez l'option d'installation sur le disque interne
5. Suivez les instructions à l'écran

### Méthode 2 : Via la ligne de commande

1. Identifiez la partition EFI de votre disque interne :
   ```bash
   diskutil list
   # Repérez la partition EFI (généralement disk0s1)
   ```

2. Montez la partition EFI :
   ```bash
   sudo diskutil mount disk0s1
   ```

3. Copiez l'EFI généré vers la partition EFI :
   ```bash
   sudo cp -R ~/Projets/OpenCore-Legacy-Patcher/Build-Folder/OpenCore-Build/EFI /Volumes/EFI/
   ```

4. Configurez le démarrage sur OpenCore :
   ```bash
   sudo bless --mount /Volumes/EFI --setBoot --file /Volumes/EFI/EFI/OC/OpenCore.efi
   ```

## Application des patchs post-installation

Après avoir installé macOS et démarré avec OpenCore, vous devez appliquer les patchs root pour activer complètement le support matériel :

### Méthode 1 : Via l'interface graphique

1. Lancez OpenCore Patcher dans macOS
2. Cliquez sur "Post-Install Root Patch"
3. Cliquez sur "Start Root Patching"
4. Suivez les instructions et redémarrez quand demandé

### Méthode 2 : Via la ligne de commande

```bash
python3 OpenCore-Patcher-GUI.command --patch_system --volume "/Volumes/macOS Sequoia 15.4.1"
```

Remplacez "/Volumes/macOS Sequoia 15.4.1" par le nom exact de votre volume système.

## Script d'automatisation

Voici un script bash qui automatise l'installation d'OpenCore sur le disque interne et l'application des patchs post-installation :

```bash
#!/bin/bash
set -e  # Arrête le script en cas d'erreur

# === Variables à adapter ===
EFI_BUILD_PATH="$HOME/Projets/OpenCore-Legacy-Patcher/Build-Folder/OpenCore-Build/EFI"
VOLUME_SYSTEM="/Volumes/macOS Sequoia 15.4.1"  # Nom exact du volume avec espace
OCLP_COMMAND="$HOME/Projets/OpenCore-Legacy-Patcher/OpenCore-Patcher-GUI.command"

# === 1. Identifier le disque interne principal ===
echo "Recherche du disque système..."
DISK_ID=$(diskutil list | grep -B 1 "APFS Container" | grep "GUID_partition_scheme" | awk '{print $6}' | sed 's/://')
EFI_PARTITION="${DISK_ID}s1"
echo "Partition EFI détectée : $EFI_PARTITION"

# === 2. Monter la partition EFI ===
echo "Montage de la partition EFI..."
sudo diskutil mount $EFI_PARTITION || { echo "Échec du montage de la partition EFI"; exit 1; }

# === 3. Copie de l'EFI généré ===
echo "Copie du dossier EFI vers /Volumes/EFI..."
sudo rm -rf "/Volumes/EFI/EFI" 2>/dev/null  # Ignore les erreurs si absent
sudo cp -R "$EFI_BUILD_PATH" "/Volumes/EFI/" || { echo "Échec de la copie de l'EFI"; exit 1; }

# === 4. Configuration du démarrage OpenCore ===
echo "Définition d'OpenCore comme bootloader par défaut..."
sudo bless --mount "/Volumes/EFI" --setBoot --file "/Volumes/EFI/EFI/OC/OpenCore.efi" || { echo "Échec de bless"; exit 1; }

# === 5. Patchs post-installation (optionnel) ===
echo "Application des patchs système sur '$VOLUME_SYSTEM'..."
python3 "$OCLP_COMMAND" --patch_system --volume "$VOLUME_SYSTEM" || { echo "Échec des patchs post-installation"; exit 1; }

echo "✅ Installation terminée avec succès !"
echo "Redémarrez avec : sudo reboot"
```

Pour utiliser ce script :
1. Enregistrez-le dans un fichier (par exemple `install_opencore.sh`)
2. Rendez-le exécutable : `chmod +x install_opencore.sh`
3. Exécutez-le: `sudo ./install_opencore.sh`

## Dépannage

### Réinitialisation du NVRAM

Si OpenCore ne démarre pas correctement, essayez de réinitialiser le NVRAM :
1. Redémarrez votre Mac
2. Maintenez enfoncées les touches ⌘ + ⌥ + P + R au démarrage
3. Relâchez après le second signal sonore

### Désinstallation d'OpenCore

Pour désinstaller OpenCore et revenir au démarrage normal :
1. Montez la partition EFI : `sudo diskutil mount disk0s1`
2. Supprimez le dossier EFI : `sudo rm -rf /Volumes/EFI/EFI`
3. Réinitialisez le NVRAM au prochain démarrage

### Vérification de l'installation

Si l'installation semble réussie mais que votre Mac ne démarre pas sur OpenCore :
1. Vérifiez que la partition EFI contient bien les fichiers OpenCore
2. Utilisez une clé USB de secours avec OpenCore installé
3. Consultez les logs dans : `/Users/votre_utilisateur/Library/Logs/Dortania/`

---

**Remarque importante** : OpenCore Legacy Patcher est un outil puissant qui permet d'installer des versions récentes de macOS sur des Mac non supportés. Cependant, il peut y avoir des limitations matérielles et certaines fonctionnalités peuvent ne pas fonctionner parfaitement. Utilisez-le à vos propres risques et gardez toujours une sauvegarde de vos données importantes.

Citations :
[1] Capture-decran-2025-04-17-a-18.53.52.jpg https://pplx-res.cloudinary.com/image/private/user_uploads/EnLYwXnbowsxxpA/Capture-decran-2025-04-17-a-18.53.52.jpg
