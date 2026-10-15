# Mobily Phone Shop POS

A point-of-sale and back-office system built for a mobile phone shop in Lebanon. Runs locally on a Windows PC, opened in a browser — no internet needed after setup. Supports Arabic and English with full RTL layout.

---

## What it does

**Point of Sale**
Tap a category or type a product name to add items to the cart. The system checks stock in real time and shows how many units are still available as you add things. Pay in USD or LBP with automatic change calculation. Add a warranty note if needed, then hit Complete Sale — the receipt opens in a new window ready to print.

**Receipts**
Designed for 80 mm thermal printers. Shows the shop logo, name, address and phone, a clean item table, totals in both currencies, paid amount, change, and warranty info if you entered it. Invoice pages are protected with a secret token so you can't just guess invoice numbers.

**Inventory**
Add products with names in Arabic and English, barcode, category, wholesale and retail prices, stock count, and phone specs (storage/RAM). Attach a product photo if you want. When stock arrives, record it in Products Received and the quantity updates automatically. The Stock Overview shows what's running low.

**Debts**
Track what customers owe the shop and what the shop owes dealers. Record payments, see the full history, and the system prevents overpayments.

**Cash Drawer**
Pick any day, week or month and see exactly how much cash should be in the drawer — opening balance plus sales plus collected debts minus payments to dealers minus expenses. Works in USD and LBP separately.

**Reports**
Revenue, cost, gross profit and items sold. Sales and purchases tables with date filters. Close Day shows a summary of the day's sales by employee, stock received and top products. New Day resets the daily view without touching any data.

**Repairs**
Log repair jobs with customer name, phone model, problem description, PIN, cost and deposit. Print a receipt for the customer with both a customer copy and a shop copy. The system tracks profit per repair job automatically.

**Users**
Three roles: Admin (full access including system reset), Owner (everything except admin tools), and Employee (POS only — never sees cost prices, reports or drawer). Employees with sales history stay in the system when deleted so old invoices still show their name.

---

## Roles at a glance

| Role | What they can do |
|---|---|
| Admin | Everything. Hidden login — press **Ctrl+Shift+A** on the login page |
| Owner | Products, inventory, reports, debts, drawer, repairs, users, settings. Sees cost prices |
| Employee | Point of sale only. No cost prices, no reports, no drawer, no user management |

---

## Security

Passwords are hashed and salted. Minimum 8 characters, no common passwords, can't contain your username. After 5 wrong attempts the account is locked for 5 minutes. Sessions live in the browser tab only — closing the browser logs you out. The server validates the session cookie on every page load, so pressing Back after logout takes you back to the login screen, not the old dashboard. Pressing Forward after going Back also requires a fresh login. Tokens are revoked server-side on logout. Auto sign-out after 30 minutes of inactivity.

The API uses Authorization headers, not cookies, so cross-site request forgery isn't a concern. Cost prices are never sent to employee accounts. The checkout runs in an exclusive database transaction so two sales can't both take the last unit in stock.

---

## Tech stack

- **Backend:** Python 3, Flask, Waitress (production WSGI server)
- **Database:** SQLite in WAL mode
- **Auth:** PyJWT + Werkzeug password hashing
- **Frontend:** Plain HTML, CSS, JavaScript — no build step, no frameworks
- **Windows service:** NSSM (runs at boot without a logged-in user)

Everything is served by the shop PC. Fonts load from Google Fonts, everything else is local. The server listens on all interfaces at port 5000 so other devices on the same network can reach it.

---

## Installation

Requirements: Windows 10 or 11, Python 3.11+, internet for first install only.

1. Copy the project folder to `C:\MOBILY POS`
2. Run **INSTALL.bat** — installs packages and starts the app
3. Set it up as a Windows service so it starts at boot:
   ```
   nssm install MobilyPOS
   ```
   Set Path to your `python.exe`, Startup directory to `C:\MOBILY POS\mobily-pos`, Arguments to `app.py`. Then `nssm start MobilyPOS`.
4. Open Edge at `http://localhost:5000`. For a nicer shortcut, target: `msedge.exe --app=http://localhost:5000`
5. Change all default passwords before handing it over

**Receipt printer:** in Edge's print dialog, pick the thermal printer, set paper to 80 mm, margins to none.

**To update:** stop the service, copy new files over the old ones (don't replace `mobily_pos.db`), start the service again.

---

## Daily use

- **Making a sale:** category or search → add items → choose USD or LBP → enter amount paid → Complete Sale
- **Receiving stock:** Inventory → Record Purchase
- **Debt payment:** Debts → Customers or Dealers → Record Payment
- **Repair job:** Repairs → New Repair → fill in the details → print receipt
- **End of day:** Reports → Close Day, then New Day for tomorrow
- **Admin login:** Ctrl+Shift+A on the login page

---

## Backups

A copy of the database is saved every day to `mobily-pos\backups`. The last 30 days are kept. Copy that folder to a USB drive regularly — if the PC dies, that's your restore point.

**Reset System** (admin → Settings) wipes all products, sales, invoices, debts, drawer history and photos, and puts the default categories back. User accounts and settings stay. A full backup is saved automatically before the reset runs, so you can undo it by restoring that file.

---

## Development

```bash
cd mobily-pos
pip install -r requirements.txt
python app.py
```

Server starts at `http://localhost:5000`. Use `PORT=xxxx` to change the port. A fresh database is created automatically on first run with default accounts — change those passwords before real use.

---

## Project layout

```
.
├── INSTALL.bat
├── START.bat
├── README.md
└── mobily-pos/
    ├── app.py                Flask backend
    ├── requirements.txt
    ├── mobily_pos.db         Created on first run (not in repo)
    ├── backups/              Daily DB copies (not in repo)
    ├── static/img/           Logos and product photos
    └── templates/
        ├── login.html
        ├── dashboard.html    Main app
        ├── invoice.html      80 mm receipt
        └── repair.html       Repair job receipt
```

The database, backups and product photos are never committed to the repository.
