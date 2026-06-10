# 📋 Expenso

**Sprint Duration:** Monday 06/08/2026 – Friday 06/12/2026 | **Deadline:** Friday 06/12 – 2:30 PM

---

## 🐳 Docker Setup — Empty GitHub Repo (Start Here)

> Follow these steps **before anything else** on an empty cloned repo.

### Step 1 — Clone your repo and enter it

```bash
git clone https://github.com/<your-username>/expense-assistant.git
cd expense-assistant
```

### Step 2 — Create Laravel project in a temp folder, then move files

```bash
# Create Laravel app in a temp folder (outside the repo)
composer create-project laravel/laravel expense-assistant-temp

# Copy everything into your cloned repo
cp -r expense-assistant-temp/. .

# Remove the temp folder
rm -rf expense-assistant-temp
```

### Step 3 — Install Laravel Sail

```bash
composer require laravel/sail --dev
php artisan sail:install
# When prompted, select: mysql
```

### Step 4 — Files to create / configure

**`.env`** — Edit the generated `.env`:

```yaml
APP_NAME=ExpenseAssistant
APP_URL=http://localhost

DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=expense_assistant
DB_USERNAME=sail
DB_PASSWORD=password

# Queue configuration
QUEUE_CONNECTION=database

# AI SDK configuration
AI_API_KEY=your_groq_api_key_here
```

**`.gitignore`** — Verify these lines exist (add if missing):

```yaml
/vendor/
/node_modules/
.env
.env.backup
/storage/*.key
/public/hot
/public/storage
```

**`compose.yaml`** — Created automatically by Sail. Verify it contains these services:

```yaml
services:
    laravel.test:
        build:
            context: ./vendor/laravel/sail/runtimes/8.3
            dockerfile: Dockerfile
        ports:
            - '${APP_PORT:-80}:80'
        environment:
            - DB_HOST=mysql
        depends_on:
            - mysql
    mysql:
        image: 'mysql/mysql-server:8.0'
        ports:
            - '${FORWARD_DB_PORT:-3306}:3306'
        environment:
            MYSQL_ROOT_PASSWORD: '${DB_PASSWORD}'
            MYSQL_DATABASE: '${DB_DATABASE}'
            MYSQL_USER: '${DB_USERNAME}'
            MYSQL_PASSWORD: '${DB_PASSWORD}'
        volumes:
            - 'sail-mysql:/var/lib/mysql'
    phpmyadmin:
        image: phpmyadmin/phpmyadmin
        ports:
            - '8081:80'
        environment:
            PMA_HOST: mysql
        depends_on:
            - mysql
volumes:
    sail-mysql:
        driver: local
```

> ⚠️ **phpMyAdmin is not added by Sail automatically** — add it manually to `compose.yaml` as shown above.

### Step 5 — Start Docker and verify

```bash
./vendor/bin/sail up -d

# Add alias for convenience (add to ~/.bashrc or ~/.zshrc)
alias sail='./vendor/bin/sail'

# Verify the app is running
# → http://localhost should show the Laravel welcome page
```

### Step 6 — Generate app key

```bash
sail artisan key:generate
```

### Step 7 — Configure Pest for TDD

```bash
sail composer require pestphp/pest pestphp/pest-plugin-laravel --dev
sail artisan pest:install

# Create a dedicated test database
# Add to .env:
# DB_TEST_DATABASE=expense_assistant_test

# phpunit.xml — verify the test database environment:
# <env name="DB_DATABASE" value="expense_assistant_test"/>
# <env name="APP_ENV" value="testing"/>
```

### Step 8 — Initial commit

```bash
git add .
git commit -m "chore: initial Laravel + Sail setup with MySQL, phpMyAdmin and Pest"
git push origin main
```

### Step 9 — Create feature branches

```bash
git checkout -b feature/recus-crud
git push origin feature/recus-crud

git checkout main
git checkout -b feature/extraction-ia
git push origin feature/extraction-ia

git checkout main
git checkout -b feature/queue-processing
git push origin feature/queue-processing
```

---

## 📋 Legend

| Label | Meaning |
|-------|---------|
| `ARCH` | Architecture / Setup |
| `DOCKER` | Docker / Infrastructure |
| `AUTH` | Authentication |
| `RECU` | Receipt Management |
| `EXTRACT` | AI Extraction (Structured Output) |
| `QUEUE` | Jobs & Queues (Asynchronous) |
| `CAST` | Eloquent Casts & Serialization |
| `SPEC` | Spec-Driven Development (OpenSpec) |
| `TEST` | Pest Tests (TDD) |
| `QA` | Code Quality / Security |
| `DEBUG` | Debugging Tools |
| `DOC` | Documentation / Deliverables |
| `BONUS` | Bonus Feature |

---

## 🎓 AUTOFORMATION BLOC 1 — Concepts before LAB 1

**Objective:** Master foundational concepts needed for CRUD Receipt management  
**Duration:** Monday 06/08 morning (before coding Sprint 1)

| Done | # | Task | Label | Priority | Time | Learning Resources & Checklist |
| :---: | :--- | :--- | :---: | :---: | :---: | :--- |
| [ ] | AF-01 | Eloquent Relationships fundamentals | `ARCH` | High | 1.5h | **Read & understand:**<br>- Doc: <https://laravel.com/docs/13.x/eloquent-relationships> 📖<br>- Focus: `hasMany`, `belongsTo` conceptually<br>**Checklist:**<br>- [ ] Understand what a foreign key is<br>- [ ] Understand `hasMany` vs `belongsTo` semantics<br>- [ ] Write pseudocode for `Recu hasMany Depense` relationship<br>**Example:** A Receipt has many Expenses; an Expense belongs to a Receipt. |
| [ ] | AF-02 | Form Request validation patterns | `ARCH` | High | 1h | **Read & understand:**<br>- Doc: <https://laravel.com/docs/13.x/validation#form-request-validation> 📖<br>- Why NOT use `$request->validate()` in controllers<br>**Checklist:**<br>- [ ] Know the 2 methods: `authorize()` and `rules()`<br>- [ ] Understand validation rule syntax<br>- [ ] Understand why Form Requests keep controllers clean<br>**Example:** `StoreRecuRequest` with rules for receipt text (required, min 10, max 5000). |
| [ ] | AF-03 | Laravel migrations & model generation | `ARCH` | High | 0.5h | **Read & understand:**<br>- Doc: <https://laravel.com/docs/13.x/migrations> 📖<br>**Checklist:**<br>- [ ] Know how `php artisan make:migration` works<br>- [ ] Understand table schema builder (`$table->string()`, `$table->enum()`, foreign keys)<br>- [ ] Know the difference between migration and model<br>**Example:** Create `recus` table with `statut` enum column. |
| [ ] | AF-04 | What is JSON Schema & why it matters | `ARCH` | High | 0.5h | **Read & understand:**<br>- Quick intro: <https://json-schema.org/learn/getting-started-step-by-step> 📖<br>**Checklist:**<br>- [ ] Understand the JSON contract concept<br>- [ ] Recognize the project's contract (articles array with libellé, quantité, etc.)<br>- [ ] Know why validation against a schema prevents garbage data<br>**Example:** The JSON payload from Groq must match a defined schema. |
| [ ] | AF-05 | Database design: MCD (Conceptual) basics | `ARCH` | High | 0.5h | **Read & understand:**<br>- Quick guide: entities, attributes, relationships, cardinality<br>- Focus: draw 3 entities (User, Recu, Depense) and their relationships<br>**Checklist:**<br>- [ ] Draw MCD on paper or draw.io: entities + relationships + cardinalities (1:N)<br>- [ ] Identify primary keys (id)<br>- [ ] Identify foreign keys conceptually (user_id, recu_id)<br>**Example:** User (1) --- (N) Recu --- (N) Depense |

**AUTOFORMATION BLOC 1 — Definition of Done:**

- [ ] Can explain what `Recu hasMany Depense` means in English
- [ ] Can write a Form Request rule for "receipt text must be 10–5000 chars"
- [ ] Understand why validation happens before the controller logic
- [ ] Can sketch an MCD by hand or in draw.io (3 entities)
- [ ] Recognize JSON schema as a "contract" the AI output must follow

---

## 🏃 Sprint 1 — Infrastructure, Setup & First Tests

**Objective:** Docker up, Laravel initialized, Pest configured, migrations ready, debugging tools installed, Tailwind working  
**Duration:** Monday 06/08 — Lundi après-midi + Tuesday 06/09 matin

| Done | # | Task | Label | Priority | Time | Detailed Implementation & Files |
| :---: | :--- | :--- | :---: | :---: | :---: | :--- |
| [ ] | T-01 | Initialize GitHub repo + branches | `ARCH` | High | 0.3h | **Action:**<br>- Create branches: `feature/recus-crud`, `feature/extraction-ia`, `feature/queue-processing`<br>- `.gitignore`: ignore `vendor/`, `.env`, `node_modules/`, `storage/*.key`<br>- `README.md`: push initial skeleton<br>- Jira board created and shared with mentor before 16:00 |
| [ ] | T-02 | Install Laravel via Sail | `DOCKER` | High | 1h | **Action:**<br>- Follow the Docker Setup section above<br>- `composer create-project laravel/laravel expense-assistant-temp`<br>- Copy files into cloned repo<br>- `composer require laravel/sail --dev`<br>- `php artisan sail:install` → choose **mysql**<br>- Add phpMyAdmin service manually to `compose.yaml` |
| [ ] | T-03 | Start Docker + verify environment | `DOCKER` | High | 0.5h | **Action:**<br>- `./vendor/bin/sail up -d`<br>- Verify `http://localhost` → Laravel welcome page<br>- Verify `http://localhost:8081` → phpMyAdmin login<br>- Add alias: `alias sail='./vendor/bin/sail'` |
| [ ] | T-04 | Configure `.env` | `ARCH` | High | 0.3h | **Files to Edit:**<br>- `.env`: `APP_NAME=ExpenseAssistant`, `DB_HOST=mysql`, `DB_DATABASE=expense_assistant`, `DB_USERNAME=sail`, `DB_PASSWORD=password`<br>- `APP_URL=http://localhost`<br>- `QUEUE_CONNECTION=database` (we'll use database queue driver initially)<br>- Run `sail artisan key:generate` if not done |
| [ ] | T-05 | Install Tailwind CSS (v4) | `ARCH` | High | 0.5h | **Action:**<br>- `sail npm install`<br>- `sail npm install -D @tailwindcss/vite`<br>- Edit `vite.config.js` → add `@tailwindcss/vite` plugin<br>- `resources/css/app.css` → add `@import 'tailwindcss';`<br>- `sail npm run dev` → verify styles load on welcome page |
| [ ] | T-06 | Install + configure Pest | `TEST` | High | 0.5h | **Action:**<br>- `sail composer require pestphp/pest pestphp/pest-plugin-laravel --dev`<br>- `sail artisan pest:install`<br>- `phpunit.xml`: set `DB_DATABASE=expense_assistant_test`, `APP_ENV=testing`<br>- `sail artisan test` → must pass the default welcome test ✅<br>- Add `uses(RefreshDatabase::class)` in `tests/Pest.php` for Feature tests |
| [ ] | T-07 | Migration — Table `users` (default) | `ARCH` | High | 0.2h | **Action:**<br>- Default Laravel `users` migration is already present<br>- Verify columns: `id`, `name`, `email`, `email_verified_at`, `password`, `remember_token`, `timestamps`<br>- No changes needed — Breeze will handle it |
| [ ] | T-08 | Migration — Table `recus` | `ARCH` | High | 1h | **Files to Create:**<br>- `sail artisan make:migration create_recus_table`<br>- **Columns:** `id` (PK), `user_id` (FK → users.id, onDelete cascade), `texte_brut` (longText — Darija text), `statut` (enum: `['en_attente', 'traité', 'échoué']`, default: `en_attente`), `payload_ia` (json, nullable — raw AI response), `deleted_at` (timestamp, nullable — for SoftDeletes), `timestamps`<br>- Use `$table->softDeletes();` — do NOT add `deleted_at` manually<br>- Use `$table->enum('statut', ['en_attente', 'traité', 'échoué'])->default('en_attente')` |
| [ ] | T-09 | Migration — Table `depenses` | `ARCH` | High | 1h | **Files to Create:**<br>- `sail artisan make:migration create_depenses_table`<br>- **Columns:** `id` (PK), `recu_id` (FK → recus.id, onDelete cascade), `libellé` (string, 255), `quantité` (integer), `prix_unitaire` (decimal: 10,2), `catégorie` (enum: `['alimentaire', 'boissons', 'hygiène', 'entretien', 'autre']`), `timestamps`<br>- Use `$table->enum('catégorie', [...])->default('autre')` |
| [ ] | T-10 | Model `Recu` + SoftDeletes + relationships | `ARCH` | High | 1h | **Files to Create:**<br>- `sail artisan make:model Recu`<br>- Add `use SoftDeletes;` trait (import `Illuminate\Database\Eloquent\SoftDeletes`)<br>- `$fillable = ['user_id', 'texte_brut', 'statut', 'payload_ia']`<br>- Relationships:<br>&nbsp;&nbsp;- `user(): BelongsTo` → `User::class`<br>&nbsp;&nbsp;- `depenses(): HasMany` → `Depense::class`<br>- **Accessor `getStatutLabelAttribute()`:**<br>&nbsp;&nbsp;`return match($this->statut) { 'en_attente' => 'En attente', 'traité' => 'Traité', 'échoué' => 'Échoué' };`<br>- **Cast `statut` as enum** (Illuminate\Database\Eloquent\Casts\AsEnumCollection if using native enums, or string if using enum values) |
| [ ] | T-11 | Model `Depense` + relationships | `ARCH` | High | 0.5h | **Files to Create:**<br>- `sail artisan make:model Depense`<br>- `$fillable = ['recu_id', 'libellé', 'quantité', 'prix_unitaire', 'catégorie']`<br>- Relationships:<br>&nbsp;&nbsp;- `recu(): BelongsTo` → `Recu::class`<br>- **Accessor `getCategorieLabelAttribute()`:**<br>&nbsp;&nbsp;`return match($this->catégorie) { 'alimentaire' => 'Alimentaire', 'boissons' => 'Boissons', 'hygiène' => 'Hygiène', 'entretien' => 'Entretien', 'autre' => 'Autre' };`<br>- **Cast `catégorie` as enum** |
| [ ] | T-12 | Model `User` + relationships | `ARCH` | High | 0.3h | **Files to Edit:**<br>- `app/Models/User.php`<br>- Add: `recus(): HasMany` → `Recu::class`<br>- Verify `$fillable` includes `name`, `email`, `password` |
| [ ] | T-13 | Seeders | `ARCH` | High | 1h | **Files to Create:**<br>- `sail artisan make:seeder UserSeeder` → 2 users: `test@example.com` / `password` and `other@example.com` / `password`<br>- `sail artisan make:seeder RecuSeeder` → 5 receipts for user 1 (mixed statuses: 2 en_attente, 2 traité, 1 échoué), 2 for user 2<br>- `sail artisan make:seeder DepenseSeeder` → 15 expenses distributed across receipts (3 per receipt) with varied categories<br>- `DatabaseSeeder.php`: call all three seeders in order<br>- `sail artisan migrate:fresh --seed` must pass ✅ |
| [ ] | T-14 | Install Laravel Debugbar | `DEBUG` | High | 0.5h | **Action:**<br>- `sail composer require barryvdh/laravel-debugbar --dev`<br>- Set `DEBUGBAR_ENABLED=true` in `.env` (only in dev)<br>- Open `http://localhost` → Debugbar panel must appear at bottom<br>- Confirm the **SQL** tab is visible |
| [ ] | T-15 | Pest — Model unit tests | `TEST` | High | 1h | **Files to Create:**<br>- `sail artisan pest:test Unit/RecuTest --unit`<br>- **Tests to write:**<br>&nbsp;&nbsp;- `it('has correct fillable fields')` → assert `$fillable` matches expected array<br>&nbsp;&nbsp;- `it('returns statut_label accessor in French')` → create model instance, assert each status maps to correct French label<br>&nbsp;&nbsp;- `it('belongs to a user')` → assert relationship method exists and returns correct type<br>&nbsp;&nbsp;- `it('has many depenses')` → assert relationship exists<br>- `sail artisan pest:test Unit/DepenseTest --unit`<br>&nbsp;&nbsp;- `it('returns categorie_label accessor in French')`<br>&nbsp;&nbsp;- `it('belongs to a recu')`<br>- `sail artisan test` → all unit tests must pass ✅ |

**Sprint 1 — Definition of Done:**

- [ ] `sail up -d` starts all services without error
- [ ] `http://localhost` shows Laravel + Tailwind working
- [ ] `http://localhost:8081` shows phpMyAdmin
- [ ] `sail artisan migrate:fresh --seed` runs cleanly with no errors
- [ ] Debugbar visible on every page
- [ ] `sail artisan test` passes all unit tests (models, accessors, relationships)
- [ ] Both models have `$fillable`, relationships and accessors defined
- [ ] `AGENTS.md` created in repo root (empty, ready for documentation)

---

## 🎓 AUTOFORMATION BLOC 2 — Spec-Driven Dev & Structured Output

**Objective:** Master OpenSpec workflow and laravel/ai SDK structured output  
**Duration:** Tuesday 06/09 afternoon

| Done | # | Task | Label | Priority | Time | Learning Resources & Checklist |
| :---: | :--- | :--- | :---: | :---: | :---: | :--- |
| [ ] | AF-06 | OpenSpec fundamentals | `SPEC` | High | 1.5h | **Read & understand:**<br>- Hub: <https://intent-driven.dev/knowledge/openspec/> 📖<br>- Getting Started: <https://github.com/Fission-AI/OpenSpec/blob/main/docs/getting-started.md> 📖<br>**Checklist:**<br>- [ ] Understand the 3 phases: Intent → Specs → Tasks<br>- [ ] Know what a "Feature Spec" document contains<br>- [ ] Understand versioning in git (specs/ folder, commit per change)<br>**Example:** From "Extract expenses from receipt" intent, create a Spec document with acceptance criteria. |
| [ ] | AF-07 | Video: Spec-Driven Development with OpenCode & OpenSpec | `SPEC` | High | 1h | **Watch & take notes:**<br>- Video: <https://www.youtube.com/watch?v=M3dp9u1wZes> 🎥<br>**Checklist:**<br>- [ ] Watch how specs guide the Agent's workflow<br>- [ ] See real OpenSpec example in the video<br>- [ ] Note how specs prevent rework (agent + human aligned)<br>**After video:** Sketch out one feature intent → spec outline (pen & paper). |
| [ ] | AF-08 | laravel/ai SDK — official documentation | `EXTRACT` | High | 1h | **Read & understand:**<br>- Docs: <https://laravel.com/docs/13.x/ai-sdk> 📖<br>- Focus: Provider setup, completion calls, structured output section<br>**Checklist:**<br>- [ ] Know the 3 main components: Provider, Model, Prompt<br>- [ ] Understand how `AI::prompt()` works conceptually<br>- [ ] Recognize structured output syntax (JSON schema param)<br>**Example:** Call the Groq provider with a structured schema for expense extraction. |
| [ ] | AF-09 | Article: Introducing the Laravel AI SDK (launch article) | `EXTRACT` | High | 0.5h | **Read & take notes:**<br>- Article: <https://laravel.com/blog/introducing-the-laravel-ai-sdk> 📖<br>**Checklist:**<br>- [ ] See a real example of structured output in action<br>- [ ] Understand why structured output prevents parse errors<br>- [ ] Note the difference between "completion" and "structured" calls<br>**Example:** The article shows extracting data from text with guaranteed JSON structure. |
| [ ] | AF-10 | Tutorial: Laravel AI SDK in 30 minutes (hands-on) | `EXTRACT` | High | 1h | **Read & code along:**<br>- Tutorial: <https://hafiz.dev/blog/laravel-ai-sdk-tutorial-build-a-smart-assistant-in-30-minutes> 📖💻<br>**Checklist:**<br>- [ ] Follow the tutorial on your local machine<br>- [ ] Get a working call to Groq API via laravel/ai<br>- [ ] See structured output work in real code<br>- [ ] Adapt the example: extract invoice items instead of generic data (mental exercise)<br>**Deliverable:** Working code snippet calling laravel/ai with structured output. |
| [ ] | AF-11 | Video: Build live agent with Laravel AI SDK | `EXTRACT` | High | 1h | **Watch & take notes:**<br>- Video: <https://redberry.international/Laravel-AI-SDK-practical-guide/> 🎥<br>**Checklist:**<br>- [ ] See live coding with the SDK<br>- [ ] Note error handling patterns (fallback providers)<br>- [ ] Understand how to debug AI calls<br>**After video:** Write pseudocode for `ExtractExpensesFromRecuJob` using what you saw. |

**AUTOFORMATION BLOC 2 — Definition of Done:**

- [ ] Can explain the 3 phases of OpenSpec (Intent → Specs → Tasks)
- [ ] Ran the 30-minute tutorial successfully and got structured output to work
- [ ] Understand the project's JSON contract (articles array structure)
- [ ] Can sketch a Feature Spec for "Extract expenses from receipt"
- [ ] Have a working `.env` with valid `AI_API_KEY` (Groq or OpenRouter) tested

---

## 🏃 Sprint 2 — Authentication (US1)

**Objective:** Registration, Login and Logout fully functional with main layout  
**Duration:** Tuesday 06/09 afternoon → Wednesday 06/10 morning  
**Branch:** `feature/auth` (if separating auth from CRUD)

| Done | # | Task | Label | Priority | Time | Detailed Implementation & Files |
| :---: | :--- | :--- | :---: | :---: | :---: | :--- |
| [ ] | T-16 | Install Laravel Breeze (Blade) | `AUTH` | High | 0.5h | **Action:**<br>- `sail composer require laravel/breeze --dev`<br>- `sail artisan breeze:install blade`<br>- `sail npm run dev`<br>- `sail artisan migrate`<br>- Test `http://localhost/register` → register form must appear |
| [ ] | T-17 | `US1` — Register / Login / Logout | `AUTH` | High | 1.5h | **Files to Verify/Edit:**<br>- `resources/views/auth/register.blade.php`: fields `name`, `email`, `password`, `password_confirmation`, `@csrf`, `@error` messages<br>- `resources/views/auth/login.blade.php`: fields `email`, `password`, `remember`, `@csrf`<br>- `RegisteredUserController@store`: validate → `User::create()` → redirect `/dashboard`<br>- `AuthenticatedSessionController@destroy`: `Auth::logout()` → redirect `/`<br>- **Routes:** `GET/POST /register`, `GET/POST /login`, `POST /logout` |
| [ ] | T-18 | Main layout `layouts/app.blade.php` | `AUTH` | High | 1.5h | **Files to Create/Edit:**<br>- `resources/views/layouts/app.blade.php`:<br>&nbsp;&nbsp;- `@vite(['resources/css/app.css', 'resources/js/app.js'])` in `<head>`<br>&nbsp;&nbsp;- Navbar with `@auth` → Dashboard + Logout button · `@guest` → Login + Register<br>&nbsp;&nbsp;- Flash messages: `@if(session('success'))` and `@if(session('error'))`<br>&nbsp;&nbsp;- `@yield('content')` in main body<br>&nbsp;&nbsp;- Add dark navy theme (`#1a1f36`) if specified |
| [ ] | T-19 | Protect all routes under `auth` middleware | `AUTH` | High | 0.3h | **Files to Edit:**<br>- `routes/web.php`: wrap all receipt + expense routes in `Route::middleware('auth')->group(function () { ... })`<br>- Redirect to `/login` is default behavior with `auth` middleware<br>- Test: direct access to `/dashboard` without login → must redirect to `/login` |
| [ ] | T-20 | Pest — Authentication feature tests | `TEST` | High | 1.5h | **Files to Create:**<br>- `sail artisan pest:test Feature/AuthTest`<br>- **Tests to write:**<br>&nbsp;&nbsp;- `it('allows a guest to register with valid data')` → POST `/register`, assert user created in DB, assert redirected to `/dashboard`<br>&nbsp;&nbsp;- `it('blocks registration with invalid data')` → POST without email, assert session has errors<br>&nbsp;&nbsp;- `it('allows a registered user to login')` → POST `/login`, assert authenticated<br>&nbsp;&nbsp;- `it('rejects login with wrong password')` → assert not authenticated<br>&nbsp;&nbsp;- `it('logs out an authenticated user')` → POST `/logout`, assert guest<br>&nbsp;&nbsp;- `it('redirects guests to login when accessing dashboard')` → GET `/dashboard`, assert redirect to `/login`<br>- `sail artisan test` → all auth tests must pass ✅ |

**Sprint 2 — Definition of Done:**

- [ ] Registration creates a new user and redirects to `/dashboard`
- [ ] Login with seeded credentials works
- [ ] Logout redirects to `/` or `/login`
- [ ] Direct access to `/dashboard` without login → redirect `/login`
- [ ] `@auth`/`@guest` in layout shows correct nav links
- [ ] Flash messages display correctly
- [ ] All authentication Pest tests pass ✅

---

## 🏃 Sprint 3 — Receipts CRUD (US2, US3, US4, US5)

**Objective:** Full receipt management — list, create, view, delete  
**Duration:** Wednesday 06/10 afternoon → Thursday 06/11 morning  
**Branch:** `feature/recus-crud`

| Done | # | Task | Label | Priority | Time | Detailed Implementation & Files |
| :---: | :--- | :--- | :---: | :---: | :---: | :--- |
| [ ] | T-21 | `RecuPolicy` — Authorization rules | `ARCH` | High | 0.5h | **Files to Create:**<br>- `sail artisan make:policy RecuPolicy --model=Recu`<br>- **Methods to define:**<br>&nbsp;&nbsp;- `viewAny(User $user)` → always true<br>&nbsp;&nbsp;- `view(User $user, Recu $recu)` → `$recu->user_id === $user->id`<br>&nbsp;&nbsp;- `create(User $user)` → always true<br>&nbsp;&nbsp;- `delete(User $user, Recu $recu)` → `$recu->user_id === $user->id`<br>&nbsp;&nbsp;- `restore(User $user, Recu $recu)` → `$recu->user_id === $user->id` |
| [ ] | T-22 | `RecuController` — Scaffold | `RECU` | High | 0.5h | **Files to Create:**<br>- `sail artisan make:controller RecuController --resource`<br>- Add `$this->authorize()` calls in every relevant method<br>- Use `with()` on every query — zero N+1 |
| [ ] | T-23 | `US2` — Dashboard (receipt list) | `RECU` | High | 1.5h | **Files to Create/Edit:**<br>- `RecuController@index`:<br>&nbsp;&nbsp;- `$recus = auth()->user()->recus()->with('depenses')->get()`<br>&nbsp;&nbsp;- Pass to view: `['recus' => $recus]`<br>- `resources/views/recus/index.blade.php`:<br>&nbsp;&nbsp;- `@forelse` loop — display empty state if no receipts<br>&nbsp;&nbsp;- Card per receipt: date, status label, depenses count, preview of first 100 chars of text<br>&nbsp;&nbsp;- View Detail + Delete buttons<br>- **Route:** `GET /dashboard` → `recus.index` (named route) |
| [ ] | T-24 | `US3` — Submit a receipt | `RECU` | High | 1.5h | **Files to Create/Edit:**<br>- `RecuController@create`: `$this->authorize('create', Recu::class)`<br>- `RecuController@store`:<br>&nbsp;&nbsp;- `$this->authorize('create', Recu::class)`<br>&nbsp;&nbsp;- Validate via `StoreRecuRequest`<br>&nbsp;&nbsp;- `$recu = Recu::create([..., 'user_id' => auth()->id()])`<br>&nbsp;&nbsp;- Return redirect with success flash: "Reçu reçu. Traitement en cours..."<br>- `resources/views/recus/create.blade.php`: textarea for `texte_brut` (min 10, max 5000 chars), `@csrf`<br>- **Routes:** `GET /recus/create` → `recus.create` · `POST /recus` → `recus.store` |
| [ ] | T-25 | `StoreRecuRequest` + `UpdateRecuRequest` | `ARCH` | High | 0.5h | **Files to Create:**<br>- `sail artisan make:request StoreRecuRequest`<br>&nbsp;&nbsp;- `authorize()` → `return true;`<br>&nbsp;&nbsp;- `rules()`: `texte_brut\|required\|string\|min:10\|max:5000`<br>- `sail artisan make:request UpdateRecuRequest`<br>&nbsp;&nbsp;- Same rules as Store (if needed for editing status later) |
| [ ] | T-26 | `US4` — View receipt details | `RECU` | High | 1h | **Files to Create/Edit:**<br>- `RecuController@show`:<br>&nbsp;&nbsp;- `$this->authorize('view', $recu)`<br>&nbsp;&nbsp;- `$recu->load('depenses')`<br>- `resources/views/recus/show.blade.php`:<br>&nbsp;&nbsp;- Display `texte_brut` as submitted text (formatted, maybe in `<pre>` tag)<br>&nbsp;&nbsp;- Display `statut_label` with color coding (pending → yellow, processed → green, failed → red)<br>&nbsp;&nbsp;- Expenses list section: libellé, quantité, prix_unitaire, categorie_label<br>&nbsp;&nbsp;- If status = `traité`, show total amount (sum of quantité * prix_unitaire)<br>- **Route:** `GET /recus/{recu}` → `recus.show` |
| [ ] | T-27 | `US5` — Delete a receipt (SoftDelete) | `RECU` | High | 0.5h | **Files to Edit:**<br>- `RecuController@destroy`:<br>&nbsp;&nbsp;- `$this->authorize('delete', $recu)`<br>&nbsp;&nbsp;- `$recu->delete()` (SoftDeletes — sets `deleted_at`)<br>&nbsp;&nbsp;- `return redirect()->route('recus.index')->with('success', 'Reçu supprimé.')`<br>- Delete button in `recus/index.blade.php` with `@method('DELETE')` form, `@csrf`, JS confirm<br>- **Route:** `DELETE /recus/{recu}` → `recus.destroy` |
| [ ] | T-28 | Pest — Receipt policy tests | `TEST` | High | 1h | **Files to Create:**<br>- `sail artisan pest:test Feature/RecuPolicyTest`<br>- **Tests to write:**<br>&nbsp;&nbsp;- `it('blocks a user from viewing another user receipt')` → actingAs(user2), GET `/recus/{user1_recu}`, assert 403<br>&nbsp;&nbsp;- `it('blocks a user from deleting another user receipt')` → actingAs(user2), DELETE, assert 403<br>&nbsp;&nbsp;- `it('allows a user to manage their own receipt')` → actingAs(owner), assert 200/redirect<br>- `sail artisan test` → all policy tests must pass ✅ |
| [ ] | T-29 | Pest — Receipt CRUD feature tests | `TEST` | High | 1.5h | **Files to Create:**<br>- `sail artisan pest:test Feature/RecuCrudTest`<br>- **Tests to write:**<br>&nbsp;&nbsp;- `it('creates a receipt with valid text')` → actingAs(user), POST `/recus`, assert in DB with status `en_attente`, assert redirect<br>&nbsp;&nbsp;- `it('rejects receipt creation with text too short')` → POST with 5 chars, assert session errors<br>&nbsp;&nbsp;- `it('rejects receipt creation with text too long')` → POST with 6000+ chars, assert session errors<br>&nbsp;&nbsp;- `it('shows receipt in dashboard after creation')` → create, GET dashboard, assert visible<br>&nbsp;&nbsp;- `it('displays receipt details page correctly')` → GET show, assert `texte_brut` and depenses visible<br>&nbsp;&nbsp;- `it('deletes a receipt using soft delete')` → DELETE, assert `deleted_at` is not null in DB<br>- `sail artisan test` → all CRUD tests must pass ✅ |

**Sprint 3 — Definition of Done:**

- [ ] Dashboard shows only the authenticated user's receipts
- [ ] Status labels displayed in French (`statut_label`) — no raw enum values
- [ ] `@forelse` on all lists with empty state handled
- [ ] Create, view, delete all functional
- [ ] Policy blocks cross-user access (403)
- [ ] Zero `abort(403)` — all through `$this->authorize()`
- [ ] All Pest CRUD and Policy tests pass ✅
- [ ] Commit message: `feat(recus): implement CRUD with soft delete and authorization`

---

## 🎓 AUTOFORMATION BLOC 3 — Jobs, Queues & Casts

**Objective:** Master asynchronous processing and data typing  
**Duration:** Thursday 06/11 afternoon

| Done | # | Task | Label | Priority | Time | Learning Resources & Checklist |
| :---: | :--- | :--- | :---: | :---: | :---: | :--- |
| [ ] | AF-12 | Laravel Queue Mastery series (videos) | `QUEUE` | High | 2h | **Watch all videos in series:**<br>- Series: <https://laracasts.com/series/laravel-queue-mastery> 🎥<br>**Checklist after watching:**<br>- [ ] Understand why queues exist (don't freeze the page)<br>- [ ] Know the difference between sync and async drivers<br>- [ ] See a Job being created and dispatched<br>- [ ] Understand the queue:work command (the worker loop)<br>**Example from series:** Job dispatches → queue stores it → worker picks it up → executes it. |
| [ ] | AF-13 | Official Laravel Queues documentation | `QUEUE` | High | 1h | **Read & understand:**<br>- Docs: <https://laravel.com/docs/13.x/queues> 📖<br>- Focus: Creating jobs, dispatching, drivers (database, redis)<br>**Checklist:**<br>- [ ] Understand `php artisan make:job JobName`<br>- [ ] Know what `handle()` method does<br>- [ ] Recognize `dispatch()` syntax: `JobName::dispatch($param)`<br>- [ ] Know why database driver is easiest for this project<br>**Example:** Pseudo-code for `ExtractExpensesJob::dispatch($recu)`. |
| [ ] | AF-14 | Beginner's guide to Laravel Queues (step-by-step) | `QUEUE` | High | 1.5h | **Read & code along:**<br>- Guide: <https://www.jtechbd.net/laravel-queues-jobs-tutorial-beginners> 📖💻<br>**Checklist:**<br>- [ ] Create a test job (follows the guide)<br>- [ ] Dispatch it from a controller<br>- [ ] Run `queue:work` and see it execute<br>- [ ] Verify the job removes from queue after success<br>**Deliverable:** Working code: controller dispatches a job, worker processes it. |
| [ ] | AF-15 | Eloquent Accessors, Mutators & Casting | `CAST` | High | 1h | **Read & understand:**<br>- Docs: <https://laravel.com/docs/13.x/eloquent-mutators> 📖<br>- Focus: Casts section (enum, array, json, date)<br>**Checklist:**<br>- [ ] Understand what a cast does (serialize/deserialize)<br>- [ ] Know the difference between Accessor and Cast<br>- [ ] Recognize `protected $casts = [...]` syntax<br>- [ ] Know why JSON payload should be cast as `array`<br>**Example:** `payload_ia` cast as `array` so you can access `$recu->payload_ia['articles']` naturally. |
| [ ] | AF-16 | Practice: Cast & accessor integration | `CAST` | High | 0.5h | **Code exercise (on your repo):**<br>- Open your `Recu` model<br>- Add cast for `payload_ia` as `array`<br>- Write a test: create Recu with JSON `payload_ia`, verify you can access nested keys<br>- **Example:** `$recu->payload_ia['articles'][0]['libellé']` should work naturally<br>**Deliverable:** Working unit test showing cast + accessor together. |

**AUTOFORMATION BLOC 3 — Definition of Done:**

- [ ] Watched all Queue Mastery videos and took notes
- [ ] Can explain why a queue job prevents page freezing
- [ ] Ran the beginner's guide tutorial successfully (job dispatched & processed)
- [ ] Understand the flow: Controller → dispatch Job → queue stores → worker executes
- [ ] Can cast `payload_ia` as `array` and access nested keys
- [ ] Have a working `.env` with `QUEUE_CONNECTION=database`

---

## 🏃 Sprint 4 — AI Extraction with Structured Output (US6)

**Objective:** Extract expenses from receipt text using laravel/ai with guaranteed JSON structure  
**Duration:** Thursday 06/11 afternoon → Friday 06/12 morning  
**Branch:** `feature/extraction-ia`

> **Before starting:** Complete AUTOFORMATION BLOC 2 (Spec-Driven Dev & Structured Output)

| Done | # | Task | Label | Priority | Time | Detailed Implementation & Files |
| :---: | :--- | :--- | :---: | :---: | :---: | :--- |
| [ ] | T-30 | `AGENTS.md` — Project AI usage documentation | `SPEC` | High | 0.5h | **Files to Create:**<br>- `AGENTS.md` in repo root<br>- **Content sections:**<br>&nbsp;&nbsp;- "Overview" — what this AI assistant does in this project<br>&nbsp;&nbsp;- "Model" — which LLM we use (Groq grok-4.3 via laravel/ai)<br>&nbsp;&nbsp;- "Prompting Strategy" — how we structure the prompt for structured output<br>&nbsp;&nbsp;- "Structured Contract" — the JSON schema expenses must follow<br>&nbsp;&nbsp;- "Error Handling" — how we handle AI failures gracefully<br>**Example:** See InterviewPrep's AGENTS.md format |
| [ ] | T-31 | Feature Spec: "Extract expenses from receipt" via OpenSpec | `SPEC` | High | 1h | **Files to Create:**<br>- `specs/feature-extract-expenses.md` (managed by OpenSpec)<br>- **Sections:**<br>&nbsp;&nbsp;- Feature Intent: "Parse Darija receipt text into structured expense list"<br>&nbsp;&nbsp;- JSON Contract (the schema)<br>&nbsp;&nbsp;- Acceptance Criteria (3 clear criteria)<br>&nbsp;&nbsp;- Example: "Given receipt text in Darija with 3 items, the AI extracts exactly 3 expenses with correct categories"<br>- Commit: `docs: add spec for expense extraction via OpenSpec` |
| [ ] | T-32 | Install laravel/ai SDK + configure Groq provider | `EXTRACT` | High | 0.5h | **Action:**<br>- `sail composer require laravel/ai`<br>- Add to `.env`: `AI_API_KEY=<your-groq-api-key>` (get from groq.com)<br>- Publish config: `sail artisan vendor:publish --provider="Laravel\AI\AIServiceProvider"`<br>- Edit `config/ai.php`: set default driver to `groq`, ensure model is `grok-4.3`<br>- **Test:** Create a simple test route that calls `AI::prompt()` and see it work |
| [ ] | T-33 | Create `ExtractExpensesJob` | `EXTRACT` | High | 1.5h | **Files to Create:**<br>- `sail artisan make:job ExtractExpensesFromRecu`<br>- **Constructor:**<br>&nbsp;&nbsp;- Accept `Recu $recu` (pass by ID, not instance, to avoid serialization issues)<br>&nbsp;&nbsp;- Store `$this->recuId = $recu->id`<br>- **`handle()` method:**<br>&nbsp;&nbsp;1. Fetch recu: `$recu = Recu::findOrFail($this->recuId)`<br>&nbsp;&nbsp;2. Build prompt with recu text + JSON contract<br>&nbsp;&nbsp;3. Call `AI::structured()` with the schema<br>&nbsp;&nbsp;4. Save payload: `$recu->update(['payload_ia' => $payload, 'statut' => 'traité'])`<br>&nbsp;&nbsp;5. Create Depense records for each article<br>&nbsp;&nbsp;6. On error: catch exception, log it, set `statut = 'échoué'`, save error in `payload_ia`<br>- **Prompt structure:**<br>&nbsp;&nbsp;``r>&nbsp;&nbsp;"Extract expenses from this Darija receipt text into structured JSON.<br>&nbsp;&nbsp;Receipt text: {$recu->texte_brut}<br>&nbsp;&nbsp;Return JSON matching this contract: ..."<br>&nbsp;&nbsp;```<br>- Use `AI::structured('groq:grok-4.3', [...])->json()` |
| [ ] | T-34 | Dispatch job from controller on receipt creation | `EXTRACT` | High | 0.5h | **Files to Edit:**<br>- `RecuController@store`:<br>&nbsp;&nbsp;- After `Recu::create()`, immediately dispatch job:<br>&nbsp;&nbsp;- `ExtractExpensesFromRecu::dispatch($recu)`<br>&nbsp;&nbsp;- Return flash: "Reçu reçu. Traitement en cours..." (no wait)<br>- No change to view — submission returns immediately |
| [ ] | T-35 | Implement structured JSON contract in Job | `EXTRACT` | High | 0.5h | **Files to Edit:**<br>- `ExtractExpensesJob@handle()`: use the project's JSON contract:<br>&nbsp;&nbsp;```json<br>&nbsp;&nbsp;{<br>&nbsp;&nbsp;  "articles": [<br>&nbsp;&nbsp;    {<br>&nbsp;&nbsp;      "libellé": "string",<br>&nbsp;&nbsp;      "quantité": "integer&nbsp;&nbsp;      "prix_unitaire": "number",<br>&nbsp;&nbsp;      "catégorie": "enum (alimentaire|boissons|hygiène|entretien|autre)"<br>&nbsp;&nbsp;    }<br>&nbsp;&nbsp;  ],<br>&nbsp;&nbsp;  "total_estimé": "number",<br>&nbsp;&nbsp;  "devise": "string"<br>&nbsp;&nbsp;}<br>&nbsp;&nbsp;```<br>- Pass this schema to `AI::structured()`'s third parameter (if using jsonSchema lib) |
| [ ] | T-36 | Create Depense records from extracted payload | `EXTRACT` | High | 1h | **Files to Edit:**<br>- `ExtractExpensesJobndle()`: after AI returns payload<br>&nbsp;&nbsp;- Loop through `$payload['articles']`<br>&nbsp;&nbsp;- For each article, create Depense:<br>&nbsp;&nbsp;```php<br>&nbsp;&nbsp;Depense::create([<br>&nbsp;&nbsp;  'recu_id' => $recu->id,<br>&nbsp;&nbsp;  'libellé' => $article['libellé'],<br>&nbsp;&nbsp;  'quantité' => $article['quantité'],<br>&nbsp;&nbsp;  'prix_unitaire' => $article['prix_unitaire'],<br>&nbsp;&nbsp;  'catégorie' => $article['catégorie'],<br>&nbsp;&nbsp;])<br>&nbsp;&nbsp;```<br>- Validatefield before creating (or wrap in validator)<br>- If validation fails, throw exception to mark recu as failed |
| [ ] | T-37 | Error handling: Mark receipt as failed on exception | `EXTRACT` | High | 0.5h | **Files to Edit:**<br>- `ExtractExpensesJob@handle()`: wrap in try/catch<br>&nbsp;&nbsp;```php<br>&nbsp;&nbsp;try {<br>&nbsp;&nbsp;  // AI call + depense creation<br>&nbsp;&nbsp;} catch (Exception $e) {<br>&nbsp;&nbsp;  Log::error('Expense extraction failed', ['recu_id' => $recu->id, 'error' => $e->getMessage()]);<br>&nbsp;&nbsp;  $recu->update([\br>&nbsp;&nbsp;    'statut' => 'échoué',<br>&nbsp;&nbsp;    'payload_ia' => ['erreur' => $e->getMessage()]\br>&nbsp;&nbsp;  ]);<br>&nbsp;&nbsp;  throw $e; // Let queue know job failed<br>&nbsp;&nbsp;}<br>&nbsp;&nbsp;```<br>- Status now reflects real outcome: `en_attente` → `traité` | `échoué` |
| [ ] | T-38 | Pest — Test structured output with fake SDK | `TEST` | High | 1.5h | **Files to Create:**<br>- `sail artisan pest:test Feature/ExtractionTest`<br>- avel/ai fake:**<br>&nbsp;&nbsp;```php<br>&nbsp;&nbsp;AI::fake([\br>&nbsp;&nbsp;  AI::structured() => response(\br>&nbsp;&nbsp;    new Content(\br>&nbsp;&nbsp;      text: '',<br>&nbsp;&nbsp;      data: [\br>&nbsp;&nbsp;        'articles' => [\br>&nbsp;&nbsp;          ['libellé' => 'Lait', 'quantité' => 2, 'prix_unitaire' => 5.5, 'catégorie' => 'alimentaire'],\br>&nbsp;&nbsp;        ],\br>&nbsp;&nbsp;        'total_estimé' => 11,\br>&nbsp;&nbsp;        'devise' => 'MAD'\br>&nbsp;&nbsp;      ]\br>&nbsp;&nb   )\br>&nbsp;&nbsp;  )<br>&nbsp;&nbsp;]);<br>&nbsp;&nbsp;```<br>- **Tests to write:**<br>&nbsp;&nbsp;- `it('extracts expenses from receipt text with fake SDK')` → dispatch job, assert Depenses created in DB<br>&nbsp;&nbsp;- `it('creates depense records with correct categories')` → verify category mapping<br>&nbsp;&nbsp;- `it('sets receipt status to traité after extraction')` → assert `$recu->statut == 'traité'`<br>&nbsp;&nbsp;- `it('handles extraction failure gracefully')` → fake an error respons`statut == 'échoué'`<br>- `sail artisan test` → all extraction tests must pass ✅ |
| [ ] | T-39 | Manual test: Full flow receipt → extraction | `TEST` | High | 1h | **Checklist:**<br>- [ ] Create fresh user via registration<br>- [ ] Navigate to `/recus/create`<br>- [ ] Paste a sample Darija receipt text (min 10 chars)<br>- [ ] Submit → flash "Traitement en cours"<br>- [ ] Back to dashboard → status still `en_attente` (extraction hasn't run yet)<br>- [ ] Run `sail artisan queue:work` in separate - [ ] Refresh dashboard → status should be `traité` + expenses appear<br>- [ ] Each expense shows correct libellé, quantité, prix, categorie_label<br>- [ ] Click View → see full receipt details with expense table |

**Sprint 4 — Definition of Done:**

- [ ] `AGENTS.md` documented with AI strategy
- [ ] Feature spec created and committed
- [ ] laravel/ai SDK installed and Groq configured
- [ ] Job created and dispatches from controller
- [ ] Structured output contract enforced
- [ ] Depense records rom AI payload
- [ ] Error handling: failed extractions marked with `statut = 'échoué'`
- [ ] Pest tests with fake SDK all pass ✅
- [ ] Manual end-to-end flow works (receipt → extraction → expenses visible)
- [ ] Commit message: `feat(extraction): implement AI-powered structured extraction with job dispatch`

---

## 🎓 AUTOFORMATION BLOC 4 — N+1 Prevention & Eager Loading

**Objective:** Identify and eliminate N+1 queries using Debugbar  

**Duration:** Friday 06/12 morning (parallel with Sprint | # | Task | Label | Priority | Time | Learning Resources & Checklist |
| :---: | :--- | :--- | :---: | :---: | :---: | :--- |
| [ ] | AF-17 | Eager Loading section from Laravel docs | `DEBUG` | High | 0.5h | **Read & understand:**<br>- Docs section: <https://laravel.com/docs/13.x/eloquent-relationships#eager-loading> 📖<br>**Checklist:**<br>- [ ] Understand N+1 problem conceptually (1 query for list + 1 per item)<br>- [ ] Know `with()` syntax: `$recus = $user->recus()->with('depenses')->get()`<br>- [ ] Reognize `withCount()` for counts: `->withCount('depenses')`<br>- [ ] Know lazy loading vs eager loading difference<br>**Example:** Without `with()`, looping `$recus->each($r => $r->depenses)` fires 1+N queries. |
| [ ] | AF-18 | Use Debugbar to detect N+1 in your app | `DEBUG` | High | 1h | **Hands-on exercise:**<br>- Open `/dashboard` with seeded data (5+ receipts)<br>- Open Debugbar (bottom right)<br>- Click **SQL** tab<br>- **Count queries:** Should be ≈ 2–3 (one for recus, one for depenses if eager-ld)<br>- If you see 7+ queries, you have N+1<br>- **Fix:** add `.with('depenses')` to controller query<br>- Refresh and verify query count drops<br>**Deliverable:** Screenshot of Debugbar SQL tab showing ≤ 3 queries for dashboard |
| [ ] | AF-19 | Verify zero N+1 across all routes | `DEBUG` | High | 0.5h | **Audit each route:**<br>- `GET /dashboard` → should load recus + depenses in ≤ 3 queries<br>- `GET /recus/{id}` → should load recu + depenses in ≤ 2 queries<br>- Use Debugbar on each → document<br>- If any route > expected, add eager loading<br>**Deliverable:** List of all routes with their query counts (confirmed via Debugbar) |

**AUTOFORMATION BLOC 4 — Definition of Done:**

- [ ] Read eager loading docs and understand N+1 conceptually
- [ ] Used Debugbar to detect N+1 on your dashboard
- [ ] Fixed any N+1 issues with `.with()` or `.withCount()`
- [ ] Verified all routes use eager loading
- [ ] Dashboard loads in ≤ 3 queries regardless of number of receipts

---

## 🏃 Sprint 5 — Queusing & Status Tracking (US7)

**Objective:** Make queue worker process jobs, display status updates in real-time, handle failures gracefully  
**Duration:** Friday 06/12 morning  
**Branch:** `feature/queue-processing`

> **Before starting:** Complete AUTOFORMATION BLOC 3 (Jobs & Queues)

| Done | # | Task | Label | Priority | Time | Detailed Implementation & Files |
| :---: | :--- | :--- | :---: | :---: | :---: | :--- |
| [ ] | T-40 | Migrate queue table for database driver | `QUEUE` | High | 0.3h | **Action:**<br>- `sail artisan queue:table`<br>- `sail artisan migrate`<br>- Verify `jobs` and `failed_jobs` tables exist in DB (via phpMyAdmin) |
| [ ] | T-41 | `US7` — Status tracking with visual indicators | `RECU` | High | 1h | **Files to Edit:**<br>- `resources/views/recus/index.blade.php`:<br>&nbsp;&nbsp;- Add status badge next to each receipt:<br>&nbsp;&nbsp;```blade<br>&nbsp;&nbsp;@if($recu->statut === 'en_attente')<br>&nbsp;&nbsp;  <span class="badge bg-yellow-600">En attente</span><br>&nbsp;&nbsp;@elsf($recu->statut === 'traité')<br>&nbsp;&nbsp;  <span class="badge bg-green-600">Traité ({{ $recu->depenses_count }} articles)</span><br>&nbsp;&nbsp;@else<br>&nbsp;&nbsp;  <span class="badge bg-red-600">Échoué</span><br>&nbsp;&nbsp;@endif<br>&nbsp;&nbsp;```<br>- Use Tailwind utility classes for color coding<br>- Show depenses count only if `traité` |
| [ ] | T-42 | Auto-refresh dashboard to show status updates | `RECU` | High | 0.5h | **Files to Edit:**<br>- `resources/views/recus/index.blade.php`: add e polling with HTML `<meta>` refresh<br>&nbsp;&nbsp;```html<br>&nbsp;&nbsp;<meta http-equiv="refresh" content="3"> <!-- Refresh every 3 seconds --><br>&nbsp;&nbsp;```<br>- Or use a small JS interval for smoother UX (optional)<br>- User submits receipt → sees "En attente" → page auto-refreshes → status changes when job completes |
| [ ] | T-43 | Run queue worker in separate terminal | `QUEUE` | High | 0.3h | **Action:**<br>- Keep `sail up -d` running<br>- In new terminal: `sail artisan queue:work --tim00`<br>- Jobs from queue now process immediately<br>- Open another browser → submit receipt → see status change from `en_attente` to `traité` after 2–3 seconds |
| [ ] | T-44 | Pest — Queue job processing tests | `TEST` | High | 1.5h | **Files to Create:**<br>- `sail artisan pest:test Feature/QueueProcessingTest`<br>- **Tests to write:**<br>&nbsp;&nbsp;- `it('dispatches extraction job on receipt creation')` → create receipt, assert job in queue<br>&nbsp;&nbsp;- `it('processes job and creates depeispatch job, call`handle()` directly or use `Queue::fake()`, assert Depenses created<br>&nbsp;&nbsp;-`it('marks receipt as traité after successful extraction')` → assert `statut` changed<br>&nbsp;&nbsp;- `it('marks receipt as échoué on extraction error')` → simulate error, assert `statut = 'échoué'`<br>&nbsp;&nbsp;-`it('logs error when extraction fails')` → assert log entry created<br>- `sail artisan test` → all queue tests must pass ✅ |
| [ ] | T-45 | Pest — Expense filtering feature tets | `TEST` | High | 1h | **Files to Create:**<br>- `sail artisan pest:test Feature/DepenseFilteringTest`<br>- **Tests to write:**<br>&nbsp;&nbsp;- `it('lists all expenses for a user')` → assert all depenses visible on dashboard<br>&nbsp;&nbsp;- `it('filters expenses by category')` → pass `?categorie=alimentaire`, assert only that category shown<br>&nbsp;&nbsp;- `it('returns empty list when filtering yields no results')` → filter with non-existent category, assert empty<br>- `sail artisan test` → aling tests must pass ✅ |

**Sprint 5 — Definition of Done:**

- [ ] Queue table created and migrated
- [ ] Status badges visible on dashboard (yellow/green/red)
- [ ] Dashboard auto-refreshes every 3 seconds
- [ ] Queue worker running processes jobs asynchronously
- [ ] Manual flow: submit receipt → "En attente" → page refreshes → "Traité" (while worker processes)
- [ ] Failed extractions show "Échoué" status
- [ ] All queue processing tests pass ✅

---

## 🏃 Sprint 6 — Expenses List & Firing (US8 + Bonus)

**Objective:** Full expense tracking with category filtering  
**Duration:** Friday 06/12 morning (parallel with Sprint 5)

| Done | # | Task | Label | Priority | Time | Detailed Implementation & Files |
| :---: | :--- | :--- | :---: | :---: | :---: | :--- |
| [ ] | T-46 | `DepenseController` — Scaffold | `ARCH` | High | 0.3h | **Files to Create:**<br>- `sail artisan make:controller DepenseController`<br>- Methods: `index()` (list all user expenses with filters), `destroy()` (delete sile expense) |
| [ ] | T-47 | `US8` — Filterable expense list | `ARCH` | High | 1.5h | **Files to Create/Edit:**<br>- `DepenseController@index`:<br>&nbsp;&nbsp;- `$depenses = auth()->user()->recus()->with('depenses')->get()->pluck('depenses')->flatten()`<br>&nbsp;&nbsp;- Read `?categorie=` query param<br>&nbsp;&nbsp;- Filter: `$depenses = $depenses->when($categorie, fn($c) => $c->where('categorie', $categorie))`<br>&nbsp;&nbsp;- Pass to view: `['depenses' => $depenses, 'categories' => ['alimentaire', 'boisns', 'hygiène', 'entretien', 'autre']]`<br>- `resources/views/depenses/index.blade.php`:<br>&nbsp;&nbsp;- Filter form: `GET` method, `<select>` for categories, Submit + Clear buttons<br>&nbsp;&nbsp;- Table: libellé, quantité, prix_unitaire, categorie_label, recu link<br>&nbsp;&nbsp;- `@forelse` with empty state<br>&nbsp;&nbsp;- Optional: totals row at bottom (sum by category)<br>- **Route:** `GET /depenses` → `depenses.index` (named route) |
| [ ] | T-48 | Add Expenses link to navbar | `ARCH` | High | | **Files to Edit:**<br>- `resources/views/layouts/app.blade.php`:<br>&nbsp;&nbsp;- Add link to `/depenses` in authenticated navbar<br>&nbsp;&nbsp;- Text: "Dépenses" |
| [ ] | T-49 | Pest — Depense filtering tests (see T-45 also) | `TEST` | High | 1h | **Tests to write (if not in T-45):**<br>- `it('displays total amount per category')` → create mixed depenses, assert totals calculated<br>- `it('shows empty state when no depenses')` → brand new user, assert message<br>- `it('preserves filter selectione refresh')` → filter → refresh → filter persists<br>- `sail artisan test` → all depense tests must pass ✅ |
| [ ] | T-50 | Bonus: Upload receipt image instead of text | `BONUS` | Low | 2h | **Files to Create/Edit:**<br>- Migration: add `image_path` (string, nullable) to `recus` table<br>- `StorRecuRequest`: add `image\|nullable\|image\|mimes:jpg,png,jpeg\|max:5120` OR make `texte_brut` nullable if uploading<br>- `RecuController@create`: show both textarea and file upload<br>- `RecuController@storge uploaded, save to`storage/app/public/receipts/` via `$request->file('image')->store()`<br>-`ExtractExpensesJob`: if image exists, use multimodal prompt (send image to Groq instead of text)<br>- **Note:** Groq supports image input; construct the prompt with image encoding<br>-`sail artisan storage:link` to create public symlink<br>- Pest test: `it('extracts expenses from uploaded image')` with fake SDK returning sample payload |

**Sprint 6 — Definition of Done:**

- [ ] Expenses page shows all user'depenses across all receipts
- [ ] Filter by category working (preserves selection)
- [ ] Empty state message if no depenses
- [ ] Category labels in French on expense list
- [ ] Expenses linked back to their receipt
- [ ] (Bonus) Image upload + multimodal extraction working
- [ ] All filtering tests pass ✅

---

## 🏃 Sprint 7 — QA, Debugging & Final Livrables

**Objective:** Security audit, N+1 fix, full test suite green, README, commits check, live demo readiness  
**Duration:** Friday 06/12 (all d

| Done | # | Task | Label | Priority | Time | Detailed Implementation & Files |
| :---: | :--- | :--- | :---: | :---: | :---: | :--- |
| [ ] | T-51 | Audit — All Form Requests in place | `QA` | High | 0.3h | **Files to Audit:**<br>- `RecuController@store` → uses `StoreRecuRequest` (not `$request->validate()`) ✅<br>- `DepenseController` → no direct creation (handled by job) ✅<br>- Auth forms → handled by Breeze ✅ |
| [ ] | T-52 | Audit — `@csrf` on all forms | `QA` | High | 0.2h | **Forms to Audit:**<br>- `recus/create.blade.php` → `@csrf` ✅<br>- `recus/index.blade.php` (delete form) → `@csrf` + `@method('DELETE')` ✅<br>- `depenses/index.blade.php` (filter form) → no CSRF needed (GET) ✅ |
| [ ] | T-53 | Audit — `$fillable` on all models | `QA` | High | 0.2h | **Files to Check:**<br>- `Recu::$fillable` = `['user_id', 'texte_brut', 'statut', 'payload_ia']`<br>- `Depense::$fillable` = `['recu_id', 'libellé', 'quantité', 'prix_unitaire', 'catégorie']`<br>- `User::$fillable` = `['nme', 'email', 'password']` (set by Breeze) |
| [ ] | T-54 | Audit — `@forelse` on all lists | `QA` | High | 0.2h | **Views to Check:**<br>- `recus/index.blade.php` → `@forelse` with `@empty` block showing "Aucun reçu" ✅<br>- `depenses/index.blade.php` → `@forelse` with `@empty` showing "Aucune dépense" ✅ |
| [ ] | T-55 | Audit — Zero `abort(403)` | `POLICY` | High | 0.2h | **Action:**<br>- `grep -r "abort(403)" app/Http/Controllers/` → must return **0 results**<br>- Every authorization through `$this->authorize()` only<br>- Every action button uses `@can` or `@cannot` in views |
| [ ] | T-56 | Debugbar — Detect and fix N+1 | `DEBUG` | High | 1h | **Action:**<br>- Open `/dashboard` → check SQL tab in Debugbar<br>- Expected: ≤ 3 queries (recus + depenses eager load + counts)<br>- If higher: add `.with('depenses').withCount('depenses')` to controller<br>- Open `/depenses` (expense list) → check query count<br>- Confirm total is stable regardless of number of receipts/expenses |
| [ ] | T-t — Full test suite green | `TEST` | High | 1h | **Action:**<br>- `sail artisan test` → **all tests must pass** ✅<br>- Fix any failing tests before proceeding<br>- Expected test count: ≥ 25 tests across all suites (auth, CRUD, policies, extraction, queue, filtering)<br>- `sail artisan test --coverage` (optional) to visualize coverage<br>- Document test count in README |
| [ ] | T-58 | Full flow test — manual verification | `QA` | High | 1.5h | **Scenarios to verify:**<br>- Access `/dashboard` with→ redirect `/login` ✅<br>- Register new user → dashboard shows empty state ✅<br>- Create receipt with valid Darija text → appears with "En attente" status ✅<br>- Run queue worker → status changes to "Traité" + expenses appear ✅<br>- Click receipt → view details (text + expense table with totals) ✅<br>- Navigate to `/depenses` → see all expenses ✅<br>- Filter by "alimentaire" → only food expenses shown ✅<br>- Delete receipt → disappears from dashboard ✅<br>- Login as second uer → cannot access first user's receipts (403) ✅<br>- Submit receipt with AI error → status shows "Échoué" + error logged ✅ |
| [ ] | T-59 | MCD & MLD diagrams | `DOC` | High | 1h | **Create & commit:**<br>- **MCD:** 3 entities (User, Recu, Depense) with attributes (no types, no FK), relationships with cardinalities (1:N)<br>- **MLD:** Tables with column types, PK (underlined), FK (with arrows)<br>- Ensure exact correspondence with migrations<br>- Format: draw.io, Looping, or high-res scanned papee as: `docs/mcd.png` + `docs/mld.png` (or `.pdf`)<br>- Commit: `docs: add MCD and MLD diagrams` |
| [ ] | T-60 | `README.md` complete | `DOC` | High | 1h | **Sections to include:**<br>- **Description:** What Expenso does + business context (Brahim's problem)<br>- **Stack:** Laravel 11, PHP 8.3, MySQL 8.0, Docker Sail, Tailwind CSS v4, Pest, laravel/ai SDK, Groq<br>- **Installation:** Step-by-step (clone → env → sail up → migrate:fresh --seed)<br>- **Test credentials:** `test@example.com` / `rd`<br>- **Running tests:** `sail artisan test` command<br>- **Queue worker:** `sail artisan queue:work --timeout=300`<br>- **Running the app:** `sail up -d` + `sail npm run dev`<br>- **Routes table:** All named routes (GET /dashboard, GET /recus/create, POST /recus, GET /recus/{id}, DELETE /recus/{id}, GET /depenses, etc.)<br>- **Concepts explained:** Jobs & Queues, Eloquent Casts, Structured Output, SoftDeletes, Policies<br>- **Project structure:** `app/Models/`, `app/Http/Controllers/`, `app/Jobs/`, `resources/views/`<br>- **MCD & MLD:** Links or embedded images<br>- **AI Strategy:** Link to `AGENTS.md` |
| [ ] | T-61 | `AGENTS.md` — Complete AI documentation | `DOC` | High | 0.5h | **Sections:**<br>- **Model:** Groq grok-4.3 via laravel/ai SDK<br>- **Prompting:** How prompt is structured (Darija text + JSON schema)<br>- **Structured Contract:** Full JSON schema with examples<br>- **Error Handling:** How failures are logged and marked<br>- **Testing:** Using laravel/ai fake for deterministic tests<br>- *xample Groq Call:** Pseudocode showing the structured() call |
| [ ] | T-62 | Specs folder — 3+ documented features | `SPEC` | High | 0.5h | **Verify specs/ exists with:**<br>- `feature-extract-expenses.md` — Expense extraction<br>- `feature-receipt-management.md` — CRUD operations<br>- `feature-queue-processing.md` — Asynchronous processing<br>- Each spec has: Intent, JSON Contract (if applicable), Acceptance Criteria, Example<br>- All specs committed with clear commit messages<br>- Example: `docs:c for queue-based extraction via OpenSpec` |
| [ ] | T-63 | Jira board + Backlog | `DOC` | High | 0.3h | **Action:**<br>- Verify Jira board is shared with mentor before deadline<br>- Sprint backlog complete with all User Stories (US1 → US8)<br>- Each US has tickets with clear acceptance criteria<br>- Burndown chart visible (shows progress over time)<br>- No tickets created at last minute (history shows daily work) |
| [ ] | T-64 | Git audit — commits & branches | `DOC` | High | 0.3h | **Action:**<br>- `log --oneline` → verify ≥ 15 commits with explicit messages<br>- Example good messages:<br>&nbsp;&nbsp;- `feat(extraction): implement structured AI extraction with laravel/ai`<br>&nbsp;&nbsp;- `feat(queue): add job dispatch for async processing`<br>&nbsp;&nbsp;- `fix(n+1): add eager loading to dashboard query`<br>&nbsp;&nbsp;- `test(depenses): add filtering feature tests`<br>&nbsp;&nbsp;- `docs: add AGENTS.md with AI strategy`<br>&nbsp;&nbsp;- `chore: add OpenSpec feature documentation`<br>- Verify bran exist: `feature/recus-crud`, `feature/extraction-ia`, `feature/queue-processing`<br>- Mention AI usage in commit messages (e.g., "with OpenCode assistance") |
| [ ] | T-65 | Video demo script preparation | `DOC` | High | 1h | **Prepare talking points:**<br>- **Intro (1 min):** Explain the problem (Brahim's receipts) + solution (AI extraction)<br>- **Architecture (2 min):** Diagram: User → Controller → Job → Queue → AI API → Depenses stored<br>- **Live demo (5 min):**<br>&nbsp;&nbsp;1. Register nebr>&nbsp;&nbsp;2. Create receipt with Darija text<br>&nbsp;&nbsp;3. Show "En attente" status (page auto-refreshes)<br>&nbsp;&nbsp;4. Show queue:work running in background<br>&nbsp;&nbsp;5. Refresh → status "Traité" + expenses visible<br>&nbsp;&nbsp;6. Click receipt → see details (text + expense table)<br>&nbsp;&nbsp;7. Navigate to `/depenses` → filter by "Alimentaire"<br>&nbsp;&nbsp;8. Show MCD/MLD slides<br>- **Questions prep:** Be ready to explain Jobs, Casts, Policies, Structured Output |

**Sprinefinition of Done:**

- [ ] All forms have `@csrf` and use Form Request classes
- [ ] All models have `$fillable` defined
- [ ] `@forelse` used on all lists with empty state handled
- [ ] Zero `abort(403)` in controllers — all through `$this->authorize()`
- [ ] N+1 confirmed fixed via Debugbar (≤ 3 queries on dashboard)
- [ ] `sail artisan test` → all tests pass ✅ (≥ 25 tests)
- [ ] README complete with install instructions + credentials + queue:work command
- [ ] AGENTS.md complete with AI strate 3 feature specs in `specs/` folder with OpenSpec format
- [ ] ≥ 15 commits with clear messages mentioning AI assistance
- [ ] Both user flows tested and working (own data vs cross-user 403)
- [ ] Demo script prepared and rehearsed

---

## 📦 Final Deliverables Checklist

| Livrable | Critère | Statut |
|----------|---------|--------|
| GitHub Repo | ≥ 15 commits avec messages explicites | ⬜ |
| GitHub Repo | Feature branches utilisées (`feature/recus-crud`, etc.) | ⬜ |
| GitHub Repo | README aions d'installation complètes | ⬜ |
| GitHub Repo | `AGENTS.md` documenting AI strategy | ⬜ |
| Jira | Board shared with mentor before deadline | ⬜ |
| Jira | Sprint backlog complete with all User Stories (US1 → US8) | ⬜ |
| MCD | Entities, attributes, relationships with cardinalities | ⬜ |
| MLD | Tables, types, PK, FK | ⬜ |
| MCD/MLD | Submitted in `docs/` folder + committed | ⬜ |
| Specs | Minimum 3 features documented with OpenSpec in `specs/` | ⬜ |
| README.md | Installation with Saiocker + test credentials | ⬜ |
| README.md | `sail artisan test` command documented | ⬜ |
| README.md | `sail artisan queue:work` documented | ⬜ |
| Migrations | All tables via migrations (zero SQL manuel) | ⬜ |
| Seeders | Users, receipts, expenses with varied statuses | ⬜ |
| Debugbar | N+1 identified and fixed (≤ 3 queries per route) | ⬜ |
| Tests Pest | `sail artisan test` → ≥ 25 tests pass | ⬜ |
| Tests Pest | Scenarios covered: auth, CRUD, policies, extraction, queue, filtering | âive Demo | Full flow: register → submit → process → filter | ⬜ |

---

## 🏆 Performance Criteria

### Respect du cahier des charges (25%)

| Critère | Statut |
|---------|--------|
| Toutes les User Stories livrées et fonctionnelles (US1 → US8) | ⬜ |
| Routes toutes nommées | ⬜ |
| Toutes les routes protégées par le middleware `auth` | ⬜ |
| Validation via Form Request classes — zéro `$request->validate()` dans les controllers | ⬜ |
| `$fillable` défini sur chaque modèle | â tous les formulaires | ⬜ |
| `@forelse` sur toutes les listes avec cas vide géré | ⬜ |
| Soft Deletes sur `Recu` (optionnel pour cette version) | ⬜ |
| Statuts et catégories affichés en français (via accessors/labels) | ⬜ |
| Zéro N+1 vérifiable en live avec Debugbar | ⬜ |
| MCD et MLD corrects et validés | ⬜ |

### Architecture Laravel & Concepts (30%)

| Critère | Statut |
|---------|--------|
| `RecuPolicy` — `$this->authorize()` dans tous les controllers | ⬜ |
| Accessors `stl`, `categorie_label` utilisés dans les vues | ⬜ |
| Zéro `abort(403)` manuel dans le code | ⬜ |
| Relations Eloquent correctes (`hasMany`, `belongsTo`) avec eager loading | ⬜ |
| Eloquent Casts: `payload_ia` cast as `array`, `statut` as enum | ⬜ |
| Job `ExtractExpensesFromRecu` created, dispatched, processed by queue | ⬜ |
| Structured output via laravel/ai SDK with JSON schema | ⬜ |
| Error handling: failed extractions marked + logged | ⬜ |

### Workflow AI-Assisted (20%)

| Critère | Sttut |
|---------|--------|
| `AGENTS.md` présent et complet | ⬜ |
| Specs folder avec ≥ 3 features documentées via OpenSpec | ⬜ |
| Commits avec mention explicite de l'utilisation d'AI | ⬜ |
| Capacité à expliquer ce que l'agent a généré vs ce qui a été modifié | ⬜ |
| Capacité à expliquer pourquoi Structured Output & Queue existent (décisions architecturales) | ⬜ |

### Tests Pest (15%)

| Critère | Statut |
|---------|--------|
| Tests unitaires sur les models (accessors, relatioable`) | ⬜ |
| Tests feature sur l'authentification | ⬜ |
| Tests feature sur les politiques d'accès (403 cross-user) | ⬜ |
| Tests feature sur le CRUD receipts | ⬜ |
| Tests feature sur la structured extraction (avec fake SDK) | ⬜ |
| Tests feature sur le queue processing | ⬜ |
| `sail artisan test` → ≥ 25 tests pass sans erreur | ⬜ |

### Qualité de la présentation et démonstration (10%)

| Critère | Statut |
|---------|--------|
| Démo live fluide et complète sans bug bloquant | des claires avec MCD/MLD expliqués | ⬜ |
| Discours structuré (contexte, architecture, décisions techniques, difficultés rencontrées) | ⬜ |

---

## 🎤 Live Demo & Q&A Preparation

### Phase 1 — Présentation + Démonstration (10 min)

| Scénario à démontrer | Préparé |
|----------------------|---------|
| Full flow: registration → paste receipt → auto-process → view expenses | ⬜ |
| Queue worker running: show receipt going from "En attente" → "Traité" | ⬜ |
| Expense list: fiter by category ("Alimentaire", "Boissons", etc.) | ⬜ |
| Receipt details: show extracted text + expense table with totals | ⬜ |
| Authorization: try to access another user's receipt as logged-in user → 403 | ⬜ |
| Run `sail artisan test` → show all tests passing live | ⬜ |

### Phase 2 — Code Review & Q&A (20 min)

| Concept | Préparé |
|---------|---------|
| Expliquer comment `RecuPolicy` bloque l'accès cross-user (show code) | ⬜ |
| Expliquer comment `ExtractExpensesJob` processes asyronously | ⬜ |
| Expliquer structured output: pourquoi JSON schema → guaranteed valid data | ⬜ |
| Expliquer pourquoi `StoreRecuRequest` au lieu de `$request->validate()` | ⬜ |
| Montrer un test Pest (extraction or queue) et expliquer | ⬜ |
| Expliquer comment Debugbar a aidé à détecter/corriger N+1 | ⬜ |
| Expliquer l'accessor `statut_label`: raw value → French display | ⬜ |
| Expliquer Eloquent Cast for `payload_ia` as array | ⬜ |

### Phase 3 — Mise en situation (15 min)

| Compétréparé |
|-----------|---------|
| Given a new feature request (e.g., "allow CSV export"), propose a methodical approach | ⬜ |
| Identify the right Laravel layer (Controller / Job / Model / Policy) | ⬜ |
| Write or read a Pest test to validate a behavior | ⬜ |
| Discuss design trade-offs (database queue vs Redis, eager load vs lazy, etc.) | ⬜ |

---

## 📝 Notes for Mentor / Evaluator

**Project Duration:** 5 days (Monday 06/08 → Friday 06/12)

**Key Concepts Demonstrated:**

1. **Spec-Driven D OpenSpec throughout
2. **Structured Output** — Guaranteed JSON validation via laravel/ai
3. **Asynchronous Processing** — Job dispatch + queue:work (no page freeze)
4. **Eloquent Features** — Casts, Accessors, Relationships, Policies
5. **Testing** — Pest with fake SDK, comprehensive test suite
6. **N+1 Prevention** — Debugbar verification, eager loading

**Tech Stack:**

- Laravel 11 + Sail (Docker)
- MySQL 8.0
- Pest (TDD)
- laravel/ai SDK
- Groq API (grok-4.3)
- Tailwind CSS v4
- OpenSpec for sliverables:**
- ≥ 15 commits with AI assistance noted
- ≥ 25 passing tests
- Complete documentation (README + AGENTS.md + Specs)
- MCD + MLD diagrams
- Live demo (5+ min)

---

*Dernière mise à jour : 06/08/2026*
