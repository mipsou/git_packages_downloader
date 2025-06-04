[English Version](README.en.md)
#  роль Ansible: git_packages_downloader 📥

[![Ansible Galaxy](https://img.shields.io/badge/Ansible%20Galaxy-mipsou.git__packages__downloader-blue?style=flat-square)](https://galaxy.ansible.com/mipsou/git_packages_downloader)
[![GitHub tag](https://img.shields.io/github/v/tag/mipsou/git_packages_downloader?sort=semver&style=flat-square)](https://github.com/mipsou/git_packages_downloader/tags)
[![Ansible Lint](https://github.com/mipsou/git_packages_downloader/actions/workflows/lint.yml/badge.svg)](https://github.com/mipsou/git_packages_downloader/actions/workflows/lint.yml)
[![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg?style=flat-square)](LICENSE)

Ce rôle Ansible télécharge et met en miroir les derniers artefacts de publication depuis des dépôts basés sur Git (par exemple, GitHub) vers un dépôt RPM local.

## ✨ Fonctionnalités

- Télécharge dynamiquement les releases `.rpm` (ou tout autre format) depuis GitHub 📦
- Filtre par architecture et extension (par exemple, `amd64`, `aarch64`)  фильтр
- Idempotent et compatible avec Foreman ✅
- Prend en charge la création de dépôts locaux et les contextes SELinux 🛡️

## 📋 Prérequis

- Le paquet `python3-jmespath` doit être installé sur l'hôte cible où ce rôle est exécuté. Le rôle inclut une tâche pour tenter son installation.
- Si vous utilisez SELinux, les outils nécessaires (`restorecon`, `semanage`) doivent être disponibles sur la cible.

## ⚙️ Configuration

Ce rôle est configuré principalement via la variable `git_packages`, ou alternativement via `corp_mirror_git` pour les configurations centralisées.

### Définition des Paquets (`git_packages`)

La variable `git_packages` est une liste de dictionnaires, où chaque dictionnaire définit un paquet à télécharger et à gérer. Par défaut, cette liste est vide (`git_packages: []`), vous devez donc la définir pour utiliser le rôle.

**Structure pour chaque élément de paquet :**

*   `name` (chaîne, requis) : Un nom descriptif pour le paquet (par exemple, "mercure", "mon-app"). Utilisé pour les logs et les étiquettes.
*   `source` (dictionnaire, requis) : Définit d'où télécharger le paquet.
    *   `provider` (chaîne, requis) : Le type de fournisseur Git. Actuellement, `"github"` est pris en charge.
    *   `owner` (chaîne, requis pour GitHub) : Le propriétaire ou l'organisation du dépôt GitHub (par exemple, `dunglas`).
    *   `repo` (chaîne, requis pour GitHub) : Le nom du dépôt GitHub (par exemple, `mercure`).
    *   `arch` (chaîne, requis pour GitHub) : L'architecture à filtrer dans les artefacts de la release (par exemple, `amd64`, `x86_64`, `aarch64`, `noarch`).
    *   `ext` (chaîne, requis pour GitHub) : L'extension de fichier à filtrer (par exemple, `rpm`, `deb`, `tar.gz`).
*   `repository` (dictionnaire, requis) : Définit les détails du dépôt local.
    *   `name` (chaîne, requis) : Le nom du répertoire du dépôt local à créer (par exemple, `mercure`, `mon-depot-perso`). Ce sera un sous-répertoire sous `base_path`.
    *   `base_path` (chaîne, requis) : Le chemin de base sur l'hôte cible où le répertoire du dépôt sera créé (par exemple, `/var/www/html/public/repos`, `/opt/mirrors`).
    *   `mode` (chaîne, optionnel, défaut : `"0755"`) : Permissions pour le répertoire du dépôt.
    *   `owner` (chaîne, optionnel, défaut : `apache`) : Propriétaire du répertoire du dépôt et des fichiers téléchargés. Les alternatives courantes incluent `nginx` ou `www-data` selon votre serveur web.
    *   `group` (chaîne, optionnel, défaut : `apache`) : Groupe pour le répertoire du dépôt et des fichiers téléchargés. Les alternatives courantes incluent `nginx` ou `www-data`.
    *   `selinux_enable` (booléen, optionnel, défaut : `false`) : Indique s'il faut gérer les contextes SELinux pour le chemin du dépôt (définit `httpd_sys_content_t`).

**Exemple de définition `git_packages` :**

```yaml
git_packages:
  - name: mercure_rpm # Nom descriptif du paquet
    source:
      provider: github
      owner: dunglas
      repo: mercure
      arch: amd64 # Peut aussi être x86_64, etc.
      ext: rpm
    repository:
      name: mercure # Ceci créera /var/www/html/public/repos/mercure
      base_path: /var/www/html/public/repos # Chemin commun accessible via le web
      mode: "0755"
      owner: apache # Ou nginx, www-data, etc., selon l'utilisateur de votre serveur web
      group: apache # Ou nginx, www-data, etc.
      selinux_enable: true
  - name: another_tool_tarball
    source:
      provider: github
      owner: someuser
      repo: sometool
      arch: x86_64 # Assurez-vous que cela correspond à une partie du nom de l'artefact
      ext: tar.gz
    repository:
      name: sometool_archive
      base_path: /opt/pkg_archives # Exemple pour les archives non accessibles via le web
      owner: root # Propriétaire/groupe différent pour le contenu non web
      group: root
      # selinux_enable est false par défaut si non spécifié
```

## 🏢 Configuration Centralisée & Intégration avec The Foreman

Pour gérer les listes de paquets de manière centralisée, en particulier dans des environnements comme The Foreman, ce rôle prend en charge une méthode de configuration alternative utilisant la variable `corp_mirror_git`.

### Utilisation de `corp_mirror_git`

Si vous définissez une variable nommée `corp_mirror_git`, et que cette variable contient une clé `git_packages`, le rôle utilisera la liste fournie sous `corp_mirror_git.git_packages` comme source pour les paquets à télécharger. Cela surchargera toute définition directe de la variable `git_packages`.

Ceci est utile pour maintenir une source unique de vérité pour vos définitions de paquets à travers plusieurs hôtes ou environnements.

**Structure pour `corp_mirror_git`:**

La variable `corp_mirror_git` doit être un dictionnaire contenant une seule clé, `git_packages`. La valeur de cette clé `git_packages` doit être une liste structurée exactement comme décrit dans la section "Définition des Paquets (`git_packages`)" ci-dessus.

**Exemple de définition `corp_mirror_git` :**

```yaml
corp_mirror_git:
  git_packages:
    - name: mercure_rpm_from_corp_mirror # Nom descriptif
      source:
        provider: github
        owner: dunglas
        repo: mercure
        arch: amd64
        ext: rpm
      repository: # Assurer que cette clé est présente et correctement indentée
        name: mercure_corp
        base_path: /var/www/html/public/repos/corporate # Chemin accessible via le web
        owner: apache # Ou nginx, www-data, etc.
        group: apache # Ou nginx, www-data, etc.
        selinux_enable: true
    - name: utility_scripts_from_corp_mirror
      source:
        provider: github
        owner: my-org
        repo: internal-utils
        arch: noarch # Exemple pour aucune architecture spécifique
        ext: tar.gz
      repository:
        name: internal_utils
        base_path: /srv/internal_repo/utils
        owner: ansible_svc_user # Exemple avec un utilisateur de service spécifique
        group: ansible_svc_user
```

### Configuration dans The Foreman 🛠️

Pour utiliser ce rôle avec The Foreman, l'approche recommandée est de définir la variable `corp_mirror_git` comme une Variable Ansible dans Foreman. Cela vous permet de gérer votre liste de paquets de manière centralisée.

**Étapes :**

1.  Dans Foreman, naviguez vers **Configurer > Ansible > Variables**.
2.  Cliquez sur **Créer une Variable** (ou sélectionnez une variable existante pour la surcharger au contexte souhaité comme Groupe d'hôtes, SE, ou Hôte spécifique).
3.  **Nom de la Variable :** `corp_mirror_git`
4.  **Valeur :** Collez le contenu YAML complet pour votre structure `corp_mirror_git` (comme montré dans l'exemple "Utilisation de `corp_mirror_git`" ci-dessus) dans le champ de valeur.
    *   *Assurez-vous que le YAML est correctement formaté dans le champ de valeur.*
    *   *Note : La variable `git_packages` elle-même n'est généralement pas gérée comme un Paramètre Intelligent modifiable en raison de sa structure complexe.*

Cette méthode garantit que Foreman transmet toute la configuration des paquets au rôle.

## 💡 Philosophie de Conception

Ce rôle est portable, déclaratif et évite le codage en dur. Il est conçu pour s'intégrer facilement avec Foreman mais fonctionne également sans.

## 📜 Licence

Apache License 2.0 — © Mipsou  
Voir [LICENSE](LICENSE) pour les détails.
