# Source Code

Code source principal de l'application React.

## Structure
```
src/
├── main.jsx                # Entry point React
├── App.jsx                 # Root component
│
├── pages/                  # Pages (routes)
│   ├── UploadPage.jsx
│   ├── MatchingPage.jsx
│   ├── ValidationPage.jsx
│   ├── DashboardPage.jsx
│   ├── ReportsPage.jsx
│   └── HistoryPage.jsx
│
├── components/             # Composants réutilisables
│   ├── common/             # UI génériques
│   ├── upload/             # Upload-specific
│   ├── matching/           # Matching-specific
│   ├── validation/         # Validation-specific
│   └── reports/            # Reports-specific
│
├── services/               # API clients
│   ├── api.js              # Axios instance
│   ├── uploadService.js
│   ├── matchingService.js
│   ├── validationService.js
│   └── reportService.js
│
├── hooks/                  # Custom React hooks
│   ├── useFileUpload.js
│   ├── useMatching.js
│   ├── useValidation.js
│   └── useAuth.js
│
├── store/                  # State management (Zustand)
│   ├── uploadStore.js
│   ├── matchingStore.js
│   ├── validationStore.js
│   └── uiStore.js
│
└── utils/                  # Utilities
    ├── formatters.js       # Date/amount formatting
    ├── validators.js       # Client-side validation
    └── constants.js        # Constants
```

## main.jsx - Entry Point
```javascript
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'
import './index.css'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
)
```

## App.jsx - Root Component
```javascript
import { BrowserRouter, Routes, Route } from 'react-router-dom'
import UploadPage from './pages/UploadPage'
import MatchingPage from './pages/MatchingPage'
import ValidationPage from './pages/ValidationPage'

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<UploadPage />} />
        <Route path="/matching" element={<MatchingPage />} />
        <Route path="/validation" element={<ValidationPage />} />
      </Routes>
    </BrowserRouter>
  )
}

export default App
```

## Pages

### Pages = Routes principales
```javascript
// pages/UploadPage.jsx
import FileUploader from '../components/upload/FileUploader'
import { useFileUpload } from '../hooks/useFileUpload'

export default function UploadPage() {
  const { upload, progress } = useFileUpload()
  
  return (
    <div className="container mx-auto p-6">
      <h1 className="text-2xl font-bold mb-4">Upload Fichiers</h1>
      <FileUploader onUpload={upload} progress={progress} />
    </div>
  )
}
```

## Components

### Component Structure
```javascript
// components/matching/MatchList.jsx
import { useMatchingStore } from '../../store/matchingStore'
import MatchCard from './MatchCard'

export default function MatchList({ filter }) {
  const { matches, loading } = useMatchingStore()
  
  if (loading) return <LoadingSpinner />
  
  const filtered = matches.filter(m => m.confidence === filter)
  
  return (
    <div className="space-y-4">
      {filtered.map(match => (
        <MatchCard key={match.id} match={match} />
      ))}
    </div>
  )
}
```

## Services

### API Client Pattern
```javascript
// services/matchingService.js
import api from './api'

export const matchingService = {
  runMatching: async (bankFileId, ledgerFileId) => {
    const response = await api.post('/matching/run', {
      bank_file_id: bankFileId,
      ledger_file_id: ledgerFileId
    })
    return response.data
  },
  
  getMatches: async (filters) => {
    const response = await api.get('/matches', { params: filters })
    return response.data
  },
  
  getMatchById: async (matchId) => {
    const response = await api.get(`/matches/${matchId}`)
    return response.data
  }
}
```

## Hooks

### Custom Hook Pattern
```javascript
// hooks/useMatching.js
import { useState } from 'react'
import { matchingService } from '../services/matchingService'
import { useMatchingStore } from '../store/matchingStore'

export function useMatching() {
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState(null)
  const { setMatches } = useMatchingStore()
  
  const runMatching = async (bankFileId, ledgerFileId) => {
    setLoading(true)
    setError(null)
    
    try {
      const result = await matchingService.runMatching(bankFileId, ledgerFileId)
      setMatches(result.matches)
      return result
    } catch (err) {
      setError(err.message)
      throw err
    } finally {
      setLoading(false)
    }
  }
  
  return { runMatching, loading, error }
}
```

## Store (Zustand)

### State Management Pattern
```javascript
// store/matchingStore.js
import { create } from 'zustand'

export const useMatchingStore = create((set, get) => ({
  // State
  matches: [],
  selectedMatch: null,
  filters: {
    confidence: 'all',
    status: 'pending'
  },
  
  // Actions
  setMatches: (matches) => set({ matches }),
  
  selectMatch: (matchId) => {
    const match = get().matches.find(m => m.id === matchId)
    set({ selectedMatch: match })
  },
  
  updateFilter: (key, value) => set((state) => ({
    filters: { ...state.filters, [key]: value }
  })),
  
  clearFilters: () => set({
    filters: { confidence: 'all', status: 'pending' }
  })
}))
```

## Utils

### Formatters
```javascript
// utils/formatters.js

export const formatAmount = (amount, currency = 'XOF') => {
  return new Intl.NumberFormat('fr-FR', {
    style: 'currency',
    currency: currency
  }).format(amount)
}

export const formatDate = (date) => {
  return new Date(date).toLocaleDateString('fr-FR', {
    day: '2-digit',
    month: '2-digit',
    year: 'numeric'
  })
}

export const formatConfidence = (level) => {
  const labels = {
    high: 'Haute',
    medium: 'Moyenne',
    low: 'Faible'
  }
  return labels[level] || level
}
```

### Constants
```javascript
// utils/constants.js

export const CONFIDENCE_LEVELS = {
  HIGH: 'high',
  MEDIUM: 'medium',
  LOW: 'low'
}

export const CONFIDENCE_COLORS = {
  high: 'bg-green-100 text-green-800',
  medium: 'bg-yellow-100 text-yellow-800',
  low: 'bg-red-100 text-red-800'
}

export const API_ENDPOINTS = {
  UPLOAD: '/upload',
  MATCHING: '/matching/run',
  MATCHES: '/matches',
  VALIDATION: '/validation/validate'
}
```

## Best Practices

### Component Organization
```javascript
// ✅ Good: Single responsibility
const MatchCard = ({ match }) => {
  return <div>{/* Match card UI */}</div>
}

const MatchList = ({ matches }) => {
  return matches.map(m => <MatchCard key={m.id} match={m} />)
}

// ❌ Bad: Too many responsibilities
const MatchPage = () => {
  return <div>{/* Card + List + Filters + Pagination */}</div>
}
```

### State Management
```javascript
// ✅ Good: Zustand for global state
const { matches } = useMatchingStore()

// ✅ Good: useState for local UI state
const [isOpen, setIsOpen] = useState(false)

// ❌ Bad: Props drilling for global data
<Parent>
  <Child1 matches={matches}>
    <Child2 matches={matches}>
      <Child3 matches={matches} />
    </Child2>
  </Child1>
</Parent>
```

### File Naming

- Components: `PascalCase.jsx` (MatchCard.jsx)
- Hooks: `camelCase.js` (useMatching.js)
- Utils: `camelCase.js` (formatters.js)
- Stores: `camelCase.js` (matchingStore.js)