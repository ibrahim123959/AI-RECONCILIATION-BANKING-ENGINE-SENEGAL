# Public Assets

Assets statiques servis directement sans processing par Vite.

## Usage

Fichiers dans `public/` sont copiés tels quels dans `dist/` lors du build.

### Accès dans HTML
```html
<!-- index.html -->
<link rel="icon" type="image/svg+xml" href="/favicon.ico" />
```

### Accès dans Code
```javascript
// ❌ N'IMPORTE pas les fichiers public/
import logo from '/logo.png' // ERREUR

// ✅ Référencer via URL absolue
<img src="/logo.png" alt="Logo" />
```

## Structure
```
public/
├── index.html              # Template HTML principal
├── favicon.ico             # Favicon site
├── logo.png                # Logo application
└── robots.txt              # SEO (optionnel)
```

## index.html

Template HTML principal avec injection Vite :
```html
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8" />
  <link rel="icon" type="image/x-icon" href="/favicon.ico" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Reconciliation Engine</title>
</head>
<body>
  <div id="root"></div>
  <script type="module" src="/src/main.jsx"></script>
</body>
</html>
```

## Favicon

Générer favicons multi-résolutions :
```bash
# Utiliser tool en ligne
# https://realfavicongenerator.net/

# Ou avec imagemagick
convert logo.png -resize 32x32 favicon-32x32.png
convert logo.png -resize 16x16 favicon-16x16.png
```

## Best Practices

### ✅ DO
- Mettre assets statiques ne nécessitant pas processing
- Petits fichiers (<100KB)
- Assets référencés dans HTML

### ❌ DON'T
- Mettre composants React
- Mettre assets importés dans code JS
- Mettre fichiers sensibles (secrets)

## Assets vs Public

| Location | Usage | Processing | Import |
|----------|-------|------------|--------|
| `src/assets/` | Images dans components | ✅ Optimized | `import img from './img.png'` |
| `public/` | Static files | ❌ None | `<img src="/img.png" />` |