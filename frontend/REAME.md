# Frontend - Reconciliation Engine UI

Interface utilisateur React pour la réconciliation bancaire.

## Structure
```
frontend/
├── public/                 # Assets statiques
├── src/                    # Code source
│   ├── pages/              # Pages principales
│   ├── components/         # Composants réutilisables
│   ├── services/           # API clients
│   ├── hooks/              # Custom React hooks
│   ├── store/              # State management (Zustand)
│   └── utils/              # Utilities
├── tests/                  # Tests
├── Dockerfile
├── package.json
├── vite.config.js
└── tailwind.config.js
```

## Quick Start

### Installation
```bash
# Installer dependencies
npm install

# Ou avec yarn
yarn install
```

### Configuration
```bash
# Copier template
cp .env.example .env

# Éditer .env
VITE_API_BASE_URL=http://localhost:8000/api/v1
```

### Development
```bash
# Lancer dev server avec hot reload
npm run dev

# Ouvrir dans navigateur
open http://localhost:5173
```

### Build Production
```bash
# Build optimisé
npm run build

# Preview build
npm run preview
```

## Scripts Disponibles

| Script | Description |
|--------|-------------|
| `npm run dev` | Dev server (port 5173) |
| `npm run build` | Build production |
| `npm run preview` | Preview build production |
| `npm run lint` | ESLint check |
| `npm run lint:fix` | ESLint fix auto |
| `npm test` | Run tests unitaires |
| `npm run test:e2e` | Run tests e2e (Playwright) |

## Stack Technique

| Tech | Version | Usage |
|------|---------|-------|
| **React** | 18 | UI library |
| **Vite** | 5 | Build tool (rapide) |
| **Tailwind CSS** | 3.4 | Styling |
| **Zustand** | 4.4 | State management |
| **Axios** | 1.6 | HTTP client |
| **React Router** | 6 | Navigation |

## Features Principales

### 1. Upload Fichiers
- Drag & drop upload
- Support PDF, CSV, Excel
- Preview fichiers uploadés
- Progress tracking

### 2. Matching Dashboard
- Vue globale matches
- Filtres par confiance (high/medium/low)
- Statistiques temps réel
- Export résultats

### 3. Validation Manuelle
- Vue côte-à-côte (bank vs ledger)
- Approve/Reject/Modify
- Commentaires validation
- Historique validations

### 4. Reports & Analytics
- Dashboard métriques
- Charts interactifs
- Export Excel/PDF
- Historique réconciliations

## Configuration

### Vite Config (vite.config.js)
```javascript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  server: {
    port: 5173,
    proxy: {
      '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true
      }
    }
  }
})
```

### Tailwind Config (tailwind.config.js)
```javascript
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {
      colors: {
        primary: '#3B82F6',
        secondary: '#10B981',
        danger: '#EF4444',
      }
    },
  },
  plugins: [],
}
```

## Environment Variables

| Variable | Description | Exemple |
|----------|-------------|---------|
| `VITE_API_BASE_URL` | Backend API URL | `http://localhost:8000/api/v1` |
| `VITE_ENVIRONMENT` | Environment | `development` / `production` |

**Note :** Variables doivent commencer par `VITE_` pour être exposées au client.

## Styling

### Tailwind Classes
```jsx
// Example composant avec Tailwind
const Button = ({ children, variant = 'primary' }) => {
  const baseClasses = "px-4 py-2 rounded font-medium transition"
  const variants = {
    primary: "bg-blue-500 hover:bg-blue-600 text-white",
    secondary: "bg-gray-200 hover:bg-gray-300 text-gray-800",
    danger: "bg-red-500 hover:bg-red-600 text-white"
  }
  
  return (
    <button className={`${baseClasses} ${variants[variant]}`}>
      {children}
    </button>
  )
}
```

## State Management (Zustand)
```javascript
// stores/matchingStore.js
import { create } from 'zustand'

export const useMatchingStore = create((set) => ({
  matches: [],
  loading: false,
  
  fetchMatches: async () => {
    set({ loading: true })
    const response = await matchingService.getMatches()
    set({ matches: response.data, loading: false })
  },
  
  updateMatch: (matchId, updates) => set((state) => ({
    matches: state.matches.map(m => 
      m.id === matchId ? { ...m, ...updates } : m
    )
  }))
}))
```

## API Client
```javascript
// services/api.js
import axios from 'axios'

const api = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json'
  }
})

// Request interceptor (auth token)
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token')
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})

export default api
```

## Testing

### Unit Tests (Vitest)
```bash
# Run tests
npm test

# Watch mode
npm test -- --watch

# Coverage
npm test -- --coverage
```

### E2E Tests (Playwright)
```bash
# Install Playwright
npx playwright install

# Run E2E tests
npm run test:e2e

# Run with UI
npm run test:e2e -- --ui
```

## Troubleshooting

### Port Already in Use
```bash
# Changer port dans vite.config.js
server: {
  port: 5174  // ou autre port disponible
}
```

### API Connection Error
```bash
# Vérifier backend running
curl http://localhost:8000/health

# Vérifier .env
cat .env | grep VITE_API_BASE_URL
```

### Build Errors
```bash
# Clear cache et réinstaller
rm -rf node_modules package-lock.json
npm install
```

## Déploiement

### Build Production
```bash
npm run build
# Output: dist/
```

### Deploy Render (Static Site)
```yaml
# render.yaml
services:
  - type: web
    name: reconciliation-frontend
    env: static
    buildCommand: npm run build
    staticPublishPath: ./dist
```

## Performance

### Bundle Size
```bash
# Analyze bundle
npm run build -- --analyze
```

### Optimizations Applied

- ✅ Code splitting (React.lazy)
- ✅ Tree shaking (Vite)
- ✅ Asset optimization
- ✅ Gzip compression
- ✅ CDN ready