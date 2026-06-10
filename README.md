# 📊 Expenso

> **AI-Powered Expense Tracking for Neighborhood Merchants**  
> Transform handwritten receipts into structured, categorized expenses in seconds.

<div align="center">

[![Laravel](https://img.shields.io/badge/Laravel-13-FF2D20?style=flat-square&logo=laravel)](https://laravel.com) [![PHP](https://img.shields.io/badge/PHP-8.5`-777BB4?style=flat-square&logo=php)](https://php.net)[![MySQL](https://img.shields.io/badge/MySQL-8.0-4479A1?style=flat-square&logo=mysql)](https://mysql.com)[![Tests](https://img.shields.io/badge/Tests-Passed-brightgreen?style=flat-square)](https://github.com/yourusername/expenso/actions)
</div>

---

## 🎯 The Problem

Brahim runs a small grocery store in the neighborhood. Every month, he accumulates dozens of supplier receipts — many scribbled on paper, amounts written in Darija, abbreviations only he understands. He has no idea how much he actually spends on drinks, cleaning products, or food.

**Manual entry into a spreadsheet?** He has neither the time nor the desire.

## ✨ The Solution

**Expenso** is a Laravel web application that:

1. **Accepts receipt text** (paste Darija, poorly formatted, any language)
2. **Dispatches AI processing** asynchronously (page never freezes)
3. **Extracts structured expenses** using Groq's AI with guaranteed JSON output
4. **Validates & stores** each line item (label, quantity, price, category)
5. **Tracks expenses** over time, filterable by category

**The AI handles the hard work.** Brahim just pastes, and his expenses are automatically organized.

---

## 🏗️ Architecture

```
User (Web) → Browser
    ↓
    ├─→ [Create Receipt] POST /recus
    │        ↓
    │   Validation (StoreRecuRequest)
    │        ↓
    │   Recu::create() → status = "en_attente"
    │        ↓
    │   **Dispatch Job** (ExtractExpensesFromRecu)
    │        ↓
    │   Return Flash: "Traitement en cours..." (no wait)
    │
    └─→ Queue (Database Driver)
         ↓
    **queue:work** (Worker Process)
         ↓
    ExtractExpensesJob::handle()
         ├─→ Call Groq API via laravel/ai
         │   (structured output: JSON schema enforced)
         │
         ├─→ Parse AI response → Create Depense records
         │   (for each article in payload)
         │
         └─→ Update Recu status
             ├─ "traité" ✅ (success)
             └─ "échoué" ❌ (error logged, logged)
```

**Key Insight:** The page returns immediately. The AI processing happens in the background. Users see status updates via auto-refresh.

---

## 📦 Tech Stack

| Layer | Technology |
|-------|-----------|
| **Framework** | Laravel 11.x |
| **Language** | PHP 8.3 |
| **Database** | MySQL 8.0 |
| **Container** | Docker (Laravel Sail) |
| **AI SDK** | `laravel/ai` (via Groq API) |
| **AI Model** | Groq `grok-4.3` |
| **Queuing** | Database Driver (Jobs & Queues) |
| **Frontend** | Blade + Tailwind CSS v4 |
| **Testing** | Pest (TDD) |
| **Authentication** | Laravel Breeze (Blade) |
| **Code Quality** | Laravel Debugbar |

---

## 🚀 Quick Start (5 minutes)

### Prerequisites

- **Docker & Docker Compose** ([Download](https://www.docker.com/products/docker-desktop))
- **Git**
- **Groq API Key** (free tier available at [console.groq.com](https://console.groq.com))

### 1. Clone the Repository

```bash
git clone https://github.com/yourusername/expenso.git
cd expenso
```

### 2. Copy Environment File

```bash
cp .env.example .env
```

### 3. Configure `.env`

Edit `.env` with your settings:

```env
APP_NAME=Expenso
APP_URL=http://localhost

# Database
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=expenso
DB_USERNAME=sail
DB_PASSWORD=password

# Queue (Database driver)
QUEUE_CONNECTION=database

# AI / Groq
AI_API_KEY=your_groq_api_key_here
```

> 🔑 **Get your Groq API Key:**
>
> 1. Visit [console.groq.com](https://console.groq.com)
> 2. Sign up (free)
> 3. Create an API key
> 4. Paste it into `.env`

### 4. Start Docker Containers

```bash
./vendor/bin/sail up -d
```

> **First time?** Docker will build the image (~2-3 min). Subsequent runs are instant.

### 5. Run Migrations & Seed Database

```bash
sail artisan migrate:fresh --seed
```

This creates tables and populates with sample data:

- 2 test users
- 5 sample receipts with varied statuses
- 15 sample expenses across categories

### 6. Start the Queue Worker

Open a **new terminal** (keep the first one running):

```bash
sail artisan queue:work --timeout=300
```

This worker processes jobs in the background. You'll see output like:

```
[2026-06-10 14:32:15] Processing: App\Jobs\ExtractExpensesFromRecu
[2026-06-10 14:32:18] Processed:  App\Jobs\ExtractExpensesFromRecu
```

### 7. Start Asset Compilation

Open another **new terminal**:

```bash
sail npm run dev
```

This watches Tailwind CSS and compiles assets in real-time.

### 8. Visit the App

Open your browser and go to:

```
http://localhost
```

You should see the Expenso welcome page. 🎉

---

## 📚 Installation (Full Guide)

### Step 1: Clone & Setup Project Structure

```bash
git clone https://github.com/yourusername/expenso.git
cd expenso
```

### Step 2: Install Composer Dependencies

If you haven't already:

```bash
composer install
```

### Step 3: Install Laravel Sail

```bash
composer require laravel/sail --dev
php artisan sail:install
# When prompted: select "mysql"
```

### Step 4: Generate Application Key

```bash
sail artisan key:generate
```

### Step 5: Configure Environment

```bash
cp .env.example .env
```

Edit `.env`:

- `APP_NAME=Expenso`
- `APP_URL=http://localhost`
- `DB_HOST=mysql` (important for Docker)
- `DB_DATABASE=expenso`
- `DB_USERNAME=sail`
- `DB_PASSWORD=password`
- `QUEUE_CONNECTION=database`
- `AI_API_KEY=<your-groq-key>`

### Step 6: Start Containers

```bash
sail up -d
```

Verify all services running:

```bash
sail ps
```

You should see:

- `laravel.test` (PHP/Laravel)
- `mysql` (Database)
- `phpmyadmin` (Database UI)

### Step 7: Run Database Migrations

```bash
sail artisan migrate:fresh --seed
```

Check the database:

- **phpMyAdmin:** <http://localhost:8081>
  - Username: `sail`
  - Password: `password`

### Step 8: Install NPM Dependencies

```bash
sail npm install
sail npm run build
```

For development with hot reload:

```bash
sail npm run dev
```

### Step 9: Verify Installation

Run the test suite:

```bash
sail artisan test
```

Expected output:

```
   PASS  Tests/Feature/AuthTest
   PASS  Tests/Feature/RecuCrudTest
   PASS  Tests/Feature/RecuPolicyTest
   ...
   Tests:  25 passed
```

---

## 🔐 Test Credentials

After running `migrate:fresh --seed`, login with:

| Email | Password | Role |
|-------|----------|------|
| `test@example.com` | `password` | User |
| `other@example.com` | `password` | User |

Both accounts have pre-populated receipts and expenses for testing.

---

## 📖 Usage Guide

### 1. Register or Login

Visit `http://localhost` and create a new account, or use test credentials above.

### 2. Submit a Receipt

1. Click **"Create Receipt"** (or navigate to `/recus/create`)
2. Paste your receipt text (Darija, Arabic, French, or English)
   - Example:

     ```
     لبن 2 5.5
     خبز 3 2
     صابون 1 10
     ```

3. Click **"Submit"**
4. You'll see: **"Reçu reçu. Traitement en cours..."** (immediately)

### 3. Monitor Processing

Back on the dashboard (`/dashboard`):

- Your receipt shows status: **"En attente"** (yellow badge)
- Page auto-refreshes every 3 seconds
- Queue worker (in your terminal) processes it in the background

Once processed:

- Status changes to **"Traité"** (green badge)
- Number of extracted expenses appears
- Click **"View"** to see details

### 4. View Receipt Details

Click a receipt to see:

- **Original text** (what was submitted)
- **Processing status**
- **Extracted expenses table:**

  | Libellé | Quantité | Prix Unitaire | Catégorie |
  |---------|----------|---|---|
  | Lait | 2 | 5.50 | Alimentaire |
  | Savon | 1 | 10.00 | Hygiène |

- **Total amount** (sum of quantity × price)

### 5. Filter Expenses

Click **"Expenses"** in the navbar (or visit `/depenses`):

- View all your extracted expenses
- Filter by category:
  - 🥗 **Alimentaire** (Food)
  - 🥤 **Boissons** (Drinks)
  - 🧼 **Hygiène** (Hygiene)
  - 🧹 **Entretien** (Maintenance)
  - ❓ **Autre** (Other)

---

## 🧪 Testing

### Run All Tests

```bash
sail artisan test
```

### Run Specific Test Suite

```bash
# Unit tests (models)
sail artisan test --path=tests/Unit

# Feature tests (CRUD, auth, policies)
sail artisan test --path=tests/Feature

# Extraction tests (AI + queue)
sail artisan test --filter=Extraction
```

### View Test Coverage

```bash
sail artisan test --coverage
```

### Key Test Scenarios

✅ **Authentication Tests**

- Registration with valid/invalid data
- Login/logout functionality
- Protected route access

✅ **CRUD Tests**

- Create receipt with valid text
- View receipt details
- Delete receipt (soft delete)
- Cross-user access denied (403)

✅ **AI Extraction Tests**

- Structured output validated
- Depense records created correctly
- Job dispatch & queue processing
- Error handling & status updates

✅ **Policy Tests**

- Users can only access their own receipts
- Authorization via `$this->authorize()`
- Proper 403 responses

---

## 🗄️ Database Schema

### Users Table

```sql
id, name, email, email_verified_at, password, 
remember_token, timestamps
```

### Recus Table

```sql
id, user_id (FK), texte_brut (longText), 
statut (enum: en_attente | traité | échoué),
payload_ia (json - raw AI response),
deleted_at, timestamps
```

### Depenses Table

```sql
id, recu_id (FK), libellé (string), 
quantité (integer), prix_unitaire (decimal),
catégorie (enum: alimentaire | boissons | hygiène | entretien | autre),
timestamps
```

### Database Diagram (MCD/MLD)

**Conceptual Model:**

```
User (1) ──────┐
               │ hasMany
               └──→ Recu (N)
                     │
                     │ hasMany
                     └──→ Depense (N)
```

**Logical Model:**

```
users
├─ id (PK)
├─ name
├─ email (UNIQUE)
└─ password

recus
├─ id (PK)
├─ user_id (FK → users.id)
├─ texte_brut (longText)
├─ statut (enum)
├─ payload_ia (json)
└─ deleted_at (nullable)

depenses
├─ id (PK)
├─ recu_id (FK → recus.id)
├─ libellé (string)
├─ quantité (int)
├─ prix_unitaire (decimal)
└─ catégorie (enum)
```

---

## 🌐 API Routes

All routes are named and protected by `auth` middleware (except guest routes).

| Method | Route | Name | Purpose |
|--------|-------|------|---------|
| `GET` | `/` | `home` | Welcome page |
| `GET\|POST` | `/register` | `register` | User registration |
| `GET\|POST` | `/login` | `login` | User login |
| `POST` | `/logout` | `logout` | User logout |
| **Authenticated Routes** |
| `GET` | `/dashboard` | `recus.index` | List user's receipts |
| `GET` | `/recus/create` | `recus.create` | Receipt form |
| `POST` | `/recus` | `recus.store` | Create receipt (dispatch job) |
| `GET` | `/recus/{recu}` | `recus.show` | View receipt details |
| `DELETE` | `/recus/{recu}` | `recus.destroy` | Delete receipt (soft) |
| `GET` | `/depenses` | `depenses.index` | List all expenses (filterable) |

---

## 🤖 AI & Structured Output

### How It Works

1. **User submits receipt text**

   ```
   لبن 2 5.5
   خبز 3 2
   صابون 1 10
   ```

2. **Job dispatches to queue**

   ```php
   ExtractExpensesFromRecu::dispatch($recu);
   ```

3. **Worker processes job**
   - Calls Groq API via `laravel/ai` SDK
   - Sends prompt + JSON schema

4. **Groq returns structured JSON**

   ```json
   {
     "articles": [
       {
         "libellé": "Lait",
         "quantité": 2,
         "prix_unitaire": 5.5,
         "catégorie": "alimentaire"
       },
       {
         "libellé": "Pain",
         "quantité": 3,
         "prix_unitaire": 2,
         "catégorie": "alimentaire"
       },
       {
         "libellé": "Savon",
         "quantité": 1,
         "prix_unitaire": 10,
         "catégorie": "hygiène"
       }
     ],
     "total_estimé": 21,
     "devise": "MAD"
   }
   ```

5. **Depense records created** (one per article)
   - Labels, quantities, prices, categories stored
   - Receipt status → "traité"

6. **User sees results** (auto-refresh dashboard)

### JSON Contract

The AI output **must** match this schema (enforced by `AI::structured()`):

```json
{
  "articles": [
    {
      "libellé": "string (max 255 chars)",
      "quantité": "positive integer",
      "prix_unitaire": "positive number (decimal)",
      "catégorie": "enum: alimentaire | boissons | hygiène | entretien | autre"
    }
  ],
  "total_estimé": "positive number",
  "devise": "string (e.g., 'MAD', 'EUR')"
}
```

### Error Handling

If extraction fails:

1. **Exception caught** in Job's `handle()`
2. **Error logged** to `storage/logs/laravel.log`
3. **Receipt status** set to `"échoué"` (red badge)
4. **Error details** stored in `payload_ia` for debugging
5. **User sees failure** on dashboard and can retry

---

## 🔧 Development

### Key Concepts

#### 1. **Eloquent Relationships**

```php
// Recu.php
public function user(): BelongsTo {
    return $this->belongsTo(User::class);
}

public function depenses(): HasMany {
    return $this->hasMany(Depense::class);
}

// Depense.php
public function recu(): BelongsTo {
    return $this->belongsTo(Recu::class);
}
```

#### 2. **Accessors (French Labels)**

```php
// Recu.php
public function getStatutLabelAttribute(): string {
    return match($this->statut) {
        'en_attente' => 'En attente',
        'traité' => 'Traité',
        'échoué' => 'Échoué',
    };
}

// Depense.php
public function getCategorieLabelAttribute(): string {
    return match($this->catégorie) {
        'alimentaire' => 'Alimentaire',
        'boissons' => 'Boissons',
        'hygiène' => 'Hygiène',
        'entretien' => 'Entretien',
        'autre' => 'Autre',
    };
}
```

#### 3. **Policies (Authorization)**

```php
// RecuPolicy.php
public function view(User $user, Recu $recu): bool {
    return $recu->user_id === $user->id; // Own receipts only
}

// In Controller:
$this->authorize('view', $recu);  // Throws 403 if denied
```

#### 4. **Jobs & Queues**

```php
// Dispatch from controller:
ExtractExpensesFromRecu::dispatch($recu);

// Job processes asynchronously:
class ExtractExpensesFromRecu implements ShouldQueue {
    public function handle() {
        $payload = AI::structured(...)->json();
        // Create Depense records from payload
        // Update Recu status to 'traité'
    }
}
```

#### 5. **Form Requests**

```php
// Validation before controller logic:
class StoreRecuRequest extends FormRequest {
    public function rules(): array {
        return [
            'texte_brut' => 'required|string|min:10|max:5000',
        ];
    }
}

// In Controller:
public function store(StoreRecuRequest $request) {
    $recu = Recu::create($request->validated());
}
```

#### 6. **N+1 Prevention**

```php
// ❌ Bad (N+1 queries):
$recus = $user->recus()->get();
foreach ($recus as $recu) {
    $count = $recu->depenses()->count(); // +1 query per receipt
}

// ✅ Good (eager loading):
$recus = $user->recus()->withCount('depenses')->get();
// Only 2 queries total
```

---

## 📊 Debugbar & Performance

### Enable Debugbar

Already enabled in `app.php` for local development.

Open any page and look for the **Debugbar panel** at the bottom right.

### Check for N+1

1. Open `/dashboard`
2. Click **Debugbar** → **SQL** tab
3. Count queries:
   - **Expected:** ≤ 3 (recus + depenses + counts)
   - **Bad:** 10+ (N+1 detected)

4. If N+1 found, add eager loading:

   ```php
   $recus = auth()->user()->recus()
       ->with('depenses')
       ->withCount('depenses')
       ->get();
   ```

---

## 🚨 Troubleshooting

### Docker Issues

**Container won't start?**

```bash
sail down
sail up -d
```

**Need to rebuild?**

```bash
sail build --no-cache
sail up -d
```

### Database Issues

**Migration failed?**

```bash
sail artisan migrate:rollback
sail artisan migrate:fresh --seed
```

**Check with phpMyAdmin:**

- <http://localhost:8081>
- User: `sail` | Password: `password`

### Queue Issues

**Jobs not processing?**

```bash
# In separate terminal, ensure worker is running:
sail artisan queue:work --timeout=300

# Or check failed jobs:
sail artisan queue:failed
```

**Queue table doesn't exist?**

```bash
sail artisan queue:table
sail artisan migrate
```

### AI/Groq Issues

**"Invalid API Key" error?**

1. Get new key from [console.groq.com](https://console.groq.com)
2. Update `.env`: `AI_API_KEY=...`
3. Restart job worker: Ctrl+C, then `sail artisan queue:work`

**Structured output not matching schema?**

- Check Groq response in logs: `storage/logs/laravel.log`
- Verify JSON contract in `AGENTS.md`

---

## 📖 Additional Documentation

- **[AGENTS.md](./AGENTS.md)** — AI strategy, prompting, structured output details
- **[taskboard.md](./taskboard.md)** — SCRUM sprints, task breakdown, autoformation curriculum
- **[specs/](./specs/)** — Feature specifications (OpenSpec format)
  - `feature-extract-expenses.md`
  - `feature-receipt-management.md`
  - `feature-queue-processing.md`

---

## 🤝 Contributing

Contributions welcome! Please:

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Write tests for your changes
4. Commit with clear messages: `feat(module): description`
5. Push and open a Pull Request

---

## 📝 Commits & AI Usage

This project uses **Spec-Driven Development** and AI assistance (via OpenCode, OpenSpec, and Groq).

Commit messages follow this format:

```
type(scope): subject

Examples:
feat(extraction): implement AI-powered structured extraction with job dispatch
feat(queue): add asynchronous processing via database queue
fix(n+1): add eager loading to dashboard query
test(depenses): add filtering feature tests
docs: add AGENTS.md with AI strategy
chore: initial Laravel + Sail setup
```

When AI generated significant code, mention it:

```
feat(job): implement ExtractExpensesJob with OpenCode assistance
```

---

## 🎯 Performance Criteria

✅ **Architectural Decisions**

- Asynchronous processing (no page freeze)
- Structured output with JSON schema validation
- Authorization via Policies (no `abort(403)`)
- Eager loading (zero N+1)

✅ **Code Quality**

- ≥ 25 passing Pest tests
- All models have `$fillable` defined
- All forms have `@csrf` + Form Requests
- All lists use `@forelse` with empty state

✅ **Features**

- Full CRUD (Create, Read, Delete)
- Status tracking (Pending → Processed → Failed)
- Expense filtering by category
- Cross-user access denied (403)

---

## 📞 Support

- **Questions?** Open an issue on GitHub
- **Bug report?** Create a detailed issue with steps to reproduce
- **Feature request?** Submit a feature request issue with use case

---

## 🙌 Acknowledgments

- **Groq** — Fast, free LLM API
- **Laravel** — Web framework
- **Laravel Sail** — Docker development environment
- **Pest** — Modern testing framework
- **OpenSpec** — Spec-Driven Development methodology

---

<div align="center">

**Made with ❤️ by Ayouub-aj**

🚀 [View on GitHub](https://github.com/yourusername/expenso) | 📧 [Email](mailto:your.email@example.com) | 🌍 [Portfolio](https://yourportfolio.com)

</div>
