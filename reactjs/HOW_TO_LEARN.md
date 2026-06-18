# HOW_TO_LEARN.md — Study Strategy for This Curriculum

---

## The Core Loop

Every concept in this curriculum follows the same loop:

```
Read the assignment → Type the code by hand → Break it on purpose → Fix it → Build the project
```

Do not skip steps. The "break it on purpose" step is where most of the learning happens.

---

## Daily Schedule (Recommended)

### Weekday (2 hours)
| Time | Activity |
|------|----------|
| 0:00–0:20 | Review yesterday's code — read it, not rewrite it |
| 0:20–1:10 | Work through one assignment section |
| 1:10–1:45 | Experiment: change values, add console.logs, introduce a bug |
| 1:45–2:00 | Write 3 sentences in a learning journal about what surprised you |

### Weekend (4 hours each day)
| Time | Activity |
|------|----------|
| Day 1 | Build the project from scratch using the assignment knowledge |
| Day 2 | Polish, add features not in the spec, write at least one test |

---

## How to Work Through an Assignment

### Step 1 — Skim First (5 min)
Read the entire assignment file before touching your editor. Understand the goal and the deliverables.

### Step 2 — Set Up the Environment
Each assignment assumes you have a Vite + React + TypeScript project. Create one fresh per assignment during Phase 1-2, then reuse a project for later phases.

```bash
pnpm create vite my-assignment --template react-ts
cd my-assignment
pnpm install
pnpm dev
```

### Step 3 — Type Every Code Snippet
Copy-pasting defeats the purpose. When you type code, your brain processes it differently. Typos are fine — fixing them is part of learning.

### Step 4 — Run After Each Section
Do not write 100 lines and then run. Write 10-20 lines, save, check the browser, check the console.

### Step 5 — Check the Deliverables
Every assignment has checkboxes. Do not proceed until every box is ticked from memory, not from reading the file.

---

## How to Work Through a Project

### Do Not Look at the Implementation File While Building
The project files (`P*.md`) give you the spec, architecture, and code stubs. **Build first, then compare.** The implementation is there to unblock you when stuck for more than 20 minutes — not as a template.

### Minimum Viable Project First
Get something working end-to-end before adding features. A working ugly thing beats a broken beautiful thing.

### Phases of a Project Build
1. Scaffold: create the directory, install dependencies, get `pnpm dev` running
2. Static shell: render all components with fake/hardcoded data
3. Wire up state: replace fake data with useState/useReducer
4. Connect to real data: add API calls, React Query, localStorage
5. Polish: loading states, error states, empty states, responsive layout
6. Stretch: add one feature not in the spec

---

## Environment Setup

### Required Tools
```bash
# Node.js 20+
node --version

# pnpm (preferred package manager for this curriculum)
npm install -g pnpm
pnpm --version

# VS Code extensions
# - ESLint
# - Prettier
# - Tailwind CSS IntelliSense
# - TypeScript and JavaScript Language Features (built-in)
```

### VS Code Settings
Add to `.vscode/settings.json` in each project:
```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit"
  },
  "typescript.preferences.importModuleSpecifier": "relative"
}
```

### Recommended `.prettierrc`
```json
{
  "semi": false,
  "singleQuote": true,
  "trailingComma": "es5",
  "printWidth": 100,
  "tabWidth": 2
}
```

---

## Common Mistakes to Avoid

### Mistake 1 — Treating React like jQuery
React is **declarative**. You describe what the UI should look like given the current state. You do not imperatively manipulate the DOM. If you find yourself reaching for `document.getElementById`, stop and ask "how do I express this with state?"

### Mistake 2 — Mutating State Directly
```tsx
// WRONG
const [items, setItems] = useState([1, 2, 3])
items.push(4)  // mutating — React won't re-render
setItems(items)

// CORRECT
setItems(prev => [...prev, 4])
```

### Mistake 3 — Missing Keys in Lists
```tsx
// WRONG — using index as key when list can reorder
{items.map((item, i) => <Item key={i} {...item} />)}

// CORRECT — use stable unique id
{items.map(item => <Item key={item.id} {...item} />)}
```

### Mistake 4 — useEffect with Missing Dependencies
Run `eslint-plugin-react-hooks`. Never suppress its warnings without understanding them.

### Mistake 5 — Async Functions Directly in useEffect
```tsx
// WRONG
useEffect(async () => { ... }, [])

// CORRECT
useEffect(() => {
  async function load() { ... }
  load()
}, [])
```

### Mistake 6 — Putting Server State in useState
Once you reach Phase 4, all API data goes through React Query. Stop doing this:
```tsx
const [data, setData] = useState(null)
useEffect(() => { fetch(...).then(r => r.json()).then(setData) }, [])
```

---

## Debugging Strategy

### React DevTools
Install the React DevTools browser extension. Use it to:
- Inspect component props and state
- See which components re-render (enable "Highlight updates")
- Profile rendering performance

### When Something Doesn't Work
1. Check the browser console for errors
2. Add `console.log` before and after the suspect code
3. Check React DevTools — is state what you expect?
4. Check Network tab — did the API call fire? What was the response?
5. Simplify: comment out everything except the broken part
6. Search the exact error message

### When You're Stuck for More Than 20 Minutes
Look at the project implementation file for hints. If still stuck after 10 more minutes, search online using the exact error message. If still stuck, move forward and come back.

---

## Tracking Progress

Keep a file `PROGRESS.md` at the root of your learnings folder:

```markdown
## Phase 1
- [x] A1.1 — 2025-01-05
- [x] A1.2 — 2025-01-06
- [ ] A1.3
- [ ] A1.4
- [ ] P1.1
- [ ] P1.2
```

---

## How to Know You've Learned Something

You have genuinely learned a concept when you can:
1. Explain it in plain English to someone who doesn't code
2. Write the code from memory without looking at examples
3. Debug a broken version of it
4. Know when NOT to use it

Apply this test to every deliverable before checking it off.

---

## Resources

### Official Docs (Primary)
- [react.dev](https://react.dev) — the official React docs, rewritten in 2023
- [TanStack Query docs](https://tanstack.com/query/latest)
- [React Router docs](https://reactrouter.com)
- [React Hook Form docs](https://react-hook-form.com)
- [Zod docs](https://zod.dev)
- [Tailwind CSS docs](https://tailwindcss.com)

### For Going Deeper
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/)
- [web.dev accessibility](https://web.dev/accessibility)
- [Testing Library docs](https://testing-library.com)
- [Playwright docs](https://playwright.dev)

### Tools
- [Vite](https://vitejs.dev)
- [shadcn/ui](https://ui.shadcn.com)
- [Framer Motion](https://www.framer.com/motion/)

---

## A Note on AI Assistants

Use AI tools (ChatGPT, Claude, Copilot) to:
- Explain an error message you don't understand
- Ask "what's wrong with this code?" after you've tried debugging
- Ask conceptual questions ("when should I use useReducer vs useState?")

Do not use AI tools to:
- Write your assignments for you
- Generate project code before you've attempted it

The goal of this curriculum is to build your own mental model. If an AI writes the code, you don't build the model — you just copy it.
