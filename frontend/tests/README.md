# Frontend Tests

Tests unitaires et end-to-end pour le frontend.

## Structure
```
tests/
├── unit/                   # Tests unitaires (Vitest)
│   ├── components/
│   ├── hooks/
│   ├── utils/
│   └── services/
│
└── e2e/                    # Tests end-to-end (Playwright)
    ├── upload.spec.js
    ├── matching.spec.js
    └── validation.spec.js
```

## Tests Unitaires (Vitest)

### Setup
```bash
# Installer Vitest + React Testing Library
npm install -D vitest @testing-library/react @testing-library/jest-dom
```

### Configuration (vitest.config.js)
```javascript
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    setupFiles: './tests/setup.js',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html'],
      exclude: ['node_modules/', 'tests/']
    }
  }
})
```

### Lancer Tests
```bash
# Run all tests
npm test

# Watch mode
npm test -- --watch

# Coverage
npm test -- --coverage

# Test specific file
npm test -- tests/unit/components/MatchCard.test.jsx
```

### Example Test: Component
```javascript
// tests/unit/components/MatchCard.test.jsx
import { render, screen } from '@testing-library/react'
import { describe, it, expect } from 'vitest'
import MatchCard from '../../../src/components/matching/MatchCard'

describe('MatchCard', () => {
  const mockMatch = {
    id: '123',
    bank_transaction: {
      description: 'VIR SALAIRE',
      amount: 250000,
      date: '2024-11-15'
    },
    ledger_transaction: {
      description: 'Paiement salaire',
      amount: 250000,
      date: '2024-11-15'
    },
    score: 87.5,
    confidence: 'high'
  }
  
  it('renders match details', () => {
    render(<MatchCard match={mockMatch} />)
    
    expect(screen.getByText('VIR SALAIRE')).toBeInTheDocument()
    expect(screen.getByText('Paiement salaire')).toBeInTheDocument()
    expect(screen.getByText('250,000 XOF')).toBeInTheDocument()
  })
  
  it('displays confidence level', () => {
    render(<MatchCard match={mockMatch} />)
    
    expect(screen.getByText('Haute')).toBeInTheDocument()
  })
  
  it('shows score percentage', () => {
    render(<MatchCard match={mockMatch} />)
    
    expect(screen.getByText('87.5%')).toBeInTheDocument()
  })
})
```

### Example Test: Hook
```javascript
// tests/unit/hooks/useMatching.test.js
import { renderHook, waitFor } from '@testing-library/react'
import { describe, it, expect, vi } from 'vitest'
import { useMatching } from '../../../src/hooks/useMatching'
import * as matchingService from '../../../src/services/matchingService'

// Mock service
vi.mock('../../../src/services/matchingService')

describe('useMatching', () => {
  it('runs matching successfully', async () => {
    const mockResult = {
      total_matched: 100,
      high_confidence: 85
    }
    
    matchingService.runMatching.mockResolvedValue(mockResult)
    
    const { result } = renderHook(() => useMatching())
    
    await result.current.runMatching('bank-123', 'ledger-456')
    
    await waitFor(() => {
      expect(result.current.loading).toBe(false)
      expect(result.current.error).toBeNull()
    })
  })
  
  it('handles error', async () => {
    matchingService.runMatching.mockRejectedValue(
      new Error('Network error')
    )
    
    const { result } = renderHook(() => useMatching())
    
    await expect(
      result.current.runMatching('bank-123', 'ledger-456')
    ).rejects.toThrow('Network error')
  })
})
```

### Example Test: Utils
```javascript
// tests/unit/utils/formatters.test.js
import { describe, it, expect } from 'vitest'
import { formatAmount, formatDate } from '../../../src/utils/formatters'

describe('formatters', () => {
  describe('formatAmount', () => {
    it('formats XOF correctly', () => {
      expect(formatAmount(250000, 'XOF')).toBe('250 000 FCFA')
    })
    
    it('formats EUR correctly', () => {
      expect(formatAmount(1500.50, 'EUR')).toBe('1 500,50 €')
    })
  })
  
  describe('formatDate', () => {
    it('formats date to french format', () => {
      expect(formatDate('2024-11-15')).toBe('15/11/2024')
    })
  })
})
```

## Tests E2E (Playwright)

### Setup
```bash
# Installer Playwright
npm install -D @playwright/test

# Install browsers
npx playwright install
```

### Configuration (playwright.config.js)
```javascript
import { defineConfig } from '@playwright/test'

export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  use: {
    baseURL: 'http://localhost:5173',
    trace: 'on-first-retry',
  },
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:5173',
    reuseExistingServer: !process.env.CI,
  },
})
```

### Lancer Tests E2E
```bash
# Run all E2E tests
npm run test:e2e

# Run with UI
npm run test:e2e -- --ui

# Run specific test
npm run test:e2e -- tests/e2e/matching.spec.js

# Debug mode
npm run test:e2e -- --debug
```

### Example E2E Test
```javascript
// tests/e2e/matching.spec.js
import { test, expect } from '@playwright/test'

test.describe('Matching Flow', () => {
  test('complete matching workflow', async ({ page }) => {
    // 1. Navigate to upload page
    await page.goto('/')
    
    // 2. Upload bank statement
    await page.getByText('Upload Bank Statement').click()
    await page.setInputFiles(
      'input[type="file"]',
      './tests/fixtures/bank_statement.csv'
    )
    await expect(page.getByText('Upload successful')).toBeVisible()
    
    // 3. Upload ledger
    await page.getByText('Upload Ledger').click()
    await page.setInputFiles(
      'input[type="file"]',
      './tests/fixtures/ledger.csv'
    )
    await expect(page.getByText('Upload successful')).toBeVisible()
    
    // 4. Navigate to matching page
    await page.goto('/matching')
    
    // 5. Run matching
    await page.getByRole('button', { name: 'Run Matching' }).click()
    
    // 6. Wait for results
    await expect(page.getByText('Matching completed')).toBeVisible({
      timeout: 30000
    })
    
    // 7. Verify results displayed
    await expect(page.getByText('Total Matched:')).toBeVisible()
    await expect(page.getByText('High Confidence:')).toBeVisible()
    
    // 8. Check matches list
    const matchCards = page.getByTestId('match-card')
    await expect(matchCards).toHaveCount(10, { timeout: 5000 })
  })
})
```

## Coverage Target

**Minimum :** 70%

### Par type
- Components: 75%+
- Hooks: 85%+
- Utils: 90%+
- Services: 70%+

## Best Practices

### ✅ DO
- Test comportement utilisateur, pas implémentation
- Utiliser data-testid pour éléments critiques
- Mock appels API externes
- Tests rapides (<100ms unitaire)

### ❌ DON'T
- Tester détails implémentation
- Tester librairies tierces
- Tests dépendants de l'ordre
- Snapshots excessifs

## CI/CD

Tests lancés automatiquement sur :
- Push vers `develop` ou `main`
- Pull Request
```yaml
# .github/workflows/frontend-ci.yml
- name: Run unit tests
  run: npm test -- --coverage

- name: Run E2E tests
  run: npm run test:e2e
```

## Debug

### Unit Tests
```bash
# Run en mode debug
npm test -- --inspect-brk

# Puis ouvrir chrome://inspect
```

### E2E Tests
```bash
# Mode debug avec UI
npm run test:e2e -- --debug

# Headed mode (voir browser)
npm run test:e2e -- --headed
```