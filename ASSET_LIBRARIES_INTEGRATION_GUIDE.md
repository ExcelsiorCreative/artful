# Free Asset Libraries Integration Guide for Artful

A comprehensive guide to integrating multiple free asset library APIs into your browser-based design software.

---

## 📦 Currently Integrated

### ✅ Unsplash (Photos)
- **Status**: Fully integrated
- **Files**: 
  - `backend/src/routes/unsplash.ts`
  - `frontend/src/lib/unsplash-api.ts`
  - `frontend/src/components/editor-images-panel.tsx`
- **API**: Free tier available (50 requests/hour for applications)
- **Setup**: Add `UNSPLASH_ACCESS_KEY` to `.env`

---

## 🎯 Easy-to-Add Libraries (Production-Ready)

### 1. **Pexels API** (Stock Photos)
**Cost**: Free (no API key required for basic use)  
**Quality**: High  
**License**: Free to use (CC0)

#### Backend Integration
```typescript
// backend/src/routes/pexels.ts
import { Elysia, t } from 'elysia'

export const pexelsRoutes = new Elysia({ prefix: '/pexels' })
  .get('/search', async ({ query }) => {
    const q = query.q?.trim() ?? ''
    const page = Math.max(1, Number(query.page) || 1)
    const perPage = Math.min(80, Math.max(1, Number(query.per_page) || 20))
    
    const url = new URL('https://api.pexels.com/v1/search')
    url.searchParams.set('query', q || 'nature')
    url.searchParams.set('page', String(page))
    url.searchParams.set('per_page', String(perPage))
    
    const res = await fetch(url, {
      headers: { 'Authorization': process.env.PEXELS_API_KEY || '' }
    })
    
    if (!res.ok) throw new Error(`Pexels: ${res.status}`)
    const body = await res.json()
    return {
      data: {
        photos: body.photos || [],
        hasMore: body.next_page ? true : false
      }
    }
  }, {
    query: t.Object({
      q: t.Optional(t.String()),
      page: t.Optional(t.Numeric()),
      per_page: t.Optional(t.Numeric()),
    })
  })
```

#### Frontend Integration
```typescript
// frontend/src/lib/pexels-api.ts
import { getPublicApiBase } from './public-api-base'

export type PexelsPhoto = {
  id: number
  width: number
  height: number
  src: {
    original: string
    large: string
    medium: string
    small: string
  }
  photographer: string
  photographer_url: string
}

export async function fetchPexelsSearch(
  query: string,
  page: number,
  perPage = 20,
) {
  const q = query.trim()
  const url = `${getPublicApiBase()}/pexels/search?q=${encodeURIComponent(q)}&page=${page}&per_page=${perPage}`
  const res = await fetch(url, { credentials: 'include' })
  if (!res.ok) return { photos: [], hasMore: false, error: 'Pexels error' }
  const body = await res.json()
  return { photos: body.data.photos, hasMore: body.data.hasMore }
}
```

**Setup**:
```bash
# Get free API key from https://www.pexels.com/api/
echo "PEXELS_API_KEY=your-key-here" >> backend/.env
```

---

### 2. **Pixabay API** (Photos, Vectors, Videos)
**Cost**: Free (no auth required, 5000 requests/hour/IP)  
**Quality**: High  
**License**: Pixabay License (free for commercial use)

#### Implementation Pattern
```typescript
// Same structure as Pexels
// Endpoint: GET /pixabay/search?q=query&page=1
// No auth needed, but add key for higher limits
const PIXABAY_API_KEY = process.env.PIXABAY_API_KEY || ''
const url = `https://pixabay.com/api/?q=${q}&page=${page}&key=${PIXABAY_API_KEY}`
```

**Setup**:
```bash
# Free key from https://pixabay.com/api/docs/
echo "PIXABAY_API_KEY=your-key-here" >> backend/.env
```

---

### 3. **Undraw** (Illustrations - No API, Direct CDN)
**Cost**: Free  
**Quality**: Beautiful custom illustrations  
**License**: Free to use

#### Implementation (No Backend Needed)
```typescript
// frontend/src/lib/undraw-api.ts
const UNDRAW_BASE = 'https://undraw.co/api/'

export type UndrawIllustration = {
  id: string
  title: string
  image: string
  categories: string[]
}

export async function searchUndrawIllustrations(
  query: string,
): Promise<UndrawIllustration[]> {
  try {
    const res = await fetch(`${UNDRAW_BASE}illustrations?search=${encodeURIComponent(query)}`)
    if (!res.ok) return []
    const data = await res.json()
    return data.illustrations || []
  } catch {
    return []
  }
}

export function getUndrawImageUrl(id: string, color?: string): string {
  let url = `https://undraw.co/illustrations/${id}`
  if (color) url += `?color=${encodeURIComponent(color)}`
  return url
}
```

---

### 4. **OpenDoodles** (Hand-Drawn Illustrations)
**Cost**: Free  
**Quality**: Playful, hand-drawn style  
**License**: CC0 (public domain)

#### Implementation (Static CDN)
```typescript
// frontend/src/lib/opendoodles-api.ts
export type OpenDoodlesImage = {
  id: string
  title: string
  url: string
  category: string
}

const DOODLES = [
  { id: 'blank-canvas', title: 'Blank Canvas', category: 'start' },
  { id: 'sitting', title: 'Sitting Person', category: 'people' },
  { id: 'standing', title: 'Standing Person', category: 'people' },
  // ... more from https://www.opendoodles.com
]

export function getOpenDoodlesImageUrl(id: string): string {
  return `https://cdn.shopify.com/s/files/1/0553/2427/2475/files/${id}.png`
}

export function searchOpenDoodles(query: string): OpenDoodlesImage[] {
  return DOODLES.filter(d => 
    d.title.toLowerCase().includes(query.toLowerCase()) ||
    d.category.includes(query.toLowerCase())
  )
}
```

---

### 5. **Freepik Free Elements** (Limited - Vectors, Photos)
**Cost**: Freemium (free tier + paid)  
**Quality**: Mixed  
**License**: Freepik License (requires attribution)

#### Implementation
```typescript
// Freepik API is deprecated, but you can:
// 1. Use their paid API for premium features
// 2. Embed search widget
// 3. Fetch from their free search pages (web scraping - use sparingly)
```

**Note**: Freepik requires paid API access for reliable integration.

---

## 🚀 Implementation Roadmap

### Phase 1: Add Pexels (Easiest, High Impact)
**Time**: 2-3 hours
```bash
# 1. Copy pexels.ts to backend/src/routes/
# 2. Add route to backend/src/index.ts: `.use(pexelsRoutes)`
# 3. Create pexels-api.ts in frontend/src/lib/
# 4. Add Pexels tab to EditorImagesPanel
# 5. Test and deploy
```

### Phase 2: Add Undraw + OpenDoodles
**Time**: 2 hours (no backend needed)
```bash
# 1. Create frontend-only APIs (no server changes)
# 2. Add "Illustrations" tab to EditorImagesPanel
# 3. Support color customization for Undraw
```

### Phase 3: Add Pixabay
**Time**: 1 hour (very similar to Pexels)

### Phase 4: Advanced Features
- Vector shape libraries (SVG collections)
- Icon packs beyond HugeIcons
- Template marketplace integration

---

## 📋 Complete File Setup Checklist

### Backend Files to Add/Modify
- [ ] `backend/src/routes/pexels.ts` (NEW)
- [ ] `backend/src/routes/pixabay.ts` (NEW)
- [ ] `backend/src/index.ts` (MODIFY - add new routes)
- [ ] `backend/.env.example` (MODIFY - add API keys)

### Frontend Files to Add/Modify
- [ ] `frontend/src/lib/pexels-api.ts` (NEW)
- [ ] `frontend/src/lib/pixabay-api.ts` (NEW)
- [ ] `frontend/src/lib/undraw-api.ts` (NEW)
- [ ] `frontend/src/lib/opendoodles-api.ts` (NEW)
- [ ] `frontend/src/components/editor-images-panel.tsx` (MODIFY - add tabs)
- [ ] `frontend/src/lib/avnac-ai-tambo-tools.ts` (MODIFY - add AI tools)

---

## 🔑 API Keys Required

### Required
- **Unsplash**: https://unsplash.com/oauth/applications (Already set up)
- **Pexels**: https://www.pexels.com/api/ (Free)
- **Pixabay**: https://pixabay.com/api/docs/ (Free)

### Optional (Enhanced)
- **Freepik Pro**: https://www.freepik.com/api (Paid)
- **Tambo AI**: Enhanced art direction prompts

### Not Required
- Undraw (uses public API)
- OpenDoodles (static CDN)

---

## 💡 Code Examples

### Adding a Library to EditorImagesPanel

```tsx
// frontend/src/components/editor-images-panel.tsx
import { PexelsPhoto, fetchPexelsSearch } from '../lib/pexels-api'
import { UndrawIllustration, searchUndrawIllustrations } from '../lib/undraw-api'

type Tab = 'unsplash' | 'pexels' | 'illustrations' | 'doodles'

export default function EditorImagesPanel({ open, onClose }: Props) {
  const [tab, setTab] = useState<Tab>('unsplash')
  const [photos, setPhotos] = useState<UnsplashPhoto[] | PexelsPhoto[]>([])
  const [illustrations, setIllustrations] = useState<UndrawIllustration[]>([])

  useEffect(() => {
    if (!open) return
    
    if (tab === 'pexels' && committed.q !== '') {
      fetchPexelsSearch(committed.q, committed.page).then(res => {
        setPhotos(res.photos)
      })
    }
    
    if (tab === 'illustrations') {
      searchUndrawIllustrations(committed.q).then(setIllustrations)
    }
  }, [tab, open, committed])

  return (
    <div>
      <div className="flex gap-2 border-b p-2">
        <button onClick={() => setTab('unsplash')} 
          className={tab === 'unsplash' ? 'active' : ''}>
          Unsplash
        </button>
        <button onClick={() => setTab('pexels')} 
          className={tab === 'pexels' ? 'active' : ''}>
          Pexels
        </button>
        <button onClick={() => setTab('illustrations')} 
          className={tab === 'illustrations' ? 'active' : ''}>
          Illustrations
        </button>
      </div>
      
      {/* Render based on tab */}
      {tab === 'unsplash' && <UnsplashGrid photos={photos} />}
      {tab === 'pexels' && <PexelsGrid photos={photos} />}
      {tab === 'illustrations' && <UndrawGrid illustrations={illustrations} />}
    </div>
  )
}
```

---

## 🎨 Advanced: AI Tool Integration

```typescript
// frontend/src/lib/avnac-ai-tambo-tools.ts (additions)
const searchAllAssetLibraries: TamboTool = {
  name: 'search_asset_libraries',
  description:
    'Search across all available free libraries: Unsplash, Pexels, Pixabay, Undraw illustrations.',
  tool: async (args: { query: string; source?: 'unsplash' | 'pexels' | 'pixabay' | 'undraw' }) => {
    const results = {}
    
    if (!args.source || args.source === 'unsplash') {
      results.unsplash = await fetchUnsplashSearch(args.query, 1)
    }
    if (!args.source || args.source === 'pexels') {
      results.pexels = await fetchPexelsSearch(args.query, 1)
    }
    if (!args.source || args.source === 'undraw') {
      results.undraw = await searchUndrawIllustrations(args.query)
    }
    
    return { ok: true, results }
  },
  // ... schema
}
```

---

## 📊 Comparison Table

| Library | Type | Quality | License | API Free Tier | Speed | Setup |
|---------|------|---------|---------|---------------|-------|-------|
| **Unsplash** | Photos | ★★★★★ | CC0 | 50 req/hr | Fast | Already done |
| **Pexels** | Photos | ★★★★★ | CC0 | No limit | Fast | Easy |
| **Pixabay** | Photos/Vectors | ★★★★ | Free | 5000 req/hr/IP | Fast | Easy |
| **Undraw** | Illustrations | ★★★★★ | CC0 | Unlimited | Very Fast | 1 hour |
| **OpenDoodles** | Illustrations | ★★★★ | CC0 | Unlimited | Instant | 30 min |
| **Freepik** | Mixed | ★★★★ | Freepik | Limited | Fast | Paid API |

---

## 🚨 Best Practices

### 1. **Caching**
```typescript
// Cache search results for 1 hour
const cacheKey = `${provider}:${query}:${page}`
const cached = cache.get(cacheKey)
if (cached && Date.now() - cached.time < 3600000) {
  return cached.data
}
```

### 2. **Rate Limiting**
```typescript
// Track requests per user/IP
const requests = new Map<string, number[]>()
const userKey = getUserIp(req)
const now = Date.now()
const recent = requests.get(userKey) || []
const cleaned = recent.filter(t => now - t < 60000)

if (cleaned.length > API_LIMIT) {
  throw new HttpError(429, 'Rate limit exceeded')
}
```

### 3. **Attribution**
```typescript
// Always show photographer/artist credit
const attribution = `Photo by ${photo.photographer} on Pexels`
// Store in metadata or UI
```

### 4. **Error Handling**
```typescript
// Graceful degradation
export async function searchAllSources(query) {
  const results = {}
  try {
    results.unsplash = await fetchUnsplash(query)
  } catch (e) {
    results.unsplash = { error: e.message }
  }
  // Continue with other sources even if one fails
}
```

---

## 🔗 Quick Links

- [Pexels API Docs](https://www.pexels.com/api/)
- [Pixabay API Docs](https://pixabay.com/api/docs/)
- [Undraw Collections](https://undraw.co/illustrations)
- [OpenDoodles Library](https://www.opendoodles.com/)
- [Freepik API](https://www.freepik.com/api/)
- [Unsplash API](https://unsplash.com/developers) (already integrated)

---

## 📝 Next Steps

1. **Start with Pexels** - highest ROI, easiest implementation
2. **Add Undraw** - complements photos with illustrations
3. **Integrate Pixabay** - adds vector support
4. **Consider Freepik** if you need more advanced features
5. **Build unified search UI** across all providers

---

## Questions?

See `frontend/src/components/editor-images-panel.tsx` for the current implementation pattern.
