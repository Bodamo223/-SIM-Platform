# AGENTS.md

This file tells AI coding agents how to work in this project.
Read it fully before you write or change any code.

If a rule here conflicts with the user's message, follow the user's message.
If you are not sure, ask one short question. Do not guess.

---

## 1. Project overview

- **Name:** SIM Platform
- **What it does:** Makes life easier for students in the Software Industry and Multimedia (SIM) department, Faculty of Science. It puts many student features in one platform.
- **Users:** SIM students
- **Stage:** Production. Real students use it, so be careful with data, migrations, and anything that can break existing features.

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
- This project does **not** use TypeScript. Use plain JavaScript with `.jsx` files. Do not add TypeScript, `.tsx` files, or type annotations.

## 3. Commands

> Replace these with the real commands of the project.

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
./vendor/bin/pint        # code style
```

Before you say a task is done, run the lint and the tests that apply to your change.
If you cannot run them, say so clearly.

---

## 4. How to work (most important)

### 4.1 Think before you code
1. Read the files related to the task. Do not guess how the code works.
2. For any task bigger than a small fix, write a short plan first (3 to 7 steps).
3. Wait for approval if the plan changes the database, the API, or the folder structure.

### 4.2 Keep changes small
- One task = one focused change.
- Touch only the files the task needs.
- Do not refactor unrelated code.
- Do not rename, move, or delete files unless asked.
- If you find a different problem, **report it**. Do not fix it silently.

### 4.3 Keep it simple
- Choose the simplest solution that is correct and readable.
- Do not add patterns, layers, or abstractions "just in case".
- Use a design pattern only when it solves a real problem in this code.
- Three similar lines are better than a wrong abstraction.
- If you think a solution is over-engineered, say so and offer a simpler one.

### 4.4 Do not invent things
- Do not invent API endpoints, database columns, package names, or functions.
- If something does not exist in the project, say that it is missing.
- Do not use deprecated features. When unsure, check the official documentation.

### 4.5 Ask when needed
Ask a question before you start if:
- The requirements are unclear or can be understood in two ways.
- The task needs a business decision (prices, permissions, rules).
- The task needs a destructive action (delete data, reset database, force push).

Ask **one** clear question at a time.

---

## 5. Code quality (all languages)

- **Correct first, then clear, then fast.**
- Names must explain the purpose: `getUnpaidOrders`, not `getData`.
- A function does one thing. If you need "and" to describe it, split it.
- Keep functions short and files focused.
- No dead code, no commented-out code, no leftover `console.log`, `dd()`, or `dump()`.
- No magic numbers or strings. Use named constants or config.
- Handle errors on purpose. Never swallow an error with an empty `catch`.
- Handle edge cases: empty data, null values, very long text, slow network, duplicate requests.
- Comments explain **why**, not **what**. Good code explains the what.
- Follow the style already used in the file you edit.

---

## 6. Frontend rules (React)

### 6.1 Files and structure
- One component per file. Each component has its own folder or its own pair of files:
  - `ComponentName.jsx`
  - `ComponentName.module.css`
- Component names use `PascalCase`. Hooks start with `use`. Files follow the project's existing pattern.
- Keep components small. If a component is hard to read, split it.
- Put shared logic in custom hooks. Put API calls in a separate API layer, not inside JSX.

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
- Always use `styles["class-name"]` with brackets and quotes. Never `styles.className`.
- Class names in the CSS file use `kebab-case`.
- No inline styles, except for values that are truly dynamic (for example, a calculated width).
- No global CSS for component styles. Global CSS is only for resets, variables, and fonts.
- No CSS frameworks or UI libraries unless the project already uses one.
- Keep selectors simple (one class). Avoid deep nesting and `!important`.
- Use CSS variables for colors, spacing, font sizes, and radius. Do not repeat raw values.

### 6.3 Building from a design
When the user gives you a design (image, Figma, or description):
- **Match the design.** Do not add your own colors, effects, animations, or extra sections.
- If a value is missing (a color, a size, a spacing), ask or use the closest existing project variable. Say which one you chose.
- Check the layout at mobile (360px), tablet (768px), and desktop (1280px).
- Use `rem`, `%`, `flex`, and `grid`. Avoid fixed widths that break small screens.
- Use real, semantic HTML.

### 6.4 Accessibility (not optional)
- Use the right element: `button` for actions, `a` for navigation, `nav`, `main`, `section`, `header`, `footer`.
- Every image has a meaningful `alt` (or `alt=""` if it is only decoration).
- Every input has a `label`.
- Keyboard users can reach and use everything. Keep a visible focus style.
- Text must have enough color contrast.
- Do not use `div` or `span` as a button.

### 6.5 State, data, and errors
- Keep state as local as possible. Do not add global state unless many distant components need it.
- Do not store data in state if you can calculate it from other state.
- Every data request has three states: **loading**, **error**, **success**. Also handle **empty**.
- Never trust data from the server or from the user. Validate before using it.
- Use stable, unique `key` values in lists. Never use the array index if the list can change.
- Clean up effects (timers, listeners, requests) when the component unmounts.
- Do not use `dangerouslySetInnerHTML` unless the content is sanitized and the user approved it.

### 6.6 Performance
- Do not optimize before you measure.
- Do not add `useMemo`, `useCallback`, or `memo` by default. Add them only for a real, measured problem.
- Avoid heavy work inside render.
- Lazy-load big routes and heavy components when it makes sense.

---

## 7. Backend rules (Laravel)

### 7.1 Structure
Request flow:

```
Route -> Middleware -> Controller -> Service/Action -> Model -> Database
                          |
                      Form Request (validation)
                          |
                      API Resource (response)
```

- **Controllers are thin.** They receive the request, call a service, and return a response. No business logic and no raw queries in controllers.
- **Business logic** goes in service or action classes.
- **Validation** goes in Form Request classes, not inside controllers.
- **Responses** go through API Resources, so the API shape is controlled.
- **Authorization** goes in Policies or Gates. Never trust only the frontend.
- Use Eloquent relationships, scopes, and casts instead of repeating logic.
- Use Jobs and Queues for slow work (emails, reports, external APIs).
- Use config files and `.env` for settings. Never call `env()` outside config files.

### 7.2 Database and queries
- Every schema change goes through a **migration**. Never edit the database by hand.
- Never edit an old migration that was already shared or deployed. Create a new one.
- Use correct column types and lengths.
- Add **foreign keys** and **indexes** for columns used in `WHERE`, `JOIN`, and `ORDER BY`.
- Use **unique constraints** in the database, not only in PHP validation.
- Avoid the **N+1 problem**: use `with()` to eager load relationships.
- Select only the columns you need on big tables.
- Always **paginate** list endpoints. Never return unlimited rows.
- Use **transactions** when several writes must succeed or fail together.
- Money: store as integer minor units (for example, cents) or `DECIMAL`. Never `FLOAT`.
- Use soft deletes only when the business really needs history.
- Never run `migrate:fresh`, `db:wipe`, or delete data without explicit permission.

### 7.3 API design
- Use clear, plural resource names: `GET /api/v1/orders`, `POST /api/v1/orders`.
- Use the correct HTTP method and status code:
  `200` OK, `201` created, `204` no content, `400` bad request, `401` not authenticated,
  `403` not allowed, `404` not found, `422` validation error, `429` too many requests, `500` server error.
- Use one consistent error format, for example:

```json
{
  "message": "The given data was invalid.",
  "errors": { "email": ["The email field is required."] }
}
```

- Version the API (`/v1`). Do not break existing clients without a new version.
- `GET` must never change data. Make `POST` or `PUT` safe against double submit when needed (idempotency).
- Add rate limiting to login, registration, password reset, and other sensitive routes.

---

## 8. Security (always check)

Check these on every change that touches input, data, or auth:

- **SQL injection:** use Eloquent or query bindings. Never put user input inside a raw SQL string.
- **XSS:** never render untrusted HTML. Escape output.
- **CSRF / CORS:** keep protection enabled. Allow only the origins that are needed. Never use `*` with credentials.
- **Authentication:** hash passwords with Laravel's `Hash` (bcrypt or argon2). Never store or log plain passwords.
- **Authorization:** check **who owns the data** on every read and write (prevent IDOR). Example: a user must not read another user's order by changing the ID.
- **Mass assignment:** define `$fillable`. Never use `$guarded = []` without a strong reason.
- **File uploads:** validate type, size, and extension. Store outside the public folder when possible. Rename files. Never trust the original file name.
- **Secrets:** never put keys, passwords, or tokens in code, commits, logs, or frontend code. Use `.env`. Do not commit `.env`.
- **Logs and errors:** never log passwords, tokens, or personal data. Do not show stack traces to users.
- **Dependencies:** do not add packages that are unmaintained or unknown.
- **Tokens and sessions:** use `HttpOnly`, `Secure`, and `SameSite` cookies when possible. Set expiry times.

If you see a security problem in existing code, **tell the user clearly**, even if it is not part of the task.

---

## 9. Testing

- Add or update tests for every behavior change and every bug fix.
- For a bug: first write a test that fails, then fix it.
- Test **behavior**, not implementation details.
- What to test:
  - Business rules and calculations.
  - Validation and error cases.
  - Authorization (who can and cannot do what).
  - Edge cases (empty, null, limits, duplicates).
- What not to test: framework internals, simple getters, third-party code.
- Backend: use Laravel feature tests for endpoints and unit tests for pure logic. Use a separate test database and factories.
- Frontend: test what the user sees and does (render, click, type). Do not test internal state.
- Mock only external services (payment, email, third-party APIs). Do not mock your own database in feature tests.
- Tests must be fast, independent, and repeatable.
- Never delete or weaken a test just to make it pass. If a test is wrong, explain why.

---

## 10. Git rules

- Never commit directly to `main`. Work on a branch: `feature/short-name`, `fix/short-name`.
- Do not commit, push, or open pull requests unless the user asks.
- Commit messages: short, clear, present tense, with a type:
  - `feat: add order cancellation endpoint`
  - `fix: stop duplicate payment on double click`
  - `refactor: extract price calculation to service`
  - `test: add cases for expired coupons`
  - `docs: update setup steps`
- One commit = one logical change.
- Never use `git push --force`, `git reset --hard`, or delete branches without permission.
- Never commit secrets, `node_modules`, `vendor`, build output, or `.env`.

---

## 11. Performance and reliability

- Measure before you optimize. Do not guess bottlenecks.
- Check database queries first (N+1, missing indexes, big result sets).
- Cache only when needed. If you add cache, explain how and when it is cleared.
- Set timeouts and handle failures for every external request (API, payment, email).
- Slow work goes to a queue. Jobs must be safe to run twice (idempotent) and must retry safely.
- Log important events and errors with enough context to debug (ids, action), but no secrets.

---

## 12. Dependencies

- Prefer the platform and the current stack before adding a package.
- Before you add a package, tell the user: what problem it solves, its size, its maintenance status, and the alternative without it.
- Do not upgrade major versions unless asked.

---

## 13. Documentation

- Update the README or docs when you change setup steps, environment variables, commands, or API behavior.
- New environment variables go into `.env.example` with a safe example value.
- Write docs in simple English with short sentences.

---

## 14. Definition of done

A task is done only when all of these are true:

- [ ] The code does what was asked, and only that.
- [ ] Edge cases and errors are handled.
- [ ] Security rules in section 8 are checked.
- [ ] Tests are added or updated and pass.
- [ ] Lint and formatting pass.
- [ ] No debug code or unused code is left.
- [ ] Docs and `.env.example` are updated if needed.
- [ ] The summary (section 15) is written.

---

## 15. How to report your work

When you finish, reply in this format. Keep it short and in simple English.

1. **What I did:** one or two sentences.
2. **Files changed:** list of files with one line each.
3. **Why this approach:** the key decision and the main alternative.
4. **How to test it:** exact steps or commands.
5. **Risks and open points:** anything uncertain, skipped, or worth a second look.
6. **Problems I noticed:** issues outside the task (not fixed).

If you did not run something, say so. Never claim that something works if you did not check it.

---

## 16. Learning mode

The user is a developer who is learning to become a professional engineer.
The goal is to **understand**, not only to get working code.

- Explain the important decision in each change: what, why, and the trade-off. Keep it short.
- Explain technical words the first time you use them.
- When there is a good alternative, mention it in one or two lines.
- If the user's approach has a problem, say it honestly and explain the reason. Do not just praise.
- If the user asks "why" or "how", explain the idea first, then show the code.
- If the user says they want to implement a part themselves, give a hint or a small outline. Do not give the full solution.
- Do not write huge amounts of code the user cannot review. Prefer small steps the user can read and understand.

---

## 17. Never do this

- Never delete or overwrite data, files, or branches without clear permission.
- Never run destructive commands (`rm -rf`, `migrate:fresh`, `DROP`, `git reset --hard`) on your own.
- Never expose, print, or commit secrets.
- Never disable security checks, linters, or tests to make something pass.
- Never change files outside the scope of the task.
- Never add a dependency silently.
- Never leave a half-finished change without saying what is missing.
- Never pretend to know something. Say "I'm not sure" and show how to check.
