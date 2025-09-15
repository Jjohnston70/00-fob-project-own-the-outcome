## System Overview

Forward Operating Base (FOB) is designed as a **microservices ecosystem** with six independent but coordinated pillars. Each pillar operates autonomously while sharing veteran data and coordinating services through a central platform. The architecture supports multi-city deployment, veteran-led licensing, and seamless integration with existing partner systems.

### Core Architectural Principles
- **Veteran-Centric Design:** All systems prioritize veteran outcomes and experience
- **Partner Integration First:** Enhance existing systems rather than replace them  
- **Operational Excellence:** Military-grade reliability and precision
- **Scalable by Design:** Support growth from 1 city to 100+ cities
- **Security by Default:** Veteran data protection at every layer

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     Forward Operating Base                       │
│                        API Gateway                              │
├─────────────────────────────────────────────────────────────────┤
│  Shared Services Layer                                          │
│  • Veteran Profile Management  • Authentication                │
│  • Impact Tracking            • Communication                  │
│  • City/Tenant Management     • Analytics                      │
├─────────────────────────────────────────────────────────────────┤
│                    Six Pillar Services                         │
├─────────────┬─────────────┬─────────────┬─────────────────────┤
│   Table     │   Housing   │    Intel    │     Capital         │
│ (Food Sec)  │ (Housing)   │ (Real Est)  │   (Finance)         │
├─────────────┼─────────────┼─────────────┼─────────────────────┤
│  Mobility   │    Fuel     │             │                     │
│(Transport)  │ (Logistics) │             │                     │
├─────────────────────────────────────────────────────────────────┤
│                 Integration Layer                               │
│  • Partner APIs            • Webhook Management                │
│  • External System Sync    • Real-time Messaging              │
├─────────────────────────────────────────────────────────────────┤
│                   Data Layer                                   │
│  • Veteran Profiles        • Partner Management                │
│  • Transaction History     • Impact Metrics                    │
│  • Audit Logs             • City Configuration                 │
└─────────────────────────────────────────────────────────────────┘
```

## Pillar-Specific Architectures

### Forward Operating Table (Food Security)
**Purpose:** Coordinate restaurant and supermarket food waste with veteran meal programs

**Components:**
- **Restaurant Automation Service:** POS integration, inventory tracking, donation optimization
- **Supermarket Integration Service:** Enterprise system integration, volume coordination
- **Delivery Coordination Service:** Route optimization, driver management, real-time tracking
- **Food Safety Service:** Temperature monitoring, expiration tracking, compliance verification

**Technical Stack:**
- **Backend:** Node.js with Express, Firebase Functions for real-time processing
- **Database:** Firestore for real-time inventory, PostgreSQL for analytics
- **Integrations:** Square, Toast, Clover APIs for restaurants; NCR, Oracle for supermarkets
- **Mobile:** React Native for driver app with GPS and camera functionality

**Key Databases:**
```sql
-- Restaurant Inventory
restaurants {
  id: uuid PRIMARY KEY,
  name: varchar(255),
  address: text,
  pos_system: varchar(100),
  contact_info: jsonb,
  veteran_owned: boolean,
  created_at: timestamp
}

food_inventory {
  id: uuid PRIMARY KEY,
  restaurant_id: uuid REFERENCES restaurants(id),
  item_name: varchar(255),
  quantity: decimal,
  expiration_date: date,
  donation_threshold: decimal,
  tax_value: decimal,
  status: enum('available', 'reserved', 'donated', 'expired')
}

deliveries {
  id: uuid PRIMARY KEY,
  pickup_location: uuid REFERENCES restaurants(id),
  delivery_location: uuid REFERENCES shelters(id),
  driver_id: uuid REFERENCES drivers(id),
  scheduled_time: timestamp,
  actual_pickup: timestamp,
  actual_delivery: timestamp,
  items: jsonb,
  status: enum('scheduled', 'in_progress', 'completed', 'failed')
}
```

### Forward Operating Housing (Housing Security)
**Purpose:** Coordinate builder resources with veteran housing projects

**Components:**
- **Project Management Service:** Construction project tracking, milestone management
- **Resource Coordination Service:** Material donation tracking, volunteer scheduling
- **Quality Control Service:** Construction standards, safety compliance, inspections
- **Veteran Placement Service:** Move-in coordination, housing readiness assessment

**Technical Stack:**
- **Backend:** Python Django for complex project management logic
- **Database:** PostgreSQL with PostGIS for location-based project coordination
- **Integrations:** Procore, PlanGrid APIs for existing construction management
- **Mobile:** Progressive Web App for on-site coordination and photo documentation

**Key Databases:**
```sql
-- Housing Projects
housing_projects {
  id: uuid PRIMARY KEY,
  name: varchar(255),
  address: text,
  builder_id: uuid REFERENCES builders(id),
  target_veteran_id: uuid REFERENCES veterans(id),
  project_type: enum('new_construction', 'renovation', 'accessibility_modification'),
  start_date: date,
  target_completion: date,
  actual_completion: date,
  budget: decimal,
  status: enum('planning', 'in_progress', 'inspection', 'completed')
}

project_resources {
  id: uuid PRIMARY KEY,
  project_id: uuid REFERENCES housing_projects(id),
  resource_type: enum('material', 'labor', 'equipment'),
  description: text,
  quantity_needed: decimal,
  quantity_secured: decimal,
  cost_estimated: decimal,
  cost_actual: decimal,
  donor_id: uuid REFERENCES donors(id)
}

project_milestones {
  id: uuid PRIMARY KEY,
  project_id: uuid REFERENCES housing_projects(id),
  milestone_name: varchar(255),
  target_date: date,
  completion_date: date,
  dependencies: jsonb,
  status: enum('pending', 'in_progress', 'completed', 'blocked')
}
```

### Forward Operating Intel (Housing Intelligence)
**Purpose:** Leverage real estate agent networks for veteran housing placement

**Components:**
- **Agent CRM Enhancement:** Veteran client management, property matching
- **MLS Integration Service:** Property search automation, market intelligence
- **Market Analysis Service:** Pricing trends, affordability analysis, neighborhood assessment
- **Placement Coordination Service:** Application assistance, showing coordination

**Technical Stack:**
- **Backend:** Node.js with GraphQL for flexible real estate data queries
- **Database:** PostgreSQL with full-text search for property matching
- **Integrations:** MLS APIs, Zillow, Realtor.com for property data
- **Frontend:** React with mapping libraries for property visualization

**Key Databases:**
```sql
-- Real Estate Intelligence
properties {
  id: uuid PRIMARY KEY,
  mls_id: varchar(50),
  address: text,
  city: varchar(100),
  state: varchar(2),
  zip_code: varchar(10),
  property_type: enum('house', 'apartment', 'condo', 'townhouse'),
  bedrooms: integer,
  bathrooms: decimal,
  square_feet: integer,
  rent_price: decimal,
  sale_price: decimal,
  veteran_friendly: boolean,
  accessibility_features: jsonb,
  available_date: date
}

veteran_housing_preferences {
  id: uuid PRIMARY KEY,
  veteran_id: uuid REFERENCES veterans(id),
  preferred_locations: jsonb,
  max_rent: decimal,
  max_purchase: decimal,
  accessibility_needs: jsonb,
  family_size: integer,
  pet_friendly: boolean,
  transportation_needs: jsonb
}

housing_applications {
  id: uuid PRIMARY KEY,
  veteran_id: uuid REFERENCES veterans(id),
  property_id: uuid REFERENCES properties(id),
  agent_id: uuid REFERENCES agents(id),
  application_date: date,
  status: enum('applied', 'approved', 'denied', 'withdrawn'),
  move_in_date: date,
  lease_terms: jsonb
}
```

### Forward Operating Capital (Financial Security)
**Purpose:** Coordinate veteran-specific financing through broker networks

**Components:**
- **Broker Platform Service:** Veteran client CRM, VA benefit optimization
- **Credit Management Service:** Credit monitoring, improvement tracking, debt analysis
- **Loan Coordination Service:** Application automation, rate optimization, approval tracking
- **Financial Planning Service:** Investment planning, business funding, retirement coordination

**Technical Stack:**
- **Backend:** Java Spring Boot for secure financial transaction processing
- **Database:** PostgreSQL with row-level security for financial data protection
- **Integrations:** VA loan APIs, credit bureau APIs, lender partner APIs
- **Security:** OAuth 2.0, encryption at rest and in transit, PCI DSS compliance

**Key Databases:**
```sql
-- Financial Services
veteran_financial_profiles {
  id: uuid PRIMARY KEY,
  veteran_id: uuid REFERENCES veterans(id),
  credit_score: integer,
  annual_income: decimal,
  debt_to_income_ratio: decimal,
  va_loan_entitlement: decimal,
  va_disability_rating: integer,
  employment_status: enum('employed', 'unemployed', 'retired', 'disabled'),
  banking_relationship: jsonb,
  investment_goals: text
}

loan_applications {
  id: uuid PRIMARY KEY,
  veteran_id: uuid REFERENCES veterans(id),
  broker_id: uuid REFERENCES brokers(id),
  loan_type: enum('va_home_loan', 'business_loan', 'auto_loan', 'personal_loan'),
  amount_requested: decimal,
  purpose: text,
  application_date: date,
  approval_date: date,
  funding_date: date,
  status: enum('draft', 'submitted', 'under_review', 'approved', 'denied', 'funded')
}

credit_monitoring {
  id: uuid PRIMARY KEY,
  veteran_id: uuid REFERENCES veterans(id),
  credit_score: integer,
  score_date: date,
  score_change: integer,
  report_summary: jsonb,
  improvement_recommendations: text
}
```

### Forward Operating Mobility (Transportation Independence)
**Purpose:** Ensure veteran vehicle access through dealership partnerships

**Components:**
- **Dealership Integration Service:** Veteran client CRM, vehicle matching
- **Fleet Management Service:** Operational vehicle tracking, maintenance scheduling
- **Vehicle Financing Service:** Coordinate with Forward Operating Capital for optimal rates
- **Emergency Transportation Service:** Crisis response vehicles, temporary solutions

**Technical Stack:**
- **Backend:** Node.js with real-time vehicle tracking capabilities
- **Database:** PostgreSQL with time-series data for vehicle performance tracking
- **Integrations:** Dealership CRM APIs, vehicle manufacturer APIs, GPS tracking
- **Mobile:** Flutter for cross-platform mobile access for dealership staff and veterans

**Key Databases:**
```sql
-- Vehicle & Transportation
vehicles {
  id: uuid PRIMARY KEY,
  vin: varchar(17) UNIQUE,
  make: varchar(50),
  model: varchar(50),
  year: integer,
  type: enum('personal', 'fleet_delivery', 'fleet_construction', 'emergency'),
  current_owner: uuid REFERENCES veterans(id),
  dealership_id: uuid REFERENCES dealerships(id),
  purchase_date: date,
  purchase_price: decimal,
  financing_terms: jsonb,
  insurance_info: jsonb
}

transportation_needs {
  id: uuid PRIMARY KEY,
  veteran_id: uuid REFERENCES veterans(id),
  need_type: enum('employment', 'housing_search', 'medical', 'family', 'emergency'),
  urgency: enum('immediate', 'within_week', 'within_month', 'planning'),
  vehicle_requirements: jsonb,
  budget_constraints: jsonb,
  timeline: daterange
}

fleet_operations {
  id: uuid PRIMARY KEY,
  vehicle_id: uuid REFERENCES vehicles(id),
  operation_type: enum('food_delivery', 'construction_transport', 'veteran_transport'),
  scheduled_start: timestamp,
  scheduled_end: timestamp,
  actual_start: timestamp,
  actual_end: timestamp,
  driver_id: uuid REFERENCES drivers(id),
  route_details: jsonb,
  fuel_used: decimal,
  mileage: decimal
}
```

### Forward Operating Fuel (Transportation Security)
**Purpose:** Coordinate fuel partnerships for delivery and construction operations

**Components:**
- **Partnership Management Service:** Corporate fuel partnerships, contract management
- **Route Optimization Service:** Fuel-efficient routing, delivery coordination
- **Fuel Tracking Service:** Usage monitoring, cost analysis, budget management
- **Emergency Fuel Service:** Crisis response fuel allocation, rapid deployment

**Technical Stack:**
- **Backend:** Python FastAPI for high-performance route optimization
- **Database:** TimescaleDB for time-series fuel consumption data
- **Integrations:** Fleet management APIs, fuel supplier APIs, mapping services
- **Analytics:** Machine learning for route optimization and fuel consumption prediction

**Key Databases:**
```sql
-- Fuel & Logistics
fuel_partnerships {
  id: uuid PRIMARY KEY,
  supplier_name: varchar(255),
  contact_info: jsonb,
  contract_terms: jsonb,
  discount_rate: decimal,
  coverage_area: geometry,
  services_provided: jsonb,
  partnership_start: date,
  partnership_end: date
}

fuel_transactions {
  id: uuid PRIMARY KEY,
  vehicle_id: uuid REFERENCES vehicles(id),
  supplier_id: uuid REFERENCES fuel_partnerships(id),
  transaction_date: timestamp,
  gallons: decimal,
  cost_per_gallon: decimal,
  total_cost: decimal,
  location: geometry,
  operation_type: enum('food_delivery', 'construction', 'veteran_transport', 'emergency')
}

route_optimizations {
  id: uuid PRIMARY KEY,
  operation_date: date,
  operation_type: enum('food_delivery', 'construction_transport', 'veteran_services'),
  planned_route: jsonb,
  actual_route: jsonb,
  estimated_fuel: decimal,
  actual_fuel: decimal,
  time_savings: interval,
  cost_savings: decimal
}
```

## Shared Services Architecture

### Veteran Profile Management
**Central veteran data repository accessible across all pillars**

```sql
veterans {
  id: uuid PRIMARY KEY,
  first_name: varchar(255),
  last_name: varchar(255),
  date_of_birth: date,
  ssn_hash: varchar(64), -- Hashed for privacy
  va_id: varchar(50),
  military_branch: enum('army', 'navy', 'air_force', 'marines', 'space_force', 'coast_guard'),
  service_period: daterange,
  disability_rating: integer,
  housing_status: enum('homeless', 'temporary', 'transitional', 'permanent', 'at_risk'),
  current_address: text,
  phone: varchar(20),
  email: varchar(255),
  emergency_contact: jsonb,
  consent_preferences: jsonb,
  created_at: timestamp,
  updated_at: timestamp
}

veteran_service_history {
  id: uuid PRIMARY KEY,
  veteran_id: uuid REFERENCES veterans(id),
  pillar: enum('table', 'housing', 'intel', 'capital', 'mobility', 'fuel'),
  service_type: varchar(255),
  service_date: date,
  outcome: text,
  satisfaction_rating: integer,
  notes: text
}
```

### Impact Tracking System
**Comprehensive metrics across all six pillars**

```sql
impact_metrics {
  id: uuid PRIMARY KEY,
  metric_type: enum('veteran_outcome', 'resource_utilization', 'partner_performance', 'financial_impact'),
  pillar: enum('table', 'housing', 'intel', 'capital', 'mobility', 'fuel'),
  measurement_date: date,
  metric_name: varchar(255),
  metric_value: decimal,
  unit_of_measurement: varchar(50),
  veteran_id: uuid REFERENCES veterans(id),
  partner_id: uuid,
  city_id: uuid REFERENCES cities(id)
}

city_dashboards {
  id: uuid PRIMARY KEY,
  city_id: uuid REFERENCES cities(id),
  reporting_period: daterange,
  veterans_served: integer,
  veterans_housed: integer,
  meals_provided: integer,
  pounds_waste_diverted: decimal,
  vehicles_provided: integer,
  total_partner_revenue: decimal,
  total_tax_benefits: decimal,
  community_impact_score: decimal
}
```

### Multi-City Management
**Support for licensing and franchising**

```sql
cities {
  id: uuid PRIMARY KEY,
  city_name: varchar(255),
  state: varchar(2),
  license_holder: varchar(255),
  license_type: enum('corporate', 'veteran_entrepreneur', 'nonprofit'),
  license_start_date: date,
  license_fee: decimal,
  royalty_rate: decimal,
  operational_status: enum('planning', 'launching', 'operational', 'expanding'),
  veteran_population: integer,
  target_veteran_serve: integer
}

pillar_deployments {
  id: uuid PRIMARY KEY,
  city_id: uuid REFERENCES cities(id),
  pillar: enum('table', 'housing', 'intel', 'capital', 'mobility', 'fuel'),
  deployment_status: enum('not_started', 'planning', 'development', 'testing', 'operational'),
  target_partners: integer,
  active_partners: integer,
  go_live_date: date,
  performance_metrics: jsonb
}
```

## Integration Layer

### Partner API Framework
**Standardized integration with existing partner systems**

**Restaurant POS Integration:**
- Square API for inventory and sales data
- Toast API for menu management and order tracking
- Clover API for payment processing and customer data
- Custom webhook handlers for real-time inventory updates

**Real Estate Platform Integration:**
- MLS APIs for property data and market information
- Zillow API for property valuation and market trends
- Realtor.com API for listing management and lead generation
- DocuSign API for lease agreement processing

**Financial Services Integration:**
- VA loan API for benefit verification and application processing
- Credit bureau APIs (Experian, Equifax, TransUnion) for credit monitoring
- Bank partner APIs for loan origination and account management
- Plaid API for secure bank account verification

**Vehicle and Transportation Integration:**
- Dealership CRM APIs for inventory and customer management
- Kelly Blue Book API for vehicle valuation
- AutoTrader API for vehicle listings and market data
- Fleet management APIs for GPS tracking and maintenance scheduling

### Real-Time Communication
**WebSocket connections for live coordination**

```javascript
// Real-time food delivery coordination
const deliverySocket = io('/delivery-coordination');

deliverySocket.on('food-available', (data) => {
  // Alert available drivers
  // Calculate optimal delivery route
  // Update shelter meal planning
});

deliverySocket.on('veteran-housing-match', (data) => {
  // Notify real estate agent
  // Prepare financing options
  // Schedule property showing
});
```

### Data Synchronization
**Ensure consistency across all pillars**

```python
# Cross-pillar data synchronization
class VeteranServiceCoordinator:
    def update_veteran_status(self, veteran_id, pillar, status_update):
        # Update veteran profile
        # Notify relevant pillars
        # Update impact metrics
        # Log audit trail
        
    def coordinate_services(self, veteran_id):
        # Check all pillar statuses
        # Identify coordination opportunities
        # Schedule cross-pillar actions
        # Monitor outcomes
```

## Security Architecture

### Data Protection
- **Encryption:** AES-256 for data at rest, TLS 1.3 for data in transit
- **Access Control:** Role-based permissions with principle of least privilege
- **Audit Logging:** Complete activity tracking for compliance and security monitoring
- **Privacy Controls:** Veteran consent management for data sharing between pillars

### Authentication & Authorization
```yaml
Authentication:
  - Veterans: Multi-factor authentication with phone/email verification
  - Partners: Business account verification with background checks
  - Staff: Privileged access with regular re-certification
  - API Access: OAuth 2.0 with scope-limited tokens

Authorization Roles:
  - Veteran Client: Access own data, request services, provide consent
  - Partner User: Access assigned veterans, update service status, view analytics
  - Pillar Administrator: Manage partner relationships, oversee operations
  - City Administrator: Manage all city operations, access aggregate data
  - System Administrator: Full system access for maintenance and support
```

### Compliance Framework
- **HIPAA Compliance:** Veteran health information protection
- **PCI DSS:** Financial transaction security
- **SOC 2 Type II:** Security and availability controls
- **VA Privacy Act:** Federal veteran data protection requirements

## Deployment Architecture

### Infrastructure Requirements
- **Cloud Provider:** AWS with multi-region deployment capability
- **Container Orchestration:** Kubernetes for scalable microservices deployment
- **Database:** Managed PostgreSQL with read replicas for performance
- **CDN:** CloudFront for global content delivery and performance
- **Monitoring:** Comprehensive application and infrastructure monitoring

### Scalability Design
```yaml
Performance Targets:
  - Response Time: <3 seconds for all user interactions
  - Throughput: 1,000+ concurrent users per city
  - Availability: 99.9% uptime with automated failover
  - Data Processing: <1 minute latency for cross-pillar coordination

Scaling Strategy:
  - Horizontal scaling for stateless services
  - Database read replicas for query performance
  - Auto-scaling based on demand patterns
  - Geographic distribution for multi-city deployment
```

### Disaster Recovery
- **Backup Strategy:** Daily automated backups with point-in-time recovery
- **Failover:** Automated failover to secondary regions within 5 minutes
- **Data Recovery:** Recovery point objective (RPO) of 1 hour, recovery time objective (RTO) of 4 hours
- **Business Continuity:** Manual processes documented for critical veteran services

## Mobile Architecture

### Cross-Platform Strategy
**React Native for unified development across iOS and Android**

```javascript
// Unified mobile app structure
FOBMobileApp/
├── src/
│   ├── veterans/        // Veteran client interface
│   ├── partners/        // Partner interface
│   ├── drivers/         // Delivery driver interface
│   ├── shared/          // Common components
│   └── navigation/      // App navigation
├── services/
│   ├── api/            // API integration
│   ├── auth/           // Authentication
│   ├── location/       // GPS and mapping
│   └── notifications/  // Push notifications
└── assets/
    ├── images/         // App images and icons
    └── fonts/          // Typography assets
```

### Mobile-Specific Features
- **Offline Capability:** Basic functionality during network issues
- **GPS Integration:** Real-time location for delivery and coordination
- **Camera Integration:** Photo documentation for food safety and construction
- **Push Notifications:** Time-sensitive alerts for coordination
- **Biometric Security:** Fingerprint/face authentication for veteran privacy

## Analytics & Business Intelligence

### Data Warehouse Architecture
```sql
-- Star schema for business intelligence
fact_veteran_services {
  id: bigint PRIMARY KEY,
  veteran_key: integer REFERENCES dim_veterans(veteran_key),
  service_date_key: integer REFERENCES dim_dates(date_key),
  pillar_key: integer REFERENCES dim_pillars(pillar_key),
  partner_key: integer REFERENCES dim_partners(partner_key),
  city_key: integer REFERENCES dim_cities(city_key),
  service_count: integer,
  service_cost: decimal,
  service_value: decimal,
  satisfaction_score: decimal
}

-- Aggregated impact metrics
fact_impact_metrics {
  id: bigint PRIMARY KEY,
  date_key: integer REFERENCES dim_dates(date_key),
  city_key: integer REFERENCES dim_cities(city_key),
  pillar_key: integer REFERENCES dim_pillars(pillar_key),
  veterans_served: integer,
  meals_provided: integer,
  pounds_diverted: decimal,
  veterans_housed: integer,
  vehicles_provided: integer,
  total_cost_savings: decimal,
  total_tax_benefits: decimal
}
```

### Real-Time Dashboards
- **City Operations Dashboard:** Real-time coordination across all pillars
- **Partner Performance Dashboard:** Service quality and efficiency metrics
- **Veteran Outcomes Dashboard:** Housing stability, food security, employment
- **Impact Reporting Dashboard:** Community impact and ROI measurement

## Development Standards

### Code Organization
```
forward-operating-base/
├── shared/                 // Shared libraries and utilities
│   ├── auth/              // Authentication services
│   ├── data/              // Database models and migrations
│   ├── api/               // Common API utilities
│   └── utils/             // Shared business logic
├── pillars/
│   ├── table/             // Food security services
│   ├── housing/           // Housing coordination services
│   ├── intel/             // Real estate intelligence
│   ├── capital/           // Financial services
│   ├── mobility/          // Transportation services
│   └── fuel/              // Logistics coordination
├── mobile/                // React Native mobile app
├── web/                   // Web dashboard and admin
├── docs/                  // Technical documentation
└── deployments/           // Infrastructure as code
```

### Development Workflow
1. **Feature Planning:** Define requirements in memory-bank documentation
2. **API Design:** Document all endpoints before implementation
3. **Database Migration:** Version-controlled schema changes
4. **Unit Testing:** Comprehensive test coverage for business logic
5. **Integration Testing:** Cross-pillar coordination testing
6. **Security Review:** Regular security assessment and penetration testing
7. **Performance Testing:** Load testing before deployment
8. **Documentation Update:** Keep architecture docs current with changes

---

**Architecture Version:** 1.0  
**Last Updated:** September 2025  
**Review Schedule:** Monthly during development, quarterly during operations  
**Architecture Review Board:** CTO, Lead Engineers, Security Officer, Operations Manager`
}