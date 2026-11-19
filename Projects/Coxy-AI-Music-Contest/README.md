# Coxy Wallet AI Music Contest

A web app for running a community-voted **AI music contest** paid for with **Cardano (ADA)**. Entrants connect a Cardano wallet, pay a 10 ADA entry fee, and upload an AI-generated track. The community listens and votes (one vote per wallet per track), and the track with the most votes wins the round.

Payments are verified **server-side on the blockchain** through [Blockfrost](https://blockfrost.io), so the backend never trusts the browser's claim that a payment was made.

---

## Table of Contents

- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [How It Works](#how-it-works)
- [Getting Started](#getting-started)
- [Configuration Reference](#configuration-reference)
- [API Reference](#api-reference)
- [Database Schema](#database-schema)
- [Ending a Round & Sending Results](#ending-a-round--sending-results)
- [Securing Your Deployment](#securing-your-deployment)
- [Testing Checklist](#testing-checklist)
- [Known Limitations](#known-limitations)

---

## Features

- **Cardano wallet connection** via CIP-30 browser wallets (Nami, Eternl, Flint, etc.) using the Mesh SDK
- **10 ADA entry fee**, paid directly from the user's wallet and confirmed on-chain
- **Audio upload** (MP3/WAV, up to 25 MB) with the AI tool used recorded for each track
- **Live entries list** ranked by votes, with in-page audio playback
- **One vote per wallet per track**, enforced in code and by a database constraint
- **Email notifications**: entry confirmation and end-of-round results (via SMTP / PHPMailer)
- **Round-based contests**: only one round is `active` at a time
- **No database server needed**: SQLite is a single file

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Vanilla HTML, CSS and JavaScript (ES modules, no build step) |
| Wallet / transactions | [Mesh SDK](https://meshjs.dev) (`@meshsdk/core`), loaded from the esm.sh CDN |
| Backend | PHP (PDO) |
| Database | SQLite |
| Payment verification | [Blockfrost](https://blockfrost.io) API |
| Email | [PHPMailer](https://github.com/PHPMailer/PHPMailer) over SMTP |

## Project Structure

```
coxy-wallet-contest/
├── index.html                      # Single-page frontend
├── style.css
├── app.js                          # Wallet connect, payment flow, entries, voting
├── fill-in-these-values.txt        # Quick checklist of the values you must supply
├── backend-full-setup-guide.txt    # Long-form backend setup guide
└── backend/
    ├── schema.sql                  # SQLite schema + one sample active round
    ├── config/
    │   ├── config.example.php      # Copy to config.php and fill in
    │   └── database.php            # PDO connection + helper functions
    ├── includes/
    │   ├── blockfrost.php          # On-chain payment verification
    │   └── mailer.php              # PHPMailer email helpers
    ├── api/
    │   ├── create_payment.php
    │   ├── verify_payment.php
    │   ├── upload_entry.php
    │   ├── entries.php
    │   ├── vote.php
    │   └── send_results.php        # CLI/cron script
    ├── data/                       # contest.db lives here (created by you)
    ├── uploads/audio/              # Uploaded tracks (created automatically)
    └── vendor/                     # Composer dependencies (created by Composer)
```

## How It Works

```
 Browser (app.js + Mesh SDK)                    PHP backend                 Blockfrost
 ───────────────────────────                    ───────────                 ──────────
 1. Connect CIP-30 wallet
 2. "Enter with 10 ADA"  ───────────────►  create_payment.php
                                            • find/create user
                                            • open 'pending' transaction
                         ◄───────────────  pay-to address + amount
 3. Build, sign & submit tx in wallet
    → receives tx hash
 4. Send tx hash         ───────────────►  verify_payment.php  ───────►  Did this tx pay
    (polled every 6 s,                      • mark 'confirmed' only      the right amount
     up to 10 attempts)                       if chain agrees   ◄─────── to the right address?
 5. Entry form unlocks
 6. Upload track         ───────────────►  upload_entry.php
                                            • payment confirmed & unused?
                                            • validate size + MIME type
                                            • save file, create entry
                                            • send confirmation email
 7. Listen & vote        ───────────────►  entries.php / vote.php
```

## Getting Started

### Prerequisites

- PHP 8+ with the `pdo_sqlite` and `curl` extensions (check with `php -m | grep -i -E "sqlite|curl"`)
- [Composer](https://getcomposer.org/)
- `sqlite3` CLI (or a GUI such as [DB Browser for SQLite](https://sqlitebrowser.org))
- A web server (Apache or Nginx) that can run PHP
- A free [Blockfrost](https://blockfrost.io) account
- An SMTP account for sending email
- A Cardano wallet browser extension (Nami, Eternl, Flint, ...) for testing

### 1. Create the database

```bash
mkdir -p backend/data
sqlite3 backend/data/contest.db < backend/schema.sql
```

This creates the five tables and inserts a sample **active** round ("Round 1", 7 days long). A round must be `active` for payments and entries to be accepted. Update it to match your real contest:

```sql
UPDATE contests
SET title = 'Your Contest Name', end_date = '2026-09-30 23:59:59'
WHERE id = 1;
```

Make the database writable by your web server user:

```bash
chmod 664 backend/data/contest.db
chmod 775 backend/data
chown -R www-data:www-data backend/data    # Linux, Apache/Nginx
```

### 2. Configure the backend

```bash
cp backend/config/config.example.php backend/config/config.php
```

Fill in `backend/config/config.php` (see the [configuration reference](#configuration-reference) below). `fill-in-these-values.txt` is a handy worksheet for this.

### 3. Install PHP dependencies

```bash
cd backend
composer require phpmailer/phpmailer
```

Without PHPMailer the app still runs, but emails are skipped and an error is written to the PHP error log.

### 4. Set up Blockfrost

1. Create a free account at [blockfrost.io](https://blockfrost.io)
2. Create a project on **Mainnet** (real ADA) or **Preprod** (free test network)
3. Copy the **Project ID** into `blockfrost_api_key`
4. Set `blockfrost_base_url` to match the network:
   - Mainnet: `https://cardano-mainnet.blockfrost.io/api/v0`
   - Preprod: `https://cardano-preprod.blockfrost.io/api/v0`

> 💡 **Test on Preprod first.** Get free test ADA from the [Cardano testnet faucet](https://docs.cardano.org/cardano-testnet/tools/faucet/).

### 5. Prepare the uploads folder

```bash
chmod -R 755 backend/uploads
chown -R www-data:www-data backend/uploads
```

Uploaded files must be reachable over HTTP so the `<audio>` players work. If `backend/` is under your web root they are served at `/backend/uploads/audio/<file>`.

### 6. Point the frontend at the backend

In `app.js`, check the API path matches where you deployed `backend/`:

```js
const API_BASE = "/backend/api";
```

### 7. Run it

Serve the project root with PHP's built-in server for a quick local check:

```bash
php -S localhost:8000
```

Then open <http://localhost:8000>. For production, use Apache or Nginx.

> **Note:** wallet extensions generally require the page to be served over `http://localhost` or **HTTPS**.

## Configuration Reference

All settings live in `backend/config/config.php`:

| Key | Description | Default |
|---|---|---|
| `db.path` | Path to the SQLite file | `backend/data/contest.db` |
| `contest_wallet_address` | Cardano address that receives entry fees | *(you provide)* |
| `entry_fee_ada` / `entry_fee_lovelace` | Entry fee (1 ADA = 1,000,000 lovelace) | `10` / `10000000` |
| `blockfrost_api_key` | Blockfrost Project ID | *(you provide)* |
| `blockfrost_base_url` | Mainnet or Preprod Blockfrost endpoint | Mainnet |
| `upload_dir` | Where audio files are stored | `backend/uploads/audio/` |
| `max_upload_bytes` | Max upload size | 25 MB |
| `allowed_audio_types` | Accepted MIME types | `audio/mpeg`, `audio/mp3`, `audio/wav`, `audio/x-wav` |
| `mail.host` / `port` | SMTP server and port | port `587` |
| `mail.username` / `password` | SMTP credentials | *(you provide)* |
| `mail.from_email` / `from_name` | Sender shown on emails | *(you provide)* |

If you change `entry_fee_ada`, also change `entry_fee_lovelace` (and the button/copy text in `index.html` and `app.js`), since they are separate values.

## API Reference

All endpoints live under `backend/api/` and return JSON.

| Endpoint | Method | Body | Description |
|---|---|---|---|
| `create_payment.php` | `POST` (JSON) | `{ "wallet_address" }` | Finds/creates the user, opens a `pending` entry-fee transaction, and returns `transaction_id`, `contest_id`, `pay_to_address`, `amount_lovelace`, `amount_ada`. |
| `verify_payment.php` | `POST` (JSON) | `{ "transaction_id", "tx_hash" }` | Checks the tx on-chain via Blockfrost. Returns `{ "status": "confirmed" }` or `{ "status": "pending" }`. |
| `upload_entry.php` | `POST` (multipart) | `transaction_id`, `title`, `ai_tool`, `description`, `email`, `audio` (file) | Creates an entry if the payment is confirmed and unused. Returns `{ "entry_id", "status": "submitted" }`. |
| `entries.php` | `GET` | none | Entries for the active round, ordered by votes (highest first). |
| `vote.php` | `POST` (JSON) | `{ "entry_id", "wallet_address" }` | Casts a vote. Returns the new `vote_count`, or `409` if already voted. |
| `send_results.php` | CLI | none | Emails every entrant their result and closes the round. |

Common error codes: `400` missing/invalid input, `402` payment not confirmed, `404` transaction not found, `409` no active round / payment already used / duplicate vote, `500` server error.

## Database Schema

| Table | Purpose |
|---|---|
| `users` | One row per wallet (`wallet_address` unique, auto-generated username like `wallet_ab12cd`, optional `email`) |
| `contests` | Contest rounds with `status` of `upcoming`, `active` or `closed` |
| `entries` | Submitted tracks (title, audio path, description, AI tool used) |
| `votes` | One row per vote, with `UNIQUE(entry_id, user_id)` to block double voting |
| `transactions` | ADA payments with `status` (`pending` / `confirmed` / `failed`), type (`entry_fee` / `prize_payout` / `donation`) and the on-chain `tx_hash` |

The full definition is in [`backend/schema.sql`](backend/schema.sql).

## Ending a Round & Sending Results

When a round finishes, run:

```bash
php backend/api/send_results.php
```

This emails each entrant (who provided an email) their vote count and whether they won, then sets the round's status to `closed`. **Insert a new `contests` row with `status = 'active'`** before the next round can accept payments and entries.

To automate it, use a cron job, for example every Monday at midnight:

```cron
0 0 * * 1 php /path/to/backend/api/send_results.php
```

> Ties: every entry that matches the top vote count is treated as a winner in the emails.

## Securing Your Deployment

- **Never commit secrets or data.** Add this to `.gitignore`:
  ```gitignore
  backend/config/config.php
  backend/data/
  backend/uploads/
  backend/vendor/
  ```
- **Block HTTP access** to `backend/config/` and `backend/data/`. With Apache, add a `.htaccess` containing `Deny from all` in each folder; with Nginx, add `location` blocks that deny those paths. Better still, move both folders outside the public web root.
- **Use HTTPS** in production.
- Keep your Blockfrost key and SMTP password out of the frontend and out of version control.

## Testing Checklist

- [ ] `schema.sql` creates all 5 tables and one active contest row
- [ ] `config.php` has real Blockfrost, SMTP and wallet values
- [ ] `composer require phpmailer/phpmailer` has been run
- [ ] `backend/data/` and `backend/uploads/` are writable
- [ ] Using a **Preprod** wallet with faucet ADA
- [ ] Wallet connects and its address shows in the header
- [ ] "Enter with 10 ADA" → approve in wallet → entry form unlocks
- [ ] A short test audio file uploads and appears in `backend/uploads/audio/` and the entries list
- [ ] Voting works, and a second vote with the same wallet is rejected
- [ ] Confirmation email arrives
- [ ] `send_results.php` sends results emails and marks the round `closed`

## Known Limitations

These are worth knowing about before going live with real funds:

- **Votes are not signature-verified.** `vote.php` trusts the `wallet_address` sent by the browser, so someone could submit votes for addresses they don't control. Consider requiring a CIP-30 signed message (`signData`) per vote.
- **Payments aren't tied to the paying wallet.** `verify_payment.php` confirms that *a* transaction paid the contest address the right amount, but doesn't check which address sent it.
- **Countdown is cosmetic.** The timer in `app.js` is hard-coded to "7 days from page load" and is not connected to the round's `end_date` in the database.
- **Rounds are not closed automatically.** Entries and payments remain open until `send_results.php` is run (or the round is closed manually).
- **Prize payouts are not automated.** The `prize_payout` transaction type exists in the schema, but sending the pot to the winner is a manual step.
- **No admin interface.** Rounds are managed directly in the database.
- **Wallet picker.** The frontend connects to the first detected wallet extension.

## License

No license 
