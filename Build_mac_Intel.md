---

# Procédure de compilation d’OpenCore Legacy Patcher sur macOS Intel

## Objectif

Ce guide explique comment compiler OpenCore Legacy Patcher sur un Mac Intel, en évitant les erreurs liées à l’architecture des binaires Python lors de l’utilisation de PyInstaller.

---

## Prérequis

- macOS (Intel)
- Python 3.13 installé (préférablement depuis python.org)
- Xcode Command Line Tools installés
- `pip` et `venv` disponibles

---

## Étapes détaillées

### 1. Créer et activer un environnement virtuel

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 2. Installer les dépendances

```bash
pip install -r requirements.txt
pip install pyinstaller
```

### 3. Forcer PyInstaller à cibler uniquement l’architecture x86_64

Créez le script suivant `fix-pyinstaller-arch.sh` :

```bash
#!/bin/bash
SPEC_FILE="OpenCore-Patcher-GUI.spec"
if grep -q 'target_arch="universal2"' "$SPEC_FILE"; then
    sed -i.bak 's/target_arch="universal2"/target_arch="x86_64"/g' "$SPEC_FILE"
    echo "✅ Architecture forcée sur x86_64 dans $SPEC_FILE"
else
    echo "ℹ️  Aucun target_arch=\"universal2\" trouvé dans $SPEC_FILE"
fi
if ! grep -q 'target_arch="x86_64"' "$SPEC_FILE"; then
    sed -i.bak '/^exe = EXE(/ s/$/, target_arch="x86_64"/' "$SPEC_FILE"
    echo "✅ Ajout de target_arch=\"x86_64\" à la ligne EXE"
fi
```

Rendez-le exécutable et lancez-le :

```bash
chmod +x fix-pyinstaller-arch.sh
./fix-pyinstaller-arch.sh
```

### 4. Installer wxPython (si nécessaire)

```bash
pip uninstall wxPython
pip install wxPython
```

Vérifiez l’installation :

```bash
python3 -c "import wx; print(wx.__version__)"
```

### 5. Compiler le projet

```bash
python3 Build-Project.command
```

---

## Explications

- **Pourquoi cette procédure ?**  
  PyInstaller, lorsqu’il cible l’architecture universelle (`universal2`), exige que toutes les bibliothèques Python compilées soient des "fat binaries" (x86_64 + arm64). Or, la plupart des wheels installés via pip sont uniquement x86_64 sur Mac Intel, ce qui provoque l’erreur « is not a fat binary ».
- **Pourquoi un environnement virtuel ?**  
  Cela isole les dépendances et garantit la reproductibilité.
- **Pourquoi le script de correction ?**  
  Il automatise la modification du `.spec` pour éviter toute tentative de build universel.

---

## Dépannage

- Si une erreur « ModuleNotFoundError: No module named 'wx' » apparaît, réinstallez wxPython comme indiqué ci-dessus.
- Si une erreur « is not a fat binary » persiste, vérifiez que le script de correction a bien été appliqué et que PyInstaller cible uniquement x86_64.

---

## Pour aller plus loin

Pour générer un binaire universel compatible Apple Silicon, il faut :
- Utiliser un environnement Python universel (python.org)
- S’assurer que toutes les dépendances sont elles-mêmes universelles (ce qui est rarement le cas pour les modules natifs)

---

**Ajoutez ce fichier à la racine du projet, à côté du `README.md`, pour faciliter la compilation sur Mac Intel et le partage de la procédure avec d’autres contributeurs.**

Citations:
[1] https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/40251661/77495420-01be-45f0-b10f-293baeea71bb/paste.txt
[2] https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/40251661/c7603731-3c19-426d-b3ad-808ebcb862f3/paste-2.txt
[3] https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/40251661/af3854a5-20d3-4207-8783-39b62356c699/paste-3.txt
[4] https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/40251661/10273645-5484-4b54-b64f-4741949ec71b/paste-4.txt
[5] https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/40251661/6288222c-0ea4-4225-bac3-dce84981c9d1/paste-5.txt
[6] https://stackoverflow.com/questions/76277947/i-have-a-markdown-file-that-has-errors-and-need-to-use-a-python3-12-to-find-the
[7] https://github.com/tmarktaylor/phmutest
[8] https://realpython.com/django-markdown/
[9] https://www.honeybadger.io/blog/python-markdown/
[10] https://python-markdown.github.io
[11] https://superuser.com/questions/323657/how-to-get-the-python-markdown-command
[12] https://github.com/Python-Markdown/markdown
[13] https://pypi.org/project/Markdown/
[14] https://packaging.python.org/guides/making-a-pypi-friendly-readme/
[15] https://www.jetbrains.com/help/pycharm/markdown.html

---
