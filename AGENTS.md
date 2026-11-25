# AGENTS.md

This file tells AI coding agents how to work in this project.
Read it fully before you write or change any code.

If a rule here conflicts with the user's message, follow the user's message.
If you are not sure, ask one short question. Do not guess.

---

## 0. Work mode (read this first)

**Current mode: `FRONTEND_ONLY`**

To change the mode, the user edits the line above. Allowed values: `FRONTEND_ONLY` or `FULL_STACK`.

### In `FRONTEND_ONLY` mode

- Work only in the React frontend. Do not read, edit, or run anything in the Laravel backend, migrations, or database.
- **Do not use the terminal.** Do not run any shell command on your own. This includes `npm`, `composer`, `php artisan`, `git`, `cat`, `ls`, `grep`, `find`, and `rm`.
- Use your built-in file tools to read, search, and edit files. They do not need terminal permission.
- The user runs the dev server, lint, build, and tests. You do not.
- If you think a command is needed, **do not run it**. Write it in your reply and explain why. The user decides.
- The only exception: the user clearly asks you to run a specific command in this message.
- Sections 7 (Backend) and 8.2 (Backend security) are reference only. Ignore them unless the user switches the mode or asks a backend question.

### In `FULL_STACK` mode

- You may run commands from section 3, but only the ones the task needs.
- Ask before any command that changes data (see section 18).

---

## 1. Project overview

- **Name:** SIM Platform
- **What it does:** Makes life easier for students in the Software Industry and Multimedia (SIM) department, Faculty of Science. It puts many student features in one platform.
- **Users:** SIM students
- **Stage:** Production. Real students use it, so be careful with anything that can break existing features.

## 2. Tech stack

| Layer | Technology |
|---|---|
| Frontend | React (JSX), CSS Modules |
| Backend | PHP, Laravel |
| Database | MySQL |
| Cache / Queue | Redis (only if the project already uses it) |
| API style | REST, JSON |
| Version control | Git, GitHub |

Rules:
- Use only what is already in the project.
- Do not add a new library, framework, or service without asking first.
- This project does **not** use TypeScript. Use plain JavaScript with `.jsx` files. Do not add `.tsx` files or type annotations.

## 3. Commands (for the user, and for `FULL_STACK` mode)

In `FRONTEND_ONLY` mode, these are for the user to run. You only suggest them.

```bash
# Frontend
npm install
npm run dev
npm run build
npm run lint
npm test

# Backend
composer install
php artisan serve
php artisan migrate
php artisan test
./vendor/bin/pint
```

After a change, tell the user which command to run to check it. Example: "Run `npm run lint` to check the style."
Never say a task is checked if you did not run the check. Say "not run".

---

## 4. How to work (most important)

### 4.1 Think before you code
1. Read the files related to the task. Do not guess how the code works.
2. For a small fix, just do it.
3. For a bigger task, write a short plan first (3 to 7 steps).
4. Wait for approval if the plan changes the API usage, the shared state, or the folder structure.

### 4.2 Keep changes small
- One task = one focused change.
- Touch only the files the task needs.
- Do not refactor unrelated code.
- Do not rename, move, or delete files unless asked.
- If you find a different problem, **report it**. Do not fix it silently.

### 4.3 Keep it simple
- Choose the simplest solution that is correct and readable.
- Do not add patterns, layers, or abstractions "just in case".
- Three similar lines are better than a wrong abstraction.
- If you think a solution is over-engineered, say so and offer a simpler one.

### 4.4 Do not invent things
- Do not invent API endpoints, response fields, package names, or functions.
- If something does not exist in the project, say that it is missing.
- Do not use deprecated features. When unsure, say so and point to the official documentation.

### 4.5 Ask when needed
Ask a question before you start if:
- The requirements are unclear or can be understood in two ways.
- The task needs a business decision (permissions, rules).
- The task needs a destructive action (delete files or data).

Ask **one** clear question at a time.

---

## 5. Clean code

The goal: code that is **clean, easy to read, short, and not over-engineered**.
Correct first, then clear, then short, then fast.

### 5.1 Readability
- Names explain the purpose: `getUnpaidOrders`, not `getData`. Booleans read like questions: `isActive`, `hasAccess`.
- A function does one thing. If you need "and" to describe it, split it.
- Keep functions short (about 20 lines or less) and files focused.
- Use **early returns**. Maximum 2 levels of `if` or loops.
- Prefer simple, direct code over clever code.
- Follow the style already used in the file you edit.

### 5.2 Less code
- Before you add code, check if the project or the language already does it.
- Use built-in features: `map` / `filter` / `find`, destructuring, optional chaining (`?.`), `??`.
- Remove duplicate logic only when it appears **three or more times**.
- Delete dead code, unused imports, unused variables, and unused props in the code you touch.
- Do not write wrapper functions or extra files that add no value.
- Short must stay readable. No nested ternaries, no one-letter names.

### 5.3 No over-engineering
- Do not build for future needs. Solve today's problem.
- If a simple function works, do not make it a class.

### 5.4 Hygiene
- No commented-out code and no leftover `console.log`.
- No magic numbers or strings. Use named constants.
- Handle errors on purpose. Never use an empty `catch`.
- Handle edge cases: empty data, null values, very long text, slow network, duplicate clicks.
- Comments explain **why**, not **what**.

---

## 6. Frontend rules (React)

### 6.1 Files and structure
- One component per file, with its own CSS file:
  - `ComponentName.jsx`
  - `ComponentName.module.css`
- Component names use `PascalCase`. Hooks start with `use`. Follow the project's existing folder pattern.
- Keep components small. If a component is hard to read, split it.
- Put shared logic in custom hooks. Put API calls in the API layer, not inside JSX.
- Do not repeat the same JSX block. Use a small component or `.map()`.

### 6.2 CSS Modules (strict rule)
Always write class names in this exact format:

```jsx
import styles from "./Card.module.css";

export default function Card({ title, isActive }) {
  return (
    <section className={styles["card"]}>
      <h2 className={styles["card-title"]}>{title}</h2>
    </section>
  );
}
```

For more than one class:

```jsx
<section
  className={`${styles["card"]} ${isActive ? styles["card-active"] : ""}`}
>
```

Rules:
- Always use `styles["class-name"]`. Never `styles.className`.
- Class names in the CSS file use `kebab-case`.
- No inline styles, except for truly dynamic values (for example, a calculated width).
- No global CSS for component styles. Global CSS is only for resets, variables, and fonts.
- No CSS frameworks or UI libraries unless the project already uses one.
- Keep selectors simple (one class). Avoid deep nesting and `!important`.
- Use the project's CSS variables for colors, spacing, font sizes, and radius.
- Remove unused CSS classes in the files you touch.

### 6.3 Building from a design
When the user gives you a design (image, Figma, or description):
- **Match the design.** Do not add your own colors, effects, animations, or sections.
- If a value is missing, ask, or use the closest existing project variable and say which one you chose.
- The layout must work at mobile (360px), tablet (768px), and desktop (1280px).
- Use `rem`, `%`, `flex`, and `grid`. Avoid fixed widths that break small screens.
- Use semantic HTML.

### 6.4 Accessibility
- Use the right element: `button` for actions, `a` for navigation, `nav`, `main`, `section`, `header`, `footer`.
- Every image has a meaningful `alt` (or `alt=""` if it is decoration).
- Every input has a `label`.
- Keyboard users can reach everything. Keep a visible focus style.
- Text must have enough color contrast.
- Do not use `div` or `span` as a button.

### 6.5 State, data, and errors
- Keep state as local as possible.
- Do not store data in state if you can calculate it from other state.
- Every data request handles **loading**, **error**, **success**, and **empty**.
- Never trust data from the server or the user. Validate before using it.
- Use stable, unique `key` values. Do not use the array index if the list can change.
- Clean up effects (timers, listeners, requests) on unmount.
- Do not use `useEffect` for things you can calculate during render or handle in an event handler.
- Do not use `dangerouslySetInnerHTML` unless the content is sanitized and the user approved it.

### 6.6 Performance
- Do not optimize before you measure.
- Do not add `useMemo`, `useCallback`, or `memo` by default.
- Lazy-load big routes and heavy components when it makes sense.

### 6.7 Frontend security
- Never put secrets, keys, or tokens in frontend code.
- Never render untrusted HTML. Escape output.
- Do not store sensitive tokens in `localStorage` if a safer option already exists in the project.
- Do not trust the frontend for permissions. Hiding a button is not security.

---

## 7. Backend rules (Laravel) — reference only in `FRONTEND_ONLY` mode

Do not touch the backend in `FRONTEND_ONLY` mode. If a frontend task needs a backend change, **tell the user what is needed**. Do not do it.

- Controllers are thin. Business logic goes in services or actions.
- Validation goes in Form Requests. Responses go through API Resources.
- Authorization goes in Policies or Gates. Check data ownership (prevent IDOR).
- Every schema change goes through a new migration. Never edit a deployed migration.
- Add foreign keys, indexes, and unique constraints. Avoid N+1 with `with()`. Always paginate lists.
- Use transactions when several writes must succeed or fail together.
- Never run `migrate:fresh` or `db:wipe`, and never delete data without explicit permission.
- API: plural names, `/v1` version, correct status codes, one consistent error format, rate limits on sensitive routes.
- Define `$fillable`. Never call `env()` outside config files.

---

## 8. Security (always check)

### 8.1 Always
- If you see a security problem in existing code, **tell the user clearly**, even if it is not part of the task.
- Never expose, print, or commit secrets. Do not log personal data.

### 8.2 Backend security — only in `FULL_STACK` mode
SQL injection (use bindings), CSRF/CORS (no `*` with credentials), password hashing, mass assignment, file upload validation, safe cookies.

---

## 9. Testing

- Add or update tests for every behavior change and bug fix **when the project already has a test setup**.
- If a test is needed but you cannot run it, write it and tell the user to run it.
- Test **behavior**, not implementation details.
- Test: business rules, validation, error cases, edge cases.
- Do not test: framework internals, simple getters, third-party code.
- Frontend: test what the user sees and does (render, click, type). Do not test internal state.
- Never delete or weaken a test just to make it pass.

---

## 10. Git rules

- **Do not run any `git` command** unless the user asks. This includes `status`, `diff`, and `log`.
- Never commit directly to `main`. Branch names: `feature/short-name`, `fix/short-name`.
- Commit message format: `type: short present-tense message`
  (`feat`, `fix`, `refactor`, `test`, `docs`).
- If the user asks for a commit message, write one. Do not commit.
- Never use `git push --force` or `git reset --hard`.
- Never commit secrets, `node_modules`, `vendor`, build output, or `.env`.

---

## 11. Dependencies

- Prefer the platform and the current stack before adding a package.
- Before you suggest a package, tell the user: what problem it solves, its size, its maintenance status, and the alternative without it.
- **Do not install anything.** Suggest the command. The user decides.
- Do not upgrade major versions unless asked.

---

## 12. Documentation

- Update the README only when you change setup steps, environment variables, or commands.
- New environment variables go into `.env.example` with a safe example value.
- Write docs in simple English with short sentences.

---

## 13. Cleanup mode (refactoring)

Use this mode only when the user asks to clean, simplify, or refactor code.

- Work only on the files or folders the user names. If none are named, ask.
- Do one file or one feature at a time.
- **Behavior must not change.** Same UI, same props, same API calls.
- Do not change routes, API shape, or folder structure without asking.
- List the problems first. Wait for approval if the refactor is big (many files).
- Look for: dead code, deep nesting, long components, duplicate logic (3 or more times), unclear names, over-engineering, and rule breaks from this file (for example `styles.card` instead of `styles["card"]`).
- Do not make code shorter if it becomes harder to read.
- Do not fix unrelated bugs silently. Report them.

---

## 14. Definition of done

A task is done when:

- [ ] The code does what was asked, and only that.
- [ ] The code is clean and easy to read.
- [ ] Edge cases and errors are handled.
- [ ] Security rules are checked.
- [ ] CSS Modules format is correct (`styles["..."]`).
- [ ] No debug code or unused code is left.
- [ ] The report (section 15) is written, including the commands the user should run.

Running lint and tests is **the user's step** in `FRONTEND_ONLY` mode.

---

## 15. How to report your work

Keep it short and in simple English.

1. **What I did:** one or two sentences.
2. **Files changed:** list with one line each.
3. **Why this approach:** the key decision and the main alternative.
4. **How to check it:** what to look at in the browser, and any commands the user should run.
5. **Risks and open points:** anything uncertain or skipped.
6. **Problems I noticed:** issues outside the task (not fixed).

Say clearly what you did not run or check.

---

## 16. Learning mode

The user is a developer who is learning to become a professional engineer.
The goal is to **understand**, not only to get working code.

- Explain the important decision in each change: what, why, and the trade-off. Keep it short.
- Explain technical words the first time you use them.
- Mention a good alternative in one or two lines when there is one.
- If the user's approach has a problem, say it honestly and explain the reason. Do not just praise.
- If the user asks "why" or "how", explain the idea first, then show the code.
- If the user wants to implement a part themselves, give a hint or an outline. Do not give the full solution.
- Prefer small steps the user can read and understand.

---

## 17. Never do this

- Never run terminal commands on your own in `FRONTEND_ONLY` mode.
- Never delete or overwrite files, data, or branches without clear permission.
- Never run destructive commands (`rm -rf`, `migrate:fresh`, `DROP`, `git reset --hard`).
- Never disable security checks, linters, or tests to make something pass.
- Never change files outside the scope of the task.
- Never add a dependency silently.
- Never leave a half-finished change without saying what is missing.
- Never pretend to know something. Say "I'm not sure" and show how to check.