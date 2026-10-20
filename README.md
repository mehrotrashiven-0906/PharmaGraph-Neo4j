# PharmaGraph — Pharmacy Management System

A full-stack React + Express + Neo4j AuraDB website using your **existing** pharmacy database. The website does not seed, wipe, or replace existing data.

## Requirements
- Node.js 20.19+ (or 22.12+)
- An existing Neo4j AuraDB instance with the DA3 pharmacy graph
- Your AuraDB connection URI, database username, and database password (never send them to anyone or put them in client files)

## Setup (Windows PowerShell)
1. Extract the ZIP, open a terminal in the extracted folder.
2. `cd server`
3. `Copy-Item .env.example .env`
4. Edit `server/.env` with your own Neo4j **connection URI** and **database** credentials. Use the URI shown under Aura **Connect** (usually `neo4j+s://...`). The database is usually `neo4j`.
5. `npm install`
6. `npm run dev`
7. In a **second terminal**: `cd client`, `npm install`, `npm run dev`
8. Open `http://localhost:5173`.

For macOS/Linux: `cp .env.example .env` instead of Copy-Item.

## Check connectivity
Open `http://localhost:4000/api/health` in your browser. You should see `{"ok":true,"database":"neo4j"}`. If not, verify URI, password, instance status and database name. Do not use your Aura Console email as the database username unless your connection settings explicitly say to; the default database user is often `neo4j`.

## Features
- Live dashboard with counts, low stock and estimated sales value
- Add, read, edit, delete Supplier, Customer, Employee, Medicine, Prescription and Sale nodes
- Create/delete SUPPLIES, HAS_PRESCRIPTION, MAKES, HANDLES and CONTAINS relationships (including quantity)
- Customer purchase history, linked suppliers and estimated totals
- Interactive force-directed graph explorer
- All API Cypher is parameterized for user-supplied values. Entity labels and relationship types are selected from server-side allowlists.

## Important notes
- **No seed or reset script is run**. Your existing Neo4j data is used directly.
- Entity IDs are unique; use a new ID when creating a record. Duplicate IDs return an error.
- Delete uses `DETACH DELETE`: deleting a node also removes its relationships. The UI asks for confirmation.
- When creating a Sale, separately create its MAKES, HANDLES and CONTAINS relationships in the Relationships page. Likewise link prescriptions and medicines to their existing records.
- Estimated sales totals use **current** medicine prices, not historical sale-time prices, because the sample DA3 `CONTAINS` relationship only stores quantity. This is a demonstration, not a production accounting system.
- The Graph Explorer displays up to 300 relationships and only nodes participating in those relationships.
- This is a **local coursework/demo application**, not a production deployment: no login/authorization, audit log, rate limiting or HTTPS termination is provided. Do not expose port 4000 publicly. Never commit `.env` or put credentials in React.
- For a hosted deployment, add user authentication/authorization, HTTPS, secure server hosting, secret management, backup and monitoring first.

## API overview
`GET /api/health`, `/api/meta`, `/api/dashboard`, `/api/purchases`, `/api/graph`, `/api/relationships`

`GET /api/entities/:label`, `POST /api/entities/:label`, `PUT /api/entities/:label/:id`, `DELETE /api/entities/:label/:id`

`POST /api/relationships`, `DELETE /api/relationships`

## Existing schema
Nodes: Supplier(supplier_id,supplier_name,phone), Customer(customer_id,customer_name,phone), Employee(employee_id,employee_name,phone), Medicine(medicine_id,medicine_name,category,price,stock,expiry_date), Prescription(prescription_id,prescription_date,doctor_name), Sale(sale_id,sale_date,payment_mode,payment_status).

Relationships: Supplier-[:SUPPLIES]->Medicine; Customer-[:HAS_PRESCRIPTION]->Prescription; Customer-[:MAKES]->Sale; Employee-[:HANDLES]->Sale; Sale-[:CONTAINS {quantity}]->Medicine.

If you later change your Neo4j property names, update `definitions` in `server/src/index.js` to match them.
