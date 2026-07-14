# Sales Forecasting & Retail Management Platform

A full-stack business-management tool for retailers, built as my graduation project (BSc Software Engineering, The British University in Egypt). It combines a MERN-stack web app with a Python machine learning model that forecasts weekly sales.

> **A note on this repository:** the original source code was lost to a hardware failure when I switched laptops. What's preserved here is the full project report (`report.pdf`), which documents the architecture, implementation, and evaluation in detail, along with screenshots of the working application. I'm keeping this project listed because the engineering behind it — the model comparison, the reusable UI architecture, the honest evaluation of what didn't work — is real work worth showing, even without a live repo to click through.

## What it does

Retailers get a role-based dashboard (Admin, Manager, Team Leader) to manage staff, stores, and supplies, plus a sales forecasting tool that predicts future demand from historical sales data — so a business owner can plan inventory and staffing ahead of time instead of reacting to it.


## Tech stack

| Layer | Technology |
|---|---|
| Frontend | React, Material UI + Tailwind |
| Backend | Node.js + Express (two servers: one for the main API, one dedicated to email notifications) |
| Database | MongoDB |
| Forecasting model | Python — scikit-learn (Random Forest) and Keras/TensorFlow (LSTM), served via a Flask microservice |
| Auth | JWT, hashed passwords, role-based access (Admin / Manager / Team Leader) |

## Architecture

The app is split into four layers:

- **Front-end layer** — pages, navigation, and reusable UI components (data grids, dialogs, and forms that are driven by props rather than hard-coded per page, so the same component renders staff tables, supply tables, or team member cards)
- **Utility layer** — custom hooks (`useAuthToken`, `useFetchStaffMembers`, etc.) that decode the JWT from cookies and gate data fetching behind a valid, role-checked session
- **Business layer** — Express controllers and routers handling CRUD for staff, stores, supplies, and orders
- **Data layer** — Mongoose schemas/models

The forecasting model runs as a separate Flask service (using `flask-cors`) rather than living inside the Node backend — keeping the ML pipeline isolated from the web app's request/response cycle.


## The forecasting model

Trained and evaluated on Walmart's public 45-store historical sales dataset (weekly sales by store and department, plus contextual features like holidays, fuel price, and CPI).

- **Feature selection**: after checking each feature's relationship to weekly sales, low-signal features (CPI, unemployment, fuel price, temperature, markdowns) were dropped — the model kept store, department, and date as the primary predictors.
- **Models compared**: ARIMA, Random Forest, and an LSTM (recurrent neural network).
- **Result**: the LSTM model reached an R² of ~0.95 and RMSE of ~4980 on held-out weekly sales, forecasting reasonably well in the short range before accuracy degraded further out.
- **Known limitation** (documented honestly in the original report): the model trained and evaluated well on the prepared dataset, but wiring up the live "predict from real user input" endpoint wasn't finished — matching a live user's input shape to the model's expected training input turned out to be the hardest integration problem in the project, and it's called out directly in the report's future-work section rather than glossed over.


## Notable engineering details

- **Auth**: passwords are hashed, JWTs carry the user's id/role and are decoded client-side via a custom hook to drive redirects and gate API calls
- **Reusable components**: one generic `DataGridComponent`, `Dialog`, and form component power every table, confirmation prompt, and pop-up form in the app, configured entirely through props
- **Notifications**: a Nodemailer-based email service, decoupled into its own small Express server to avoid conflicting with the main API server
- **Usability testing**: the app was evaluated by 12 real users across three age groups on design, ease of use, navigation, and perceived usefulness — every respondent rated design, ease of use, and navigation 7/10 or higher

## Full report

📄 [`report.pdf`](./report.pdf) — the complete write-up: literature review, methodology, requirements, UML diagrams (package, use case, sequence), implementation walkthrough with code, model training details, testing, evaluation, and future work.
