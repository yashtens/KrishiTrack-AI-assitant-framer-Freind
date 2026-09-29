# 🌾 KrishiTrack — Smart Farm Management System

A full-stack farm management web application built with **Python Flask** and **MySQL**.
Designed for small farmers to manage daily activities from seeding to harvesting.

---

## 📸 Features

| Module | What it does |
|--------|-------------|
| **Dashboard** | Stats overview, monthly expense chart, crop profit chart, upcoming harvests |
| **Crop Management** | Add/edit/delete crops with image upload, status tracking |
| **Expense Tracking** | Record seed, fertilizer, equipment, labour & other costs — auto total |
| **Labour Management** | Track workers, days worked, payment per day — auto total payment |
| **Harvest Records** | Log production quantity, selling price — auto income calculation |
| **Profit Summary** | Crop-wise P&L, bar chart, return percentage |
| **Reports & CSV** | Export crops, expenses, labour, harvest data to CSV |

---

## 🗂️ Project Structure

```
krishitrack/
├── app.py              ← Flask routes & application factory
├── models.py           ← SQLAlchemy models (Crop, Expense, Labour, Harvest)
├── extensions.py       ← db = SQLAlchemy() (avoids circular imports)
├── config.py           ← All config (DB URL, secret key, upload folder)
├── schema.sql          ← MySQL schema + sample data (run in MySQL Workbench)
├── requirements.txt    ← Python dependencies
├── .env.template       ← Rename to .env and fill your credentials
├── uploads/            ← Crop images stored here
└── templates/
    ├── base.html       ← Master layout with sidebar, topbar, flash messages
    ├── login.html      ← Login page
    ├── dashboard.html  ← Main dashboard with charts
    ├── crops.html      ← Crop list (card grid)
    ├── crop_form.html  ← Add/Edit crop form
    ├── crop_detail.html← Detailed crop view with all related data
    ├── expenses.html   ← Expense list
    ├── expense_form.html← Add/Edit expense (auto total)
    ├── labour.html     ← Labour list
    ├── labour_form.html← Add/Edit labour (auto total pay)
    ├── harvest.html    ← Harvest list
    ├── harvest_form.html← Add/Edit harvest (auto income)
    ├── profit.html     ← P&L summary with bar chart
    └── reports.html    ← Export page
```

---

## ⚙️ Setup Instructions

### Step 1: MySQL Setup
1. Open **MySQL Workbench**
2. Open and run `schema.sql` — this creates the database + all tables + sample data

### Step 2: Python Environment
```bash
# Create and activate virtual environment
python -m venv venv
venv\Scripts\activate        # Windows
# source venv/bin/activate   # Linux/Mac

# Install dependencies
pip install -r requirements.txt
```

### Step 3: Configure Environment
```bash
# Run these commands from the Root/ project directory
# Windows PowerShell:
Copy-Item .env.template .env
# macOS/Linux:
cp .env.template .env
```

Edit `Root/.env` and set your PostgreSQL connection details and AI settings:

```env
DB_USER=postgres
DB_PASSWORD=your_postgres_password
DB_HOST=localhost
DB_PORT=5432
DB_NAME=krishitrack
OPENAI_API_KEY=your_openai_api_key
OPENAI_MODEL=gpt-4o-mini
```

Put the real API key in `.env`, not `.env.template`. Keep `.env` private and never commit it. Restart Flask after changing environment settings so the app loads the new key.

### Step 4: Run the App
```bash
flask run
# OR
python app.py
```

Open browser: `http://localhost:5000`

## Krishi AI Assistant

The assistant is integrated into the existing Flask app at `/assistant`; it uses the existing login session and PostgreSQL models. It reads only crops with an `owner_id` matching the authenticated user. Available records are crops, expenses, labour, harvests, and crop growth photos. The schema has no dated irrigation, soil-test, or pesticide-application tables, so those details are reported as unavailable rather than inferred.

1. Back up PostgreSQL, then run `migrations/20260928_krishi_ai_assistant.sql` against the configured database. The migration adds nullable crop ownership and private conversation tables. Do not assign existing crops to a user unless ownership has been independently verified; the assistant deliberately excludes unassigned legacy crops.
2. Install requirements with `pip install -r requirements.txt` and set `OPENAI_API_KEY` in `Root/.env`. `OPENAI_MODEL` defaults to `gpt-4o-mini`; `CHATBOT_CHROMA_PATH` defaults to `Root/instance/chroma`.
3. Start the app as before with `python app.py` and open `/assistant`. The first semantic search downloads the `all-MiniLM-L6-v2` embedding model. Exact totals and recorded history questions use PostgreSQL and do not require OpenAI.

The vector synchronization endpoint is `POST /api/embeddings/sync` and requires an active admin session. It can sync all owned farmers' records, or one verified farmer with `{"farmer_id": 123}`. Chat endpoints are session-authenticated and scope conversations, SQL records, and Chroma queries to the logged-in user's ID.

Run the focused privacy and answer tests with `python -m unittest discover -s tests -v` from `Root/`.

**Default Login:** `admin` / `farm@1234`

---

## 🗃️ Database Tables

```
crops          → id, name, variety, field_area, seeding_date, expected_harvest,
                  fertilizer_details, water_schedule, status, image_path, notes
expenses       → id, crop_id, date, seeds_cost, fertilizer_cost, equipment_cost,
                  labour_cost, other_expenses, notes
labours        → id, crop_id, name, work_type, days_worked, payment_per_day, date, notes
harvests       → id, crop_id, harvest_date, total_production, unit,
                  selling_price, total_income, notes
```

All monetary totals (`expense.total`, `labour.total_payment`, `harvest.total_income`)
are computed as Python `@property` — no redundant storage needed.

---

## 🌐 Route Map

| URL | Method | Description |
|-----|--------|-------------|
| `/` or `/login` | GET/POST | Login page |
| `/logout` | GET | Clear session |
| `/dashboard` | GET | Main dashboard |
| `/crops` | GET | List all crops |
| `/crops/add` | GET/POST | Add new crop |
| `/crops/<id>` | GET | Crop detail |
| `/crops/<id>/edit` | GET/POST | Edit crop |
| `/crops/<id>/delete` | POST | Delete crop |
| `/expenses` | GET | Expense list |
| `/expenses/add` | GET/POST | Add expense |
| `/expenses/<id>/edit` | GET/POST | Edit expense |
| `/expenses/<id>/delete` | POST | Delete expense |
| `/labour` | GET | Labour list |
| `/labour/add` | GET/POST | Add labour |
| `/labour/<id>/edit` | GET/POST | Edit labour |
| `/harvest` | GET | Harvest list |
| `/harvest/add` | GET/POST | Record harvest |
| `/profit` | GET | P&L summary |
| `/reports` | GET | Reports page |
| `/reports/export/crops` | GET | Download crops CSV |
| `/reports/export/expenses` | GET | Download expenses CSV |
| `/reports/export/labour` | GET | Download labour CSV |
| `/reports/export/harvest` | GET | Download harvest CSV |

---

## 🛠️ Tech Stack

- **Backend:** Python 3.10+ / Flask 3.0
- **Database:** MySQL 8.x via PyMySQL driver
- **ORM:** Flask-SQLAlchemy 3.1
- **Frontend:** HTML5, Bootstrap 5.3
- **Charts:** Chart.js 4.4
- **Fonts:** Lora (headings) + Nunito (body) from Google Fonts
- **Icons:** Bootstrap Icons

---

## 🔒 Security Notes

1. Change `ADMIN_PASSWORD` in `.env` before going live
2. Change `SECRET_KEY` to a long random string
3. Never commit `.env` to git — add it to `.gitignore`
4. For production, use Gunicorn + Nginx instead of Flask dev server

---

## 🚀 Optional Enhancements (Portfolio Ideas)

- [ ] Add AI crop yield prediction using scikit-learn
- [ ] Weather API integration (OpenWeatherMap)
- [ ] SMS alerts for harvest reminders via Twilio
- [ ] Multi-user support with Flask-Login + roles
- [ ] Deploy on Railway / Render / AWS EC2

---

*Built with ❤️ for farmers — KrishiTrack © 2024*
