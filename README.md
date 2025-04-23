# OpenCore Legacy Patcher – Compilation Build Nightly sur macOS Intel

> **Attention : Build Nightly = version de développement**  
> Ces versions sont instables, non testées pour la production, et peuvent introduire des bugs ou changements majeurs. Utilisez-les uniquement pour contribuer ou tester les dernières nouveautés. **Sauvegardez toujours vos données avant manipulation du bootloader.**

---

## Prérequis

- macOS Intel (10.13 ou ultérieur, patché si besoin)
- Python 3.13.x (version officielle [python.org](https://www.python.org/downloads/), compilée avec `--enable-shared`)
- Xcode Command Line Tools (`xcode-select --install`)
- Git
- Connexion internet stable
- Droits administrateur

---

## 1. Clonage du dépôt Build Nightly

```
git clone https://github.com/dortania/OpenCore-Legacy-Patcher.git
cd OpenCore-Legacy-Patcher
```
*La branche `main` contient les dernières nightly.*

---

## 2. (Optionnel) Compilation manuelle d’OpenCore EFI avec gmake

Si vous souhaitez compiler vous-même OpenCore EFI (ou ses kexts) pour tester une version spécifique ou contribuer au bootloader :

```
# Installer GNU Make si besoin
brew install make

# Cloner le dépôt OpenCorePkg (ou Lilu, WhateverGreen, etc.)
git clone https://github.com/acidanthera/OpenCorePkg.git
cd OpenCorePkg

# Compiler avec gmake
gmake DEBUG=FALSE

# Le binaire EFI généré se trouve dans X64/EFI/OC/OpenCore.efi
# Vous pouvez ensuite remplacer l’EFI généré par OCLP par le vôtre dans la partition EFI ou le dossier de build.
```
> *Cette étape est réservée aux utilisateurs avancés. Pour la majorité des utilisateurs, OCLP intègre automatiquement les dernières versions officielles d’OpenCore EFI.*

---

## 3. Préparation de l’environnement Python

```
python3 --version  # Doit afficher Python 3.13.x
python3 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
pip install pyinstaller
```
> **Recommandé :** Utilisez toujours un venv propre pour éviter les conflits de dépendances.

---

## 4. Correction de l’architecture PyInstaller (x86_64)

Sur Mac Intel, PyInstaller doit cibler **x86_64** uniquement (et non `universal2`), sinon vous aurez des erreurs du type `is not a fat binary!`.

```
sed -i.bak 's/target_arch="universal2"/target_arch="x86_64"/g' OpenCore-Patcher-GUI.spec
```
Vérifiez :
```
grep 'target_arch' OpenCore-Patcher-GUI.spec
# Doit afficher : target_arch="x86_64"
```

---

## 5. Gestion des modules natifs problématiques

Certains modules Python (ex : `charset_normalizer`) peuvent générer des `.so` non universels.  
Forcez leur réinstallation en x86_64 :

```
arch -x86_64 pip install --force-reinstall --no-binary :all: charset-normalizer
```
Vérifiez l’absence de `.so` non x86_64 :
```
find venv/lib/python3.13/site-packages/charset_normalizer -name "*.so" -exec file {} \; | grep -v "x86_64"
# Rien ne doit s’afficher
```

---

## 6. Compilation du projet (Build Nightly)

Lancez la génération de l’application :
```
python3 Build-Project.command
```
- L’application sera générée dans `build/OpenCore-Patcher-GUI/` ou `dist/`.
- En cas d’erreur `is not a fat binary!`, vérifiez que TOUS les modules natifs sont bien en x86_64 et que la spec PyInstaller est correcte.

---

## 7. Lancement de l’application

```
./build/OpenCore-Patcher-GUI/OpenCore-Patcher
```
- Rendez-le exécutable si besoin : `chmod +x ./build/OpenCore-Patcher-GUI/OpenCore-Patcher`

---

## 8. Utilisation et limites du Build Nightly

- **Fonctionnalités en test** : Les nightly peuvent contenir des fonctionnalités non documentées ou instables. Consultez le [changelog](https://github.com/dortania/OpenCore-Legacy-Patcher/releases) et [SOURCE.md](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/SOURCE.md).
- **Compatibilité** : Les nightly peuvent introduire des changements de structure, de dépendances, ou de comportement (EFI, patchs, GUI…).
- **Support** : Les bugs rencontrés doivent être rapportés sur GitHub avec le maximum d’informations (log complet, version, modèle de Mac).

---

## 9. Installation de l’EFI OpenCore

1. **Montez la partition EFI** :
   ```
   diskutil list  # repérez la partition EFI, ex : disk0s1
   sudo diskutil mount disk0s1
   ```
2. **Copiez l’EFI généré** :
   ```
   sudo cp -R Build-Folder/OpenCore-Build/EFI /Volumes/EFI/
   ```
3. **Définissez OpenCore comme bootloader** :
   ```
   sudo bless --mount /Volumes/EFI --setBoot --file /Volumes/EFI/EFI/OC/OpenCore.efi
   ```

---

## 10. Patchs post-installation

- **Via l’interface graphique** :  
  Lancez OCLP, cliquez sur "Post-Install Root Patch", puis "Start Root Patching".
- **Via la ligne de commande** :  
  ```
  python3 OpenCore-Patcher-GUI.command --patch_system --volume "/Volumes/NOM_DU_VOLUME"
  ```

---

## 11. Dépannage courant

- **Erreur “is not a fat binary!”** :  
  - Vérifiez que la spec PyInstaller cible bien `x86_64`
  - Recompilez tous les modules natifs problématiques avec `arch -x86_64`
- **Erreur wxPython** :  
  - Réinstallez wxPython dans le venv
- **Problème de boot** :  
  - Vérifiez la partition EFI, réinitialisez la NVRAM

---

## 12. Ressources et documentation

- [Documentation officielle Dortania](https://dortania.github.io/OpenCore-Legacy-Patcher/)
- [SOURCE.md (Build Nightly)](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/SOURCE.md)
- [Guide valorisa](https://github.com/valorisa/OpenCore-EFI-Installer-Automation)
- [Compatibilité modèles](https://dortania.github.io/OpenCore-Legacy-Patcher/MODELS.html)

---

## Résumé

Cette procédure permet de compiler et d'utiliser la **Build Nightly** d’OpenCore Legacy Patcher sur un Mac Intel, en tenant compte des exigences de l’architecture x86_64 et des spécificités des versions de développement.  
**N’utilisez les nightly que si vous acceptez les risques d’instabilité et souhaitez contribuer au projet ou tester les nouveautés.**
