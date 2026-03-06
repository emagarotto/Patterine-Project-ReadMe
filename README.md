# # Craft Pattern SaaS Backend

A production-ready, multi-tenant SaaS backend for knitters and crocheters. This backend powers a platform where authenticated users can generate, edit, save, and share knitting/crochet patterns, while also enabling community interaction through comments and likes.

## Tech Stack

- **Runtime**: Node.js + TypeScript
- **Database**: PostgreSQL (via Supabase)
- **ORM**: Prisma
- **Auth**: Supabase Auth (JWT-based)
- **API**: REST, versioned under `/api/v1`
- **Validation**: Zod
- **Web Framework**: Express

## Project Structure

```
project/
├── prisma/
│   └── schema.prisma          # Prisma database schema
├── src/
│   ├── api/
│   │   └── v1/
│   │       ├── controllers/   # Request handlers
│   │       └── routes/        # API route definitions
│   ├── services/              # Business logic layer
│   ├── middleware/            # Auth, RBAC, validation, error handling
│   ├── lib/                   # Utilities (Prisma, Supabase clients)
│   ├── types/                 # TypeScript types and Zod schemas
│   └── server.ts              # Main entry point
├── .env                       # Environment variables
├── tsconfig.json              # TypeScript configuration
└── package.json               # Dependencies and scripts
```

## Data Models

### Core Entities

#### User
- Mirrors Supabase auth users
- Roles: USER, ADMIN
- Owns patterns, creates comments, likes patterns

#### Pattern (Primary Entity)
- Core entity for knitting/crochet patterns
- Fields: craft type, item type, title, instructions, gauge, yarn, needles/hook
- Source: AI-generated (from prompt or image) or imported from PDF
- Status: DRAFT, FINAL
- Visibility: PRIVATE, UNLISTED, PUBLIC
- Marketplace listing capability with pricing

#### Comment
- User comments on public patterns
- Cascade deleted with pattern/user

#### PatternLike
- User likes on public patterns
- Unique constraint: one like per user per pattern

### Enums
- `UserRole`: USER, ADMIN
- `CraftType`: KNITTING, CROCHET
- `PatternSource`: PROMPT, IMAGE, PDF
- `PatternStatus`: DRAFT, FINAL
- `PatternVisibility`: PRIVATE, UNLISTED, PUBLIC

## Access Control

The backend implements comprehensive Row Level Security (RLS) policies:

### Public (Unauthenticated)
- View PUBLIC patterns
- Read comments and like counts
- Cannot create, comment, or like

### Authenticated Users
- CRUD their own patterns
- Comment on PUBLIC patterns
- Like PUBLIC patterns
- View UNLISTED patterns via direct link

### Admins
- Read/update all patterns
- Hide or unlist public patterns
- Delete comments
- Full moderation capabilities

## API Endpoints

All endpoints are versioned under `/api/v1`

### Patterns

```
POST   /api/v1/patterns                Create pattern (auth required)
GET    /api/v1/patterns                List patterns (public + auth)
GET    /api/v1/patterns/:id            Get pattern by ID
PUT    /api/v1/patterns/:id            Update pattern (owner/admin)
DELETE /api/v1/patterns/:id            Delete pattern (owner/admin)
POST   /api/v1/patterns/import/pdf     Import pattern from PDF (auth required)
GET    /api/v1/patterns/:id/export     Export pattern as PDF
GET    /api/v1/patterns/:id/preview    Export pattern preview PDF
```

### Comments

```
POST   /api/v1/patterns/:id/comments         Create comment (auth required)
GET    /api/v1/patterns/:id/comments         List comments (public)
PUT    /api/v1/patterns/comments/:commentId  Update comment (author)
DELETE /api/v1/patterns/comments/:commentId  Delete comment (author/admin)
```

### Likes

```
POST   /api/v1/patterns/:id/like    Like pattern (auth required)
DELETE /api/v1/patterns/:id/like    Unlike pattern (auth required)
GET    /api/v1/patterns/:id/like    Get like status
```

## Setup Instructions

### Prerequisites

- Node.js 18+ and npm
- Supabase account and project
- Database connection string

### Installation

1. Install dependencies:
```bash
npm install
```

2. Configure environment variables in `.env`:
```env
# Supabase Configuration
SUPABASE_URL=your_supabase_url
SUPABASE_ANON_KEY=your_anon_key
SUPABASE_SERVICE_ROLE_KEY=your_service_role_key

# Database
DATABASE_URL=your_database_connection_string

# Server
PORT=3000
NODE_ENV=development

# AI Services (required for PDF import)
OPENAI_API_KEY=your_openai_api_key
```

3. Generate Prisma Client:
```bash
npm run prisma:generate
```

4. Database is already migrated via Supabase. The schema includes:
   - All tables with proper relationships
   - RLS policies for security
   - Indexes for performance

## Development

### Run Development Server
```bash
npm run dev
```

The server will start on `http://localhost:3000` with hot-reload enabled.

### Build for Production
```bash
npm run build
```

### Start Production Server
```bash
npm start
```

### Database Tools
```bash
# Open Prisma Studio (visual database browser)
npm run prisma:studio
```

## API Usage Examples

### Create a Pattern
```bash
curl -X POST http://localhost:3000/api/v1/patterns \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "craftType": "KNITTING",
    "itemType": "sweater",
    "title": "Cozy Cable Knit Sweater",
    "gauge": {
      "stitches": 20,
      "rows": 28,
      "measurement": "4",
      "unit": "inches"
    },
    "yarn": {
      "weight": "worsted",
      "amount": 1200,
      "unit": "yards"
    },
    "needles": {
      "size": "US 7",
      "type": "circular"
    },
    "instructions": [
      {
        "type": "row",
        "number": 1,
        "text": "Cast on 100 stitches"
      }
    ],
    "sourceType": "PROMPT",
    "sourceInput": "Generate a cozy cable knit sweater pattern"
  }'
```

### List Patterns
```bash
# Public access
curl http://localhost:3000/api/v1/patterns?page=1&limit=20

# With authentication (see more patterns)
curl http://localhost:3000/api/v1/patterns?page=1&limit=20 \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

### Get Pattern by ID
```bash
curl http://localhost:3000/api/v1/patterns/PATTERN_ID \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

### Create Comment
```bash
curl -X POST http://localhost:3000/api/v1/patterns/PATTERN_ID/comments \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "Beautiful pattern! Can'\''t wait to make this."
  }'
```

### Like a Pattern
```bash
curl -X POST http://localhost:3000/api/v1/patterns/PATTERN_ID/like \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

### Import Pattern from PDF
```bash
curl -X POST http://localhost:3000/api/v1/patterns/import/pdf \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -F "pdf=@pattern.pdf"
```

See `/docs/PDF_IMPORT.md` for detailed PDF import documentation.

## Security

- All sensitive operations require JWT authentication
- RLS policies enforced at database level
- Service role key bypasses RLS for admin operations
- Input validation with Zod schemas
- Proper error handling without exposing internals
- No hardcoded secrets or credentials

## Error Handling

The API returns consistent error responses:

```json
{
  "error": {
    "message": "Error description",
    "details": {}
  }
}
```

Common HTTP status codes:
- `200`: Success
- `201`: Created
- `204`: No Content (successful deletion)
- `400`: Bad Request
- `401`: Unauthorized
- `403`: Forbidden
- `404`: Not Found
- `409`: Conflict
- `422`: Validation Error
- `500`: Internal Server Error

## Features

### PDF Pattern Import (✅ Implemented)
- AI-powered extraction and normalization of existing patterns from PDF
- Automatic text extraction and field identification
- GPT-4 powered repair and structuring
- Validation against PatternInstructions schema
- Original PDF preservation for reference

See `/docs/PDF_IMPORT.md` for detailed documentation.

### Pattern PDF Export (✅ Implemented)
- Full pattern export with all sections
- Preview PDF generation for marketplace
- Subscription-aware watermarking
- Professional formatting

See `/docs/PDF_EXPORT_IMPLEMENTATION.md` for detailed documentation.

## Future Enhancements (TODO)

- [ ] AI pattern generation from prompts
- [ ] AI pattern generation from images
- [ ] Payment processing integration
- [ ] Subscription and billing management
- [ ] Marketplace transaction handling
- [ ] Advanced search and filtering
- [ ] Pattern version history
- [ ] User favorites/collections
- [ ] Email notifications
- [ ] Admin dashboard API

## Architecture Decisions

### Schema-First Design
All data models are defined in Prisma schema first, ensuring type safety and consistency.

### Service Layer Pattern
Business logic lives in services, not controllers. Controllers handle HTTP concerns only.

### Comprehensive Access Control
RLS policies at database level + middleware checks for defense in depth.

### Validation Pipeline
Request validation happens at route level using Zod schemas before reaching controllers.

### Conservative Approach
- No over-engineering
- Clear separation of concerns
- Production-minded code quality
- Explicit over implicit

## License

Proprietary - All rights reserved
