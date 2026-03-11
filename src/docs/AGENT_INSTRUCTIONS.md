# GEM Fortress Shield — Agent Instructions

> **Purpose**: This document gives a ChatGPT (or any LLM) agent full context to understand, maintain, and extend this codebase.

---

## 1. Project Overview

**GEM Fortress Shield** is a production-ready **cybersecurity monitoring & services platform** built with:

| Layer | Technology |
|-------|-----------|
| Framework | React 18 + TypeScript |
| Build | Vite |
| Styling | Tailwind CSS + shadcn/ui components |
| Backend | Supabase (Lovable Cloud) — Postgres, Auth, Edge Functions, Storage |
| Routing | React Router v6 |
| State | TanStack React Query |
| Icons | Lucide React |

**Live URL**: `https://gem-fortress-shield.lovable.app`

---

## 2. Directory Structure

```
src/
├── App.tsx                  # Root router — all routes defined here
├── main.tsx                 # Entry point
├── index.css                # Design tokens (HSL), Tailwind layers, animations
├── components/
│   ├── layout/              # Navbar, Footer (shared across public pages)
│   ├── sections/            # Landing page sections (Hero, Services, About, etc.)
│   ├── cyber/               # Cybersecurity-specific UI (ServiceCard, ThreatShield, etc.)
│   ├── portal/              # Authenticated portal components (Sidebar, StatWidget, etc.)
│   ├── effects/             # Visual effects (NetworkAnimation)
│   └── ui/                  # shadcn/ui primitives (Button, Card, Input, etc.)
├── pages/
│   ├── Index.tsx            # Landing page
│   ├── Services.tsx         # Services listing
│   ├── SentinelX.tsx        # SentinelX product page
│   ├── TrustCenter.tsx      # Trust/compliance center
│   ├── RealEstate.tsx       # Real estate vertical
│   ├── Contact.tsx          # Contact form
│   ├── Auth.tsx             # Login / Sign-up
│   ├── portal/              # Authenticated dashboard pages
│   │   ├── Dashboard.tsx
│   │   ├── TaskBoard.tsx
│   │   ├── Incidents.tsx
│   │   ├── Team.tsx
│   │   ├── Activity.tsx
│   │   └── Settings.tsx
│   └── products/            # Product detail pages
│       ├── BioVault.tsx
│       ├── NeuralNet.tsx
│       └── QuantumGuard.tsx
├── hooks/
│   ├── useAuth.tsx          # Auth context (signUp, signIn, signOut, user, session)
│   ├── use-mobile.tsx       # Mobile breakpoint hook
│   └── use-toast.ts         # Toast notifications
├── integrations/supabase/
│   ├── client.ts            # ⚠️ AUTO-GENERATED — never edit
│   └── types.ts             # ⚠️ AUTO-GENERATED — never edit
└── lib/utils.ts             # cn() utility for Tailwind class merging
```

---

## 3. Database Schema (Supabase / Postgres)

| Table | Purpose | Key Columns |
|-------|---------|-------------|
| `profiles` | User profiles | `user_id`, `full_name`, `job_title`, `department`, `status` |
| `projects` | Security projects | `name`, `owner_id`, `priority`, `risk_score`, `status` |
| `tasks` | Task board items | `title`, `assigned_to`, `project_id`, `priority`, `status`, `due_date` |
| `incidents` | Security incidents | `title`, `severity`, `status`, `type`, `assigned_to`, `affected_systems[]` |
| `threats` | Threat intelligence | `name`, `type`, `severity`, `source_ip`, `target_system`, `status` |
| `activities` | Audit log | `action`, `entity_type`, `entity_id`, `user_id`, `metadata` |
| `comments` | Entity comments | `content`, `entity_type`, `entity_id`, `user_id` |
| `user_roles` | RBAC roles | `user_id`, `role` (enum: `admin`, `manager`, `analyst`, `viewer`) |

### Role-Based Access
- Roles stored in `user_roles` table (never on `profiles`)
- `has_role(_user_id, _role)` — security-definer function for RLS policies
- RLS is enabled on all tables

---

## 4. Authentication

- **Provider**: Supabase Auth (email + password)
- **Hook**: `useAuth()` from `src/hooks/useAuth.tsx`
  - Exposes: `user`, `session`, `loading`, `signUp`, `signIn`, `signOut`
- **Auth page**: `/auth` — handles both login and registration
- **Portal routes** (`/portal/*`) require authentication

---

## 5. Design System

### Theme Tokens (defined in `src/index.css`)
All colors are **HSL values** used via CSS variables:

| Token | Purpose |
|-------|---------|
| `--background` | Deep navy (`220 30% 8%`) |
| `--foreground` | Light text (`210 40% 98%`) |
| `--primary` | Blue accent (`217 91% 60%`) |
| `--warning` | Amber (`38 92% 50%`) |
| `--success` | Green (`142 76% 36%`) |
| `--destructive` | Red (`0 72% 51%`) |
| `--gem-glow` | Cyber glow effect |

### Rules
- **Never use raw color classes** (`text-white`, `bg-black`) — always use semantic tokens (`text-foreground`, `bg-background`)
- Fonts: **Inter** (body) + **JetBrains Mono** (code/cyber elements)
- Button variants: `default`, `hero`, `hero-outline`, `cyber`, `cyber-ghost`, `nav`

---

## 6. Routing

All routes are in `src/App.tsx`:

| Path | Page | Auth Required |
|------|------|---------------|
| `/` | Landing | No |
| `/services` | Services | No |
| `/sentinel-x` | SentinelX | No |
| `/trust-center` | Trust Center | No |
| `/real-estate` | Real Estate | No |
| `/contact` | Contact | No |
| `/auth` | Login/Register | No |
| `/products/bio-vault` | BioVault | No |
| `/products/neural-net` | NeuralNet | No |
| `/products/quantum-guard` | QuantumGuard | No |
| `/portal` | Dashboard | Yes |
| `/portal/tasks` | Task Board | Yes |
| `/portal/incidents` | Incidents | Yes |
| `/portal/team` | Team | Yes |
| `/portal/activity` | Activity | Yes |
| `/portal/settings` | Settings | Yes |

---

## 7. Key Patterns

### Adding a new page
1. Create component in `src/pages/`
2. Add route in `src/App.tsx` (above the `*` catch-all)
3. Add nav link in `Navbar.tsx` and/or `PortalSidebar.tsx`

### Adding a new database table
1. Create SQL migration via Supabase migration tool
2. Enable RLS and create appropriate policies
3. Types auto-generate in `src/integrations/supabase/types.ts`

### Supabase queries
```typescript
import { supabase } from "@/integrations/supabase/client";

// Read
const { data, error } = await supabase.from('incidents').select('*');

// Insert
const { error } = await supabase.from('incidents').insert({ title: 'New incident', severity: 'high' });

// With React Query
const { data } = useQuery({
  queryKey: ['incidents'],
  queryFn: async () => {
    const { data, error } = await supabase.from('incidents').select('*');
    if (error) throw error;
    return data;
  }
});
```

### Edge Functions
- Located in `supabase/functions/`
- Auto-deployed on push
- Access secrets via `Deno.env.get("SECRET_NAME")`

---

## 8. Environment Variables

| Variable | Source | Usage |
|----------|--------|-------|
| `VITE_SUPABASE_URL` | Auto-configured | Supabase project URL |
| `VITE_SUPABASE_PUBLISHABLE_KEY` | Auto-configured | Anon key for client |
| `VITE_SUPABASE_PROJECT_ID` | Auto-configured | Project identifier |
| `LOVABLE_API_KEY` | Auto-configured (server only) | AI gateway auth |

**Never edit `.env` manually** — it's auto-generated.

---

## 9. Do NOT Modify

These files are auto-generated and must never be edited:
- `src/integrations/supabase/client.ts`
- `src/integrations/supabase/types.ts`
- `supabase/config.toml`
- `.env`
- `supabase/migrations/` (read-only history)

---

## 10. Common Tasks for Agents

| Task | How |
|------|-----|
| Fix a UI bug | Read the component, check `index.css` tokens, edit component |
| Add a feature to portal | Create component in `src/components/portal/`, add page in `src/pages/portal/`, add route |
| Query the database | Use `supabase` client with React Query, check types.ts for schema |
| Add an API integration | Create edge function in `supabase/functions/`, call via `supabase.functions.invoke()` |
| Change theme colors | Edit HSL values in `src/index.css` `:root` block |
| Add authentication guard | Check `useAuth()` hook, redirect if `!user` |

---

## 11. Testing Checklist

Before considering a change complete:
- [ ] No TypeScript errors
- [ ] Responsive on mobile (430px) and desktop
- [ ] Uses semantic design tokens (no raw colors)
- [ ] RLS policies if touching database
- [ ] Auth check if portal route
