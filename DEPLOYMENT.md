# 🚀 Déploiement GitHub Pages

Ce guide explique comment déployer le guide AZ-104 sur GitHub Pages.

## 📋 Prérequis

1. **Repository GitHub** : Votre repository doit être public ou vous devez avoir GitHub Pro
2. **Branche main** : Le code doit être sur la branche `main`
3. **Permissions** : Vous devez avoir les permissions pour activer GitHub Pages

## 🔧 Configuration

### 1. Activer GitHub Pages

1. Allez dans les **Settings** de votre repository
2. Cliquez sur **Pages** dans le menu de gauche
3. Dans **Source**, sélectionnez **Deploy from a branch**
4. Choisissez la branche `gh-pages` et le dossier `/ (root)`
5. Cliquez sur **Save**

### 2. Activer GitHub Actions

1. Allez dans l'onglet **Actions** de votre repository
2. Le workflow `Deploy to GitHub Pages` devrait s'exécuter automatiquement
3. Si ce n'est pas le cas, allez dans **Settings > Actions > General**
4. Assurez-vous que **Actions permissions** est activé

### 3. Vérifier les Permissions

1. Allez dans **Settings > Actions > General**
2. Dans **Workflow permissions**, sélectionnez **Read and write permissions**
3. Cochez **Allow GitHub Actions to create and approve pull requests**
4. Cliquez sur **Save**

## 🔄 Déploiement Automatique

Le déploiement se fait automatiquement grâce au workflow GitHub Actions :

1. **Push sur main** : Chaque push sur la branche `main` déclenche le déploiement
2. **Build automatique** : Le README.md est converti en HTML avec un style GitHub
3. **Déploiement** : Le site est déployé sur la branche `gh-pages`

## 🌐 Accès au Site

Une fois déployé, votre site sera accessible à :
```
https://[votre-username].github.io/az-104
```

## 🔧 Configuration Avancée

### Personnaliser le Style

Modifiez le fichier `.github/workflows/deploy.yml` pour changer le style CSS.

### Ajouter des Métadonnées

Modifiez le fichier `_config.yml` pour personnaliser les métadonnées du site.

### Ajouter des Images

1. Créez un dossier `assets/images/`
2. Ajoutez vos images
3. Référencez-les dans le markdown : `![Alt](assets/images/image.png)`

## 🐛 Dépannage

### Le site ne se déploie pas

1. Vérifiez les **Actions** dans l'onglet Actions
2. Regardez les logs du workflow pour identifier l'erreur
3. Vérifiez que la branche `gh-pages` a été créée

### Erreur de permissions

1. Vérifiez les permissions dans **Settings > Actions > General**
2. Assurez-vous que le repository est public ou que vous avez GitHub Pro

### Le style ne s'affiche pas

1. Vérifiez que le workflow a bien généré le fichier `index.html`
2. Regardez la console du navigateur pour les erreurs CSS/JS

## 📝 Structure des Fichiers

```
az-104/
├── README.md              # Guide principal
├── index.md               # Page d'accueil
├── _config.yml            # Configuration Jekyll
├── .github/
│   └── workflows/
│       └── deploy.yml     # Workflow de déploiement
├── .gitignore             # Fichiers à ignorer
└── DEPLOYMENT.md          # Ce fichier
```

## 🔗 Liens Utiles

- [Documentation GitHub Pages](https://docs.github.com/pages/)
- [Documentation GitHub Actions](https://docs.github.com/actions/)
- [Documentation Jekyll](https://jekyllrb.com/docs/)
- [GitHub Markdown CSS](https://github.com/sindresorhus/github-markdown-css)

## 📞 Support

Si vous rencontrez des problèmes :

1. Vérifiez les **Issues** du repository
2. Créez une nouvelle **Issue** avec les détails du problème
3. Incluez les logs d'erreur si possible 