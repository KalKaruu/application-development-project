# StartSmart AI – Sprint 1: US001 Register account + US002 Log in

Owner: Mohammad Harith Haikal
Stack: HTML + JavaScript (frontend) · Node.js/Express · MySQL (backend)

## What is in here

| Use case | What the student can do | Where the code is |
|---|---|---|
| US001 Register account | Create an account with name, email, password | `public/js/app.js` (vAuth, submitAuth), `src/routes/auth.js` (POST /api/register) |
| US002 Log in | Log in, see only their own subjects, log out (A1) | `src/routes/auth.js` (login, logout), `src/routes/subjects.js` |

Exception flows covered, using the same IDs as the SRS:

| ID | Flow | Behaviour |
|---|---|---|
| US001 E1 | Email already registered | "This email is already in use. Try logging in instead." Stays on the form. |
| US001 E2 | Invalid email or password under 8 characters | Field turns red and the rule is shown. |
| US001 E3 | Account creation fails | Error message, entered data kept. |
| US002 E1 | Wrong email or password | "Email or password is incorrect." Stays on the login screen. |
| US002 E2 | A field is empty | The empty field is highlighted. |
| US002 A1 | Log out | Session deleted on the server, login screen shown. |

## Folder structure

```
startsmart-sprint1/
├─ database/schema.sql     tables: users, sessions, subjects
├─ scripts/seed.js         creates the demo student from the prototype
├─ src/
│  ├─ db.js                MySQL connection
│  ├─ validate.js          validation rules (server side)
│  ├─ middleware.js        session creation + login check
│  └─ routes/auth.js, subjects.js
├─ public/                 the website the student sees
│  ├─ index.html
│  ├─ css/style.css        colours and fonts from the prototype
│  └─ js/api.js, validate.js, app.js
├─ server.js               starts everything
├─ package.json
└─ .env.example
```

## Setup (first time only)

### 1. Install the tools
- **Node.js LTS** (nodejs.org). Check with `node -v` (needs 18 or newer).
- **MySQL 8** (MySQL Installer on Windows, `brew install mysql` on Mac) **or XAMPP** (start "MySQL" in its control panel).
- **VS Code** (optional, any editor works).

### 2. Create the database
Open a terminal in this folder.

```bash
mysql -u root -p < database/schema.sql
```
(Enter your MySQL root password. With XAMPP and no password, just press Enter.)
Using phpMyAdmin instead? Open it, click **Import**, choose `database/schema.sql`, click **Go**.

### 3. Add your settings
```bash
cp .env.example .env        # Windows: copy .env.example .env
```
Open `.env` and set `DB_USER` and `DB_PASSWORD` to your MySQL login. Leave `DB_NAME=startsmart`.

### 4. Install packages and add the demo student
```bash
npm install
npm run seed
```
This creates the prototype demo account `aina@student.utm.my` / `studysmart` with 3 subjects.

### 5. Run
```bash
npm start
```
You should see `StartSmart AI running at http://localhost:3000` and `Database connected.`
Open **http://localhost:3000** in your browser.

Use `npm run dev` while coding: the server restarts when you save a file.

## Test it (use this as your demo and test checklist)

1. **US001 happy path:** Register tab, enter a name, `you@student.utm.my`, password `password123`, press Create account. You land on the home screen with "No subjects yet."
2. **US001 E2:** leave name empty, email `abc`, password `123`. Each field shows its rule.
3. **US001 E1:** log out, register the same email again. "This email is already in use."
4. **US002 happy path:** Log in tab, `aina@student.utm.my` / `studysmart`. You see the 3 saved subjects. Log in as your new account instead: the list is empty (students never see each other's data).
5. **US002 E2:** leave the password empty. Only that field is highlighted.
6. **US002 E1:** enter a wrong password. "Email or password is incorrect."
7. **A1 log out:** press Log out, you return to login. Pressing Back or refreshing does not bring the session back.
8. **E3:** stop MySQL, try to register. You get the error message and your typed data stays.
9. Refresh the page while logged in. You stay logged in (session restored from the saved token).

## Troubleshooting

| Message | Fix |
|---|---|
| `Database NOT connected: ER_ACCESS_DENIED_ERROR` | Wrong `DB_USER` / `DB_PASSWORD` in `.env`. |
| `ER_ACCESS_DENIED_NO_PASSWORD_ERROR` | Your MySQL user has a password. Put it in `.env`. |
| `ECONNREFUSED` | MySQL is not running. Start the service or XAMPP's MySQL. |
| `ER_BAD_DB_ERROR` | You skipped step 2 (run `schema.sql`). |
| `EADDRINUSE` | Port 3000 is taken. Change `PORT` in `.env`. |
| Blank page | Open it through `http://localhost:3000`, not by double-clicking `index.html`. |

## How it connects to the SRS diagrams

- **App → Server → Database** in the sequence diagrams is `public/js/api.js` → `src/routes/*.js` → MySQL.
- "Create account with hashed password" = bcrypt in `routes/auth.js`. Plain passwords are never stored.
- "Session token" = a random token. Only its SHA-256 hash is stored in `sessions`.
- "Get subjects of this student only" = `WHERE user_id = ?` in `routes/subjects.js`.

## For the team (other Sprint 1 members)

- Protect any new route with `requireAuth` from `src/middleware.js`. It gives you `req.user.id`, so always filter by `user_id`.
- Frontend calls go through `api.js`, which adds the login token automatically. Add new calls there.
- **US003 (subjects) and US004 (tasks)** can add routes next to `routes/subjects.js` and add their tables to `schema.sql`.
- Replace the "Add subject" comment in `vHome()` in `app.js` with the real button when US003 is ready.
- Remove the "Demo login" hint line in `vAuth()` before the final submission.

## Using Firebase instead of MySQL?

The frontend does not change. Only `src/routes/*` would be replaced by Firebase Authentication (`createUserWithEmailAndPassword`, `signInWithEmailAndPassword`, `signOut`) and Firestore. The same error messages and flows still apply. MySQL is used here because it matches the "Server → Database" and "hashed password / session token" steps in the SRS sequence diagrams.
