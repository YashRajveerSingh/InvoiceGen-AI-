# InvoiceGen AI — Smart Billing & Analytics System

**InvoiceGen AI** is a full-stack inventory management and smart billing platform designed to simplify business operations through centralized inventory management, invoice creation, and AI-powered business insights.

The system combines traditional inventory management with structured, database-grounded AI capabilities. Instead of using AI as a generic chatbot, InvoiceGen AI integrates intelligence directly into important business workflows:

- **AI Quick Invoice:** Converts natural-language order descriptions into structured invoice drafts.
- **Ask Your Ledger:** Answers business questions using real database-backed analytics.
- **Smart Reorder Brief:** Analyzes product sales velocity and generates inventory replenishment recommendations.

The backend remains the source of truth for product prices, stock quantities, invoice totals, and business calculations. Gemini AI is used to interpret information and generate useful summaries—not to independently control financial or inventory records.

---

## 🚀 Live Demo

**Live Application:** (https://stock-pilot-delta-gules.vercel.app/login)

**GitHub Repository:**[ https://github.com/YashRajveerSingh/StockPilot](https://github.com/YashRajveerSingh/InvoiceGen-AI-)

---

## ✨ Key Features

### 1. Authentication & Authorization

- User registration and login
- JWT-based authentication
- Protected application routes
- Role-Based Access Control (RBAC)
- User profile management

### 2. Dashboard & Analytics

- Overview of products, categories, suppliers, and orders
- Inventory statistics
- Business performance insights
- Revenue and order information
- Low-stock visibility

### 3. Product Management

- Add new products
- Update product information
- Delete products
- Search and filter products
- Manage product prices and stock quantities

### 4. Category Management

- Create product categories
- Update category information
- Delete categories
- View category listings

### 5. Supplier Management

- Add and manage suppliers
- Update supplier details
- Delete suppliers
- Track supplier information and status

### 6. Order & Invoice Management

- Create and manage orders
- View order history
- Update order status
- Track inventory movement
- Maintain stock consistency during invoice creation

### 7. User Management

- Manage user accounts
- Assign user roles
- Update user profiles
- Control access to protected features

---

# 🤖 AI-Powered Features

InvoiceGen AI integrates Google Gemini into business workflows to reduce manual work and support better operational decisions.

## 1. AI Quick Invoice — Natural Language to Validated Invoice

**Frontend:** `Orders.jsx`  
**API:** `POST /api/ai/parse-invoice`

### Problem

Business orders often arrive through WhatsApp messages, emails, or phone notes instead of structured forms.

For example:

> Bill Rahul Sharma for 2 Wireless Mouse and 1 Mechanical Keyboard, 18% GST, Delivered.

Manually identifying products, entering quantities, calculating taxes, and checking stock can be time-consuming and may lead to errors.

### How It Works

1. **Catalog-Grounded Extraction:** The backend retrieves the live MongoDB product catalog, including product IDs, names, categories, prices, and available stock.
2. **Structured AI Response:** Gemini receives the catalog and order description with a strict JSON response schema. It extracts the customer name, order status, tax rate, and requested product IDs and quantities.
3. **Backend Validation:** The backend verifies product IDs against MongoDB, retrieves authoritative product prices, checks stock availability, and calculates subtotal, tax, and total amounts.
4. **Human Review:** The generated invoice is populated as a draft in the invoice builder. The user can review and modify it before creating the invoice.
5. **Inventory Synchronization:** Once the invoice is created, the corresponding product stock is deducted through the application's billing workflow.

**Important:** AI does not decide the final product prices or invoice totals. These values are calculated and validated by the backend.

## 2. Ask Your Ledger — Natural Language Business Intelligence

**Frontend:** `DashboardHome.jsx`  
**API:** `POST /api/ai/ask-ledger`

### Problem

Business owners often need quick answers to questions such as:

- Who has unpaid invoices?
- What is my average order value?
- Which products generate the most revenue?
- Which products have low stock?

Finding these answers manually across multiple screens or writing database aggregation queries takes time.

### How It Works

1. **Database-Grounded Snapshot:** The backend calculates a read-only business data snapshot directly from MongoDB.
2. **AI Interpretation:** Gemini receives the prepared business information and the user's natural-language question.
3. **Structured JSON Output:** Gemini returns a structured response containing:
   - `summary`
   - `highlights`
   - `columns`
   - `rows`
   - `actionRoute`
   - `actionLabel`
4. **Interactive Dashboard:** The frontend displays the result using KPI highlight cards, structured data tables, and navigation actions.

The backend prepares the data rather than allowing the AI model to generate executable MongoDB queries.

### Business Metrics

The prepared snapshot can include:

- Delivered revenue
- Pending revenue
- Average order value
- Pending invoices
- Top customers
- Top products
- Low-stock products
- Category information

## 3. Smart Reorder Brief — Inventory Demand Analysis

**Frontend:** `DashboardHome.jsx`  
**API:** `GET /api/ai/reorder-insights`

### Problem

A fixed low-stock threshold does not consider how quickly a product is selling. A product selling five units per day needs attention sooner than a product selling one unit per month.

### How It Works

1. **Sales History Analysis:** The backend analyzes the previous 30 days of non-cancelled order items.
2. **Sales Velocity Calculation:** It calculates average daily sales for each product.
3. **Estimated Days Remaining:** It estimates how many days the current inventory may last.
4. **Reorder Quantity:** It calculates a suggested replenishment quantity using a 14-day stock buffer.
5. **AI Procurement Brief:** Gemini summarizes the calculated results into a short procurement recommendation.

### Inventory Metrics

| Metric | Description |
|---|---|
| Average Daily Sales | Average units sold per day |
| Days Left | Estimated days until stock runs out |
| 30-Day Sold | Total units sold during the previous 30 days |
| Suggested Reorder | Recommended quantity for replenishment |
| Low-Stock Status | Flags products with stock at or below 10 or estimated days left at or below 7 |

The calculations are performed by the backend. Gemini summarizes the results to make them easier to understand.

---

## 🛡️ AI Reliability & Production Resilience

**Implementation:** `server/services/aiService.js`

InvoiceGen AI includes fallback handling to reduce dependency on a single AI model.

### Multi-Model Failover

The AI service attempts multiple Gemini models:

1. `gemini-3.8-flash`
2. `gemini-flash-latest`
3. `gemini-flash-lite-latest`

It can try another configured model when an upstream request encounters supported errors such as `503`, `429`, or `404`.

### Deterministic Local Fallback

If the Gemini API key is unavailable or all configured models fail, the service can use a deterministic local fallback.

This helps preserve core billing and analytics workflows when the external AI service is unavailable. The fallback does not reproduce all generative AI capabilities.

### Reliability Principles

- Backend validation of AI-generated product references
- Database-controlled product prices and stock
- Deterministic invoice calculations
- Read-only data preparation for ledger questions
- Fallback handling for AI service failures
- Human review before invoice creation

---

## 🏗️ System Architecture

The application follows a full-stack client-server architecture.

```text
                 USER
                  |
                  v
          React Frontend
                  |
          Axios / REST API
                  |
                  v
          Node.js + Express
                  |
          +-------+--------+
          |                |
          v                v
      MongoDB          AI Service
          |                |
          |                v
          |          Google Gemini
          |                |
          |<---------------+
          |
          v
    Validated Business Data
          |
          v
   Dashboard / Invoice UI
```

### Architecture Responsibilities

| Layer | Responsibility |
|---|---|
| React Frontend | User interface, forms, dashboard, invoice builder |
| Express Backend | REST APIs, business logic, validation |
| MongoDB | Products, categories, suppliers, orders, and user data |
| Gemini AI | Natural-language interpretation and business summaries |
| AI Service | Model selection, structured responses, and fallback handling |

---

## 🛠️ Technology Stack

### Frontend

- React.js
- React Router DOM
- Axios
- Tailwind CSS
- React Icons
- JavaScript

### Backend

- Node.js
- Express.js
- REST API architecture

### Database

- MongoDB
- Mongoose

### Authentication & Security

- JSON Web Token (JWT)
- bcrypt.js
- Protected routes
- Role-Based Access Control
- Backend input validation

### Artificial Intelligence

- Google Gemini API
- Structured JSON responses
- JSON response schema
- Multi-model failover
- Deterministic fallback processing

---

## 📁 Project Structure

The repository is organized into frontend and server-side application directories.

```text
StockPilot/
│
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── Orders.jsx
│   │   │   └── DashboardHome.jsx
│   │   ├── pages/
│   │   ├── services/
│   │   ├── context/
│   │   ├── routes/
│   │   └── App.jsx
│   └── package.json
│
├── server/
│   ├── config/
│   ├── controllers/
│   ├── middleware/
│   ├── models/
│   ├── routes/
│   ├── services/
│   │   └── aiService.js
│   ├── utils/
│   └── server.js
│
├── .gitignore
└── README.md
```

*Note: This is a high-level structure. Keep the file and folder names aligned with the actual repository when updating the documentation.*

---

## 🗄️ Database Design

MongoDB is used as the primary database, with Mongoose providing schema modeling and database interaction.

The main business entities include:

| Collection / Entity | Purpose |
|---|---|
| User | Stores user account and role information |
| Product | Stores product details, prices, and stock |
| Category | Organizes products into categories |
| Supplier | Stores supplier information |
| Order / Invoice | Stores order, billing, item, tax, and status information |

### Database Responsibilities

- Store authoritative product prices and stock quantities.
- Maintain order and invoice records.
- Support dashboard analytics.
- Provide product catalog information for AI extraction.
- Supply historical order data for reorder analysis.

---

## 📡 API Endpoints

### Authentication

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/auth/register` | Register a user |
| POST | `/api/auth/login` | Authenticate a user |
| GET | `/api/auth/profile` | Retrieve user profile |

### Categories

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/category` | Get categories |
| POST | `/api/category` | Create a category |
| PUT | `/api/category/:id` | Update a category |
| DELETE | `/api/category/:id` | Delete a category |

### Products

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/product` | Get products |
| POST | `/api/product` | Create a product |
| PUT | `/api/product/:id` | Update a product |
| DELETE | `/api/product/:id` | Delete a product |

### Suppliers

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/supplier` | Get suppliers |
| POST | `/api/supplier` | Create a supplier |
| PUT | `/api/supplier/:id` | Update a supplier |
| DELETE | `/api/supplier/:id` | Delete a supplier |

### Orders

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/order` | Get orders |
| POST | `/api/order` | Create an order |
| PUT | `/api/order/:id` | Update an order |
| DELETE | `/api/order/:id` | Delete an order |

### AI Features

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/ai/parse-invoice` | Convert natural-language order text into a validated invoice draft |
| POST | `/api/ai/ask-ledger` | Answer business questions using prepared database analytics |
| GET | `/api/ai/reorder-insights` | Generate inventory demand and reorder insights |

---

## ⚙️ Installation & Setup

### 1. Clone the Repository

```bash
git clone https://github.com/YashRajveerSingh/StockPilot.git
cd StockPilot
```

### 2. Install Frontend Dependencies

```bash
cd frontend
npm install
```

### 3. Install Backend Dependencies

Open another terminal:

```bash
cd server
npm install
```

### 4. Configure Environment Variables

Create a `.env` file inside the `server` directory.

```env
PORT=5000
MONGO_URI=your_mongodb_connection_string
JWT_SECRET=your_jwt_secret_key
GEMINI_API_KEY=your_gemini_api_key
```

Use the exact environment variable names expected by your server configuration. Never commit real API keys, database credentials, or JWT secrets to GitHub.

### 5. Run the Backend

```bash
cd server
npm run dev
```

### 6. Run the Frontend

```bash
cd frontend
npm run dev
```

Open the local frontend URL displayed by Vite in your terminal.

---

## 🔒 Security Features

- JWT-based authentication
- Password hashing with bcrypt
- Protected frontend and backend routes
- Role-Based Access Control
- Input validation
- Backend-side product and stock verification
- AI-generated product references validated against MongoDB
- Invoice totals calculated by trusted backend logic
- AI ledger functionality based on prepared read-only business data
- Environment variables for sensitive credentials

---

## 📸 Application Screenshots

Add screenshots of your actual deployed application in this section.

Suggested screenshots:

1. Admin Dashboard
2. Product Management
3. Category Management
4. Supplier Management
5. Orders and Invoice Management
6. AI Quick Invoice
7. Ask Your Ledger
8. Smart Reorder Brief
9. User Profile

Example Markdown:

```markdown
![Admin Dashboard](frontend/screenshoots/Dashboard.png)

![Products](frontend/screenshoots/Products.png)

![Categories](frontend/screenshoots/Categories.png)
```

Update the image paths if your screenshot folder or filenames are different.

---

## 🚀 Future Enhancements

- Barcode and QR code scanning
- Advanced inventory demand forecasting
- Automated low-stock notifications
- Email and WhatsApp invoice delivery
- PDF and Excel invoice/report export
- Multi-warehouse inventory management
- Advanced business intelligence dashboards
- More AI-assisted operational workflows
- Improved supplier and procurement analytics

---

## 🎯 Learning Outcomes

This project demonstrates practical experience in:

- Full-stack web application development
- React-based frontend development
- REST API development using Node.js and Express
- MongoDB database design and integration
- Authentication and authorization
- CRUD operations and inventory management
- Business dashboard development
- AI integration using Google Gemini
- Structured AI response handling
- Backend validation and deterministic business calculations
- AI service fallback and error handling

---

## 👨‍💻 Author

**Yash Rajveer Singh**

Pre-Final Year B.Tech — Information Technology

### Technical Skills

- React.js
- JavaScript
- Tailwind CSS
- Node.js
- Express.js
- MongoDB
- SQL
- REST APIs
- AI API Integration

---

## 📜 License

This project is developed for educational and portfolio purposes.

You are welcome to explore, learn from, and enhance the project.

Username : admin@gmail.com
Password : admin

---

**InvoiceGen AI — Smarter Billing. Better Inventory. Data-Driven Decisions.**
