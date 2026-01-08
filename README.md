# Flexbike Web App

Flexbike is a modern Next.js-based web application for renting motorbikes. Built with pnpm and styled using Tailwind CSS, Flexbike allows users to browse motorbikes from trusted rental shops and book directly through the app.

## 📚 Documentation

**🚀 New to Flexbike?** Start here:
- [**📖 Complete Documentation Hub**](agents/docs/README.md) - Centralized guides for all aspects
- [**🎯 Agent Guide**](agents/docs/agents/README.md) - Codebase overview, conventions, and common tasks
- [**🗄️ Database Guide**](agents/docs/agents/database.md) - Schema management, migrations, and safety protocols

## Features

- **Next.js App Router:** Leverages Next.js’s modern file-based routing.
- **Responsive Design:** Fully responsive design built with Tailwind CSS.
- **Custom Fonts:** Optimized font loading using Next.js’s built-in Google Font support and custom CSS.
- **QR Code Modal:** Display QR codes for downloading the app (optimized for desktop vs. mobile).
- **Accessibility:** Implemented using [react-modal](https://reactcommunity.org/react-modal/) with proper app element configuration.
- **Optimized for Performance:** Uses server-side rendering and static generation for fast load times.

## Getting Started

Follow these instructions to run Flexbike locally.

### Prerequisites

- **Node.js:** Version 14.x or later
- **pnpm:** Preferred package manager ([Install pnpm](https://pnpm.io/installation))

### Installation

1. **Clone the Repository:**

   ```bash
   git clone https://github.com/yourusername/flexbike-web-app.git
   cd flexbike-web-app
   ```

2. **Install Dependencies:**

   ```bash
   pnpm install
   ```

3. **Set Up Environment Variables:**

   Create a `.env.local` file in the root of the project and add the necessary environment variables. Refer to `example.env` for the required keys.

   ```bash
   cp example.env .env.local
   ```

4. **Run the Development Server:**

   ```bash
   pnpm dev
   ```

   Open [http://localhost:3000](http://localhost:3000) to see the app running.

## Database Setup & Import

This project includes comprehensive scripts to set up the database and migrate data from Firebase to Supabase.

### Quick Setup (Recommended)

For new environments, use the automated setup script:

```bash
npm run import:setup
```

This script will:

- Start the Next.js dev server (if not running)
- Set up the database schema with Prisma
- Import all data from Firebase (~19,743 records)
- Run location seeding for cities/regions/countries
- Verify the import worked correctly

### Manual Setup

If you prefer to run steps individually:

1. **Start the dev server:**

   ```bash
   npm run dev
   ```

2. **Set up database schema:**

   ```bash
   npx prisma db push --accept-data-loss
   ```

3. **Import data:**

   ```bash
   npm run import:full  # Full import (~19k records)
   npm run import:small # Small import for testing (100 records)
   ```

4. **Seed location data:**
   ```bash
   node scripts/seed-locations.cjs
   ```

### Import Scripts

- **`npm run import:setup`**: Complete automated setup (recommended)
- **`npm run import:full`**: Import all data from Firebase
- **`npm run import:small`**: Limited import for testing (100 records each)
- **`npm run db:import-full`**: Legacy full database import script
- **`npm run db:import-by-companies`**: Import companies only
- **`npm run db:import-by-bookings`**: Import bookings only
- **`npm run db:import-messages`**: Import messages only
- **`npm run db:import-by-reviews`**: Import reviews only

### Verification & Testing

After import, verify everything works:

```bash
# Test Chiang Mai search (should return bikes)
curl "http://localhost:3000/thailand/northern-thailand/chiang-mai/bikes"

# Check company count
curl "http://localhost:3000/api/companies?take=1" | jq length

# Use Prisma Studio to browse data
npx prisma studio
```

### Import Data Summary

The full import includes:

- **357 companies** with rental locations
- **1,897 products** (bikes/scooters)
- **4,417 bookings** and reservations
- **1,649 conversations** between users
- **185 reviews** and ratings
- **9,619 messages** in conversations
- **Location data**: 27 countries, 79 regions, 108 cities

### Troubleshooting

**Import fails with authentication errors:**

- Ensure `SYNC_API_KEY` is set in `.env.local`
- Make sure the dev server is running on `localhost:3000`

**Database connection issues:**

- Check your `DATABASE_URL` in `.env.local`
- Run `npx prisma db push --accept-data-loss` to set up schema

**Location seeding fails:**

- Run `npx prisma db push` first to ensure tables exist
- Then run `node scripts/seed-locations.cjs`

### Preview Scripts (for debugging)

To verify data transformation without importing:

- `npm run db:preview-company -- --id <firestore-doc-id>`
- `npm run db:preview-product -- --id <firestore-doc-id>`
- `npm run db:preview-booking -- --id <firestore-doc-id>`
- `npm run db:preview-review -- --id <firestore-doc-id>`
- `npm run db:preview-user -- --id <firestore-doc-id>`
