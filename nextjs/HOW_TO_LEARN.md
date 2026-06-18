# How to Learn This Curriculum

## The Philosophy

This curriculum is built around **doing, not reading**. Each assignment has you write real code that ships. Each project builds something you can show in a portfolio. By Phase 9, you will have built 22 projects and completed 36 focused assignments.

**Do not skip assignments to reach projects faster.** Assignments build the precise knowledge that projects assume. If a project is confusing, the answer is always in the preceding assignments.

---

## How to Work Through an Assignment

1. **Read the Background section fully** before touching code.
2. **Set up the environment** described in the Tasks section.
3. **Type the code yourself** — do not paste. Typing builds muscle memory.
4. **Run the code after each step.** Don't batch multiple tasks before verifying.
5. **When something breaks**, read the error message out loud. Look up the exact error message, not a paraphrase of it.
6. **Check the Deliverables** before marking complete. Every checkbox must pass.

---

## How to Work Through a Project

1. Read the entire project file before writing a single line.
2. Build the Architecture first — create all empty files in the directory tree.
3. Fill in stubs top-down: config → layout → page → components → utilities.
4. Commit after each logical section (e.g., "add auth", "add dashboard page").
5. Deploy the project (even Phase 1 projects should go on Vercel).
6. Document one thing you learned in a comment or a short README.

---

## Environment Setup

### Required tools
```bash
# Node.js 20+ (use nvm)
nvm install 20
nvm use 20
node --version   # v20.x.x

# pnpm (faster than npm for Next.js projects)
npm install -g pnpm
pnpm --version

# Git
git --version
```

### Create a new Next.js project (used in every phase)
```bash
pnpm create next-app@latest my-project \
  --typescript \
  --tailwind \
  --eslint \
  --app \
  --src-dir \
  --import-alias "@/*"

cd my-project
pnpm dev
```

### Recommended VS Code extensions
- **ESLint** — dbaeumer.vscode-eslint
- **Prettier** — esbenp.prettier-vscode
- **Tailwind CSS IntelliSense** — bradlc.vscode-tailwindcss
- **TypeScript** — ms-vscode.vscode-typescript-next
- **Prisma** — prisma.prisma (needed from Phase 5 onward)

### Recommended VS Code settings
```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit"
  },
  "typescript.tsdk": "node_modules/typescript/lib",
  "tailwindCSS.experimental.classRegex": [
    ["cva\\(([^)]*)\\)", "[\"'`]([^\"'`]*).*?[\"'`]"],
    ["cx\\(([^)]*)\\)", "(?:'|\"|`)([^']*)(?:'|\"|`)"]
  ]
}
```

---

## Folder Structure Convention

Every project in this curriculum lives in a dedicated folder. Use this naming scheme:

```
~/projects/nextjs-curriculum/
  phase1/
    p1-portfolio/
    p1-docs-site/
  phase2/
    p2-blog/
    p2-dashboard/
  ...
```

Each project is its own Git repository pushed to GitHub. This builds a portfolio of 22 real projects.

---

## Debugging Next.js — Common Issues

### "Error: Async/await is not yet supported in Client Components"
You used `async` on a component that has `"use client"`. Remove `async` and use `useEffect` + `useState`, or move the fetch to a Server Component parent.

### "'window' is not defined"
You're accessing browser APIs in a Server Component. Either add `"use client"` or gate with `if (typeof window !== "undefined")`.

### Hydration mismatch
Server HTML differs from client render. Common causes: `Date.now()`, `Math.random()`, or conditional rendering based on browser state. Use `suppressHydrationWarning` for intentional mismatches (e.g., theme).

### Route not found after adding a file
Restart `pnpm dev`. The file watcher sometimes misses new files.

### "Cannot read properties of undefined (reading 'params')"
In Next.js 14, page props are `{ params: { id: string }, searchParams: { ... } }`. Destructure correctly: `export default function Page({ params }: { params: { id: string } })`.

---

## How to Measure Progress

After every **assignment**, run this mental checklist:
- [ ] Can I explain what this does without looking at notes?
- [ ] Can I reproduce the core pattern from memory?
- [ ] Did I hit an error I didn't understand? Did I research it until I did?

After every **project**, run this checklist:
- [ ] Is the project deployed (Vercel or Docker)?
- [ ] Does it have a README with a screenshot?
- [ ] Is the GitHub repo public?
- [ ] Can I explain every file in the project?

---

## Study Schedule Templates

### Full-time (8 hrs/day)
| Week | Phases | Focus |
|------|--------|-------|
| 1 | Phase 1 | App Router mastery |
| 2 | Phase 2–3 | Data fetching + Styling |
| 3 | Phase 4 | State management |
| 4 | Phase 5 | Auth |
| 5 | Phase 6–7 | Performance + Testing |
| 6 | Phase 8–9 | Deployment + Advanced |

### Part-time (2 hrs/day, weekdays only)
| Weeks | Phases |
|-------|--------|
| 1–3 | Phase 1 |
| 4–6 | Phase 2–3 |
| 7–9 | Phase 4 |
| 10–11 | Phase 5 |
| 12–13 | Phase 6 |
| 14–16 | Phase 7 |
| 17–18 | Phase 8 |
| 19–22 | Phase 9 |

---

## Resources by Phase

### Phase 1–2
- [Next.js official docs](https://nextjs.org/docs) — read the App Router section fully
- [React Server Components RFC](https://github.com/reactjs/rfcs/pull/188)

### Phase 3
- [Tailwind CSS docs](https://tailwindcss.com/docs)
- [CVA docs](https://cva.style/docs)
- [Framer Motion docs](https://www.framer.com/motion/)

### Phase 4
- [TanStack Query docs](https://tanstack.com/query/latest)
- [Zustand docs](https://zustand-demo.pmnd.rs/)
- [React Hook Form docs](https://react-hook-form.com/)

### Phase 5
- [Auth.js (NextAuth v5) docs](https://authjs.dev/)

### Phase 6
- [web.dev Core Web Vitals](https://web.dev/vitals/)
- [PageSpeed Insights](https://pagespeed.web.dev/)

### Phase 7
- [Testing Library docs](https://testing-library.com/docs/)
- [Playwright docs](https://playwright.dev/docs/intro)

### Phase 8
- [Vercel docs](https://vercel.com/docs)
- [GitHub Actions docs](https://docs.github.com/en/actions)

### Phase 9
- [next-intl docs](https://next-intl-docs.vercel.app/)
- [next-pwa docs](https://github.com/shadowwalker/next-pwa)
