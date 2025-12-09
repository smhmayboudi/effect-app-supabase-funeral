# Funeral Service Platform

## Position Overview:

You will be joining a team building a comprehensive web-based SaaS platform to transform funeral service planning and management. The platform aims to create transparency, accessibility, and respect in this process, allowing users to search, compare, and select suitable locations, various organizational services, and pricing plans, while providing service providers with a dashboard to manage their offerings.

## Key Responsibilities:

1. Database Design & Implementation

- Design complex relational schemas in Supabase (PostgreSQL)
- Manage core entities: users, organizations, services, pricing plans, burial locations, reservations
- Optimize queries and table relationships

2. Frontend Development

- Build UI with Next.js (App Router) and TypeScript
- Design with Tailwind CSS and potentially shadcn/ui
- Develop responsive, accessible components

3. Authentication & Security System

- Implement complete Supabase Auth
- Manage Role-Based Access Control (RBAC) with RLS
- Protect routes and API endpoints

4. Management Dashboards

- Develop separate dashboards for:
- Customers (search, compare, book)
- Organization managers (manage services, pricing, requests)
- System admin (overall platform management)

5. Map Integration & Location Search

- Connect to Mapbox/Google Maps API
- Display burial locations on maps
- Implement location-based search

6. Payment & Financial System

- Integrate Stripe for payments
- Manage webhooks for transaction confirmation
- Implement billing and payment tracking

7. Forms & Processes

- Build multi-step forms with React Hook Form + Zod
- Client and server-side validation
- Smart booking process

8. Document Generation

- Create PDF files (invoices, contracts)
- Convert HTML to PDF

9. API & Backend Development

- Write Edge Functions in Supabase
- Connect with third-party APIs
- Implement business logic

10. Testing & Quality Assurance

- Write Unit and E2E tests
- Ensure system stability

## Proposed Implementation Roadmap

### Phase 1: Foundation & Setup (Weeks 1-2)

#### Week 1: Project Setup & Architecture

1. Requirements Analysis

- Team meetings to understand business details
- Define User Stories and Use Cases

2. Project Setup

- Configure Supabase project
- Set up Vercel and GitHub
- Install and configure tools:
  - shadcn/ui
  - React Hook Form + Zod
  - TanStack Query (optional)
  - Testing suite (Jest/Vitest + Playwright)

3. Architecture Design

- Project folder structure
- Initial Design System
- Development environment setup

### Week 2: Database Design

1. ERD Design

```text
Locations
Organizations
Pricing Plans
Reservations
Services
Users
```

2. Supabase Implementation

- Create tables with RLS policies
- Establish essential relationships
- Seed initial data

3. Basic Authentication

- Configure Supabase Auth
- Signup/Login pages
- Session management

### Phase 2: Core System Development (Weeks 3-6)

#### Week 3-4: Base Frontend & Components

1. Design System

- Core components with shadcn/ui
- Shared layouts
- Theme and styles

2. Map Setup

- Integrate Mapbox/Google Maps
- Base map component
- Location display on maps

3. Navigation & Routing

- Protected routes
- Authentication middleware

#### Week 5-6: Core Business Logic

1. Customer Dashboard

- Search and filter page
- Results display
- Service comparison

2. Booking Process

- Multi-step form
- Zod validation
- Information preview

### Phase 3: Advanced Features (Weeks 7-10)

#### Week 7-8: Payment & Financial Systems

1. Stripe Integration

- Stripe account setup
- Checkout pages
- Webhook management

2. Document Generation

- HTML components for invoices
- PDF conversion

3. Management Dashboard

- Data tables and lists
- Statistics and reports

#### Week 9-10: Advanced Maps & Search

1. Location Search

- Geolocation-based search
- Advanced filters
- Routing capabilities

2. Edge Functions

- Server-side logic
- Custom APIs

### Phase 4: Completion & Testing (Weeks 11-12)

#### Week 11: Testing

1. Unit Tests

- Core components
- Utility functions

2. E2E Tests

- Main user scenarios
- Complete booking process

3. Security Testing

- RLS policy review
- Vulnerability testing

#### Week 12: Optimization & Deployment

1. Performance Optimization

- Bundle size analysis
- Image optimization
- Caching strategies

2. Production Preparation

- Production environment
- Monitoring setup
- Backup strategy

3. Documentation

- Technical documentation
- Deployment guide
- Troubleshooting guide

## Technical Recommendations & Best Practices:

### Suggested Project Structure:

#### WEB

```text
src/
├── app/
│   ├── (auth)/           # Authentication pages
│   ├── (customer)/       # Customer section
│   ├── (provider)/       # Service provider section
│   ├── (admin)/          # Admin section
│   └── api/              # API routes
├── components/
│   ├── ui/              # Base UI components
│   ├── forms/           # Form components
│   ├── maps/            # Map components
│   └── dashboard/       # Dashboard components
├── lib/
│   ├── db/              # Database utilities
│   ├── validators/      # Zod schemas
│   ├── utils/           # Utility functions
│   └── constants/       # Constants
├── hooks/               # Custom hooks
├── types/               # TypeScript types
└── tests/               # Tests
```

### Estimated Timeline:

| Phase | Duration | Key Deliverables |
| - | - | - |
| Phase 1 | 2 weeks | Architecture, DB design, Auth |
| Phase 2 | 4 weeks | Base components, Maps, Forms |
| Phase 3 | 4 weeks | Payments, Dashboards, Edge Functions |
| Phase 4 | 2 weeks | Testing, Optimization, Deployment |

Total: 12 weeks

### Risks & Considerations:

1. Sensitive Subject: UI design must respect funeral service context
2. Map Complexity: Map service integration and location search
3. Data Security: High protection needed for sensitive user data
4. Payment Integrity: Payment system must be highly reliable

### Key Success Factors:

- Regular communication with team to understand business needs
- Progressive Enhancement implementation
- Accessibility focus from the start
- Comprehensive code documentation
- Meaningful test coverage

This roadmap is flexible and can be adjusted based on team feedback and emerging requirements.
