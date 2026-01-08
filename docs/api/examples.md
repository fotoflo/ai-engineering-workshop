🚀 **FLEXBIKE IMPORT API EXAMPLES**

## Single Record Imports

### 1. Import a Single Company
curl -X POST http://localhost:3000/api/admin/companies/import \
  -H 'Content-Type: application/json' \
  -d '{
    "companyData": {
      "id": "new-company-123",
      "name": "Paradise Rentals",
      "area": "Ubud",
      "companyAddress": "Jl. Monkey Forest, Ubud",
      "phoneNumber": "+628123456789"
    }
  }'

### 2. Import a Single Product  
curl -X POST http://localhost:3000/api/admin/products/import \
  -H 'Content-Type: application/json' \
  -d '{
    "productData": {
      "id": "new-bike-456", 
      "name": "Honda Beat 2023",
      "companyID": "existing-company-id",
      "dailyPrice": 120000,
      "description": "Latest model with great fuel economy"
    }
  }'

### 3. Import a Single User
curl -X POST http://localhost:3000/api/admin/users/import \
  -H 'Content-Type: application/json' \
  -d '{
    "userData": {
      "id": "new-user-789",
      "name": "John Doe",
      "phoneNumber": "+628987654321",
      "email": "john@example.com"
    }
  }'

## Bulk Imports (Array of Records)

### 4. Bulk Import Multiple Companies
curl -X POST http://localhost:3000/api/admin/companies/bulk-import \
  -H 'Content-Type: application/json' \
  -d '[
    {
      "id": "bulk-comp-1",
      "name": "Beach Bike Rentals",
      "area": "Seminyak"
    },
    {
      "id": "bulk-comp-2", 
      "name": "Mountain Bike Adventures",
      "area": "Ubud"
    }
  ]'

### 5. Bulk Import Products with Transforms
curl -X POST http://localhost:3000/api/admin/products/bulk-import \
  -H 'Content-Type: application/json' \
  -d '[
    {
      "id": "prod-1",
      "name": "Yamaha NMAX",
      "companyID": "bulk-comp-1",
      "dailyPrice": "180000",
      "averageReviewScore": "4.5"
    },
    {
      "id": "prod-2",
      "name": "Honda PCX",
      "companyID": "bulk-comp-2", 
      "dailyPrice": 150000,
      "numHelmets": "I have my own"
    }
  ]'

### 6. Bulk Import Bookings with Validation
curl -X POST http://localhost:3000/api/admin/bookings/bulk-import \
  -H 'Content-Type: application/json' \
  -d '[
    {
      "id": "booking-1",
      "riderID": "new-user-789",
      "bikeID": "prod-1",
      "startDate": "2024-12-01T00:00:00.000Z",
      "endDate": "2024-12-05T00:00:00.000Z",
      "status": "confirmed"
    }
  ]'

## Data Quality Features

### Automatic Transformations Applied:
- **averageReviewScore**: "4.5" → 5 (rounded Int)
- **numHelmets**: "I have my own" → 0 (safe Int)
- **status**: "expired" → "cancelled" (valid enum)
- **collectionTime**: "12:00pm" → null (invalid DateTime)
- **title**: undefined → "Product {id}" (fallback)

### Validation Rules:
- Companies require valid area for location backfilling
- Products require existing company references  
- Bookings require existing user and product references
- All foreign keys are validated before import

## Full Migration Script

npx tsx full-api-import.ts

This runs the complete migration in dependency order:
1. Users → 2. Companies → 3. Products → 4. Bookings

## Error Handling

All APIs return detailed error information:
{
  "success": true,
  "results": {
    "processed": 5,
    "created": 4, 
    "errors": 1,
    "details": [
      {
        "id": "problem-record",
        "error": "Referenced company does not exist"
      }
    ]
  }
}
