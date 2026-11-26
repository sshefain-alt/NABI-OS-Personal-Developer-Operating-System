# NABI OS — Personal Developer Operating System

"An interactive personal portfolio for **Sadatun Nabi**, built as a desktop operating"
"system in the browser. Explore the profile, projects, education, skills, experience,"
"terminal, contact center, and resume through movable application windows."

"Built with Next.js (App Router), React, TypeScript, Tailwind CSS, Framer Motion, and"
Lucide React.

---

## Quick start

```bash
npm install
npm run dev
```

Open <http://localhost:3000>.

### Scripts

| Command             | Purpose                              |
| ------------------- | ------------------------------------ |
| `npm run dev`       | Start the development server         |
| `npm run build`     | Create a production build            |
| `npm start`         | Serve the production build           |
| `npm run lint`      | Run ESLint                           |
| `npm run typecheck` | Run TypeScript without emitting      |
| `npm run format`    | Format the project with Prettier     |

---

## Personalization checklist

All portfolio content is centralized in `src/data/`. Values wrapped in `[BRACKETS]`
are intentional placeholders — replace them with confirmed information. Nothing is
invented automatically.

"### 1. Profile, email, and social links"

Edit `src/data/profile.ts`:

- `email` — replace `[ADD EMAIL]`
"- `socials` — replace the GitHub, LinkedIn, and website placeholders"
"- `intro`, `interests`, `learning`, `philosophy` — adjust as needed"

The Contact app detects placeholders and disables direct links rather than creating
fake ones. It also falls back to a copyable message when email delivery is not set up.

### 2. Education

Edit `src/data/education.ts`:

- university name
- current status
- expected graduation year
#NAME?
- optional location and highlights

"Do not add a CGPA, award, or certification unless it is confirmed."

### 3. Skills

Edit `src/data/skills.ts`. Add confirmed skill strings to each category's `skills`
array. Categories without entries are displayed as “not configured”. There are no
fabricated proficiency percentages.

### 4. Experience

Edit `src/data/experience.ts`. The array is intentionally empty until real experience
"is supplied. When empty, the app shows the approved fallback message."

### 5. Projects

Edit `src/data/projects.ts`. Each project supports:

"- overview, problem statement, objectives, features, focus areas, contributions"
- status and category
- links (only public URLs should be added)

Links are only rendered as active anchors when they are valid `http(s)` URLs.

### 6. Resume

The resume viewer is honest by default: no fake PDF is generated.

To publish a resume:

1. Add your PDF at `public/resume/Sadatun-Nabi-Resume.pdf`.
"2. In `src/data/profile.ts`, set `resume.available` to `true` and confirm `resume.path`."
3. Rebuild/redeploy.

The viewer performs a `HEAD` request and only enables **Open** and **Download** when a
real PDF is reachable.

---

## Contact form delivery (optional)

The form validates on the client and the server. Real email delivery is optional and
uses [Resend](https://resend.com) through a server-side API route. Secrets are never
exposed to the browser.

1. Copy `.env.example` to `.env.local`.
2. Configure:

```bash
RESEND_API_KEY=re_...
CONTACT_TO_EMAIL=you@example.com
CONTACT_FROM_EMAIL=NABI OS <hello@your-verified-domain.com>
NEXT_PUBLIC_SITE_URL=https://your-domain.com
```

`CONTACT_FROM_EMAIL` must be a sender on a domain verified in Resend. If the provider
"is not configured, `/api/contact` returns `503` and the UI falls back to a copyable"
message (and a mailto action if a real email is configured). It never claims a message
was sent when it wasn't.

The API route also:

- validates every field server-side
- enforces length limits
- uses a honeypot field
- applies best-effort in-memory rate limiting
- escapes user content in the email HTML
- does not log message contents

"For production traffic, use a durable rate limiter (for example, Upstash) if abuse is"
a concern.

---

## Architecture

```text
src/
+-- app/
"¦   +-- layout.tsx              # metadata, fonts, theme bootstrap, OS provider"
¦   +-- page.tsx                # renders the NABI OS shell
"¦   +-- globals.css             # design tokens, light/dark themes, utilities"
¦   +-- icon.svg                # favicon
¦   +-- opengraph-image.tsx     # generated social preview
¦   +-- robots.ts
¦   +-- sitemap.ts
¦   +-- api/contact/route.ts    # optional Resend delivery endpoint
+-- components/
¦   +-- os/
"¦   ¦   +-- OSProvider.tsx      # theme, windows, palette, toasts, responsive mode"
¦   ¦   +-- NabiOS.tsx          # boot -> desktop/mobile shell
¦   ¦   +-- BootScreen.tsx
¦   ¦   +-- Desktop.tsx
¦   ¦   +-- Wallpaper.tsx
¦   ¦   +-- DesktopIcon.tsx
¦   ¦   +-- SystemBar.tsx
¦   ¦   +-- SystemMenu.tsx
¦   ¦   +-- Taskbar.tsx
¦   ¦   +-- StartMenu.tsx
¦   ¦   +-- WindowManager.tsx
¦   ¦   +-- AppWindow.tsx
¦   ¦   +-- CommandPalette.tsx
¦   ¦   +-- MobileDesktop.tsx
¦   ¦   +-- MobileAppPanel.tsx
"¦   +-- applications/           # About, Projects, Education, Skills, Experience,"
"¦   ¦                           # Terminal, Contact, Resume + content registry"
"¦   +-- ui/                     # Button, Badge, Card, EmptyState, Logo, Toaster"
"+-- data/                       # profile, apps, projects, education, skills, experience"
"+-- hooks/                      # window manager, theme, clock, media query"
"+-- lib/                        # constants, utilities, terminal command engine"
+-- types/                      # shared TypeScript interfaces
```

### Window manager

"`useWindowManager` is a `useReducer`-based state machine. It owns window geometry,"
"stacking order, focus, minimize, maximize/restore, close, viewport clamping, and layout"
persistence. A single application can only have one window. Minimized windows stay
"mounted, so terminal history and form state survive minimize/restore."

### Persistence

"- Theme: `localStorage` (`nabi-os:theme`), with system preference as the initial fallback."
"- Boot state: `sessionStorage`, so the startup sequence does not replay on internal"
  interactions.
"- Window geometry: `localStorage` (`nabi-os:layout:v1`), debounced during drags."

### Responsive behavior

- **= 768 px:** full desktop with draggable windows. Windows are clamped to the
  viewport and adapt to tablet dimensions.
"- **< 768 px:** dedicated mobile experience with a branded header, app launcher grid,"
"  full-screen app panels, back navigation, and a touch-friendly bottom dock."

### Terminal

"The terminal is fully simulated. It supports `help`, `about`, `projects`, `skills`,"
"`education`, `experience`, `contact`, `resume`, `open <app>`, `theme`, `neofetch`,"
"`date`, `whoami`, `ls`, `pwd`, `echo`, `history`, `banner`, `clear`, and `exit`."
"It never executes OS commands or arbitrary JavaScript. History navigation uses ?/?,"
"Tab completes commands, and Ctrl/Cmd+L clears."

---

## Accessibility

- Semantic landmarks and labelled dialogs
#NAME?
- Visible focus states
"- `Escape` closes the palette, menus, and mobile panels"
- `prefers-reduced-motion` respected via CSS and Framer Motion’s `MotionConfig`
- Skip-to-desktop link and a `<noscript>` fallback
"- Placeholder and error states are text-based, not colour-only"

## Security and privacy

- No analytics or tracking scripts
- No database
- No secrets in client code
#NAME?
- Security headers configured in `next.config.ts`
- Terminal input is parsed against an allow-list; it is never evaluated

---

## Deployment

The app is Vercel-ready.

1. Push the project to a Git repository.
2. Import it into Vercel.
3. Add the optional environment variables from `.env.example`.
4. Deploy.

"Set `NEXT_PUBLIC_SITE_URL` to the production URL so canonical, Open Graph, sitemap,"
and robots metadata resolve correctly.

---

## Verification

Results are recorded after the final local checks:

- `npm run typecheck` — pass
- `npm run lint` — pass
- `npm run build` — pass

See the final delivery notes for the exact build output and any known limitations.

---

## Known intentional gaps

"These are placeholders by design, not fabricated content:"

- Email and social URLs in `src/data/profile.ts`
"- University, status, and graduation details in `src/data/education.ts`"
- Skill entries (only confirmed entries are displayed)
- Experience entries (empty until real experience is added)
- Resume PDF (viewer shows an honest unavailable state)
- Project links (only rendered when public URLs exist)

Replace these values and the interface updates without component changes.
