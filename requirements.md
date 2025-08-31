# Backend Feature Specifications
 - User Authentication
 - Property Management
 - Booking Management
## 1. User Authentication

### Overview
This module handles user registration, login, and authentication. It ensures secure access to the platform for both guests and hosts.

### API Endpoints

#### Register User
- **Endpoint:** `POST /api/auth/register`
- **Input (JSON):**
  ```json
  {
    "name": "John Doe",
    "email": "john@example.com",
    "password": "securePassword123"
  }
  ```

- **Validation Rules:**
  - `name`: required, min 2 chars, max 50 chars
  - `email`: required, valid format, unique in system
  - `password`: required, min 8 chars, must contain at least 1 letter and 1 number

- **Output (JSON):**
  ```json
  {
    "id": 1,
    "name": "John Doe",
    "email": "john@example.com",
    "created_at": "2025-08-28T12:00:00Z"
  }
  ```

#### Login User
- **Endpoint:** `POST /api/auth/login`
- **Input (JSON):**
  ```json
  {
    "email": "john@example.com",
    "password": "securePassword123"
  }
  ```

- **Output (JSON):**
  ```json
  {
    "access_token": "jwt-token-here",
    "token_type": "Bearer",
    "expires_in": 3600
  }
  ```

- **Validation Rules:**
  - Credentials must match a registered user
  - Return 401 Unauthorized if invalid

#### Get Current User (Protected)
- **Endpoint:** `GET /api/auth/me`
- **Headers:** `Authorization: Bearer <jwt-token>`
- **Output (JSON):**
  ```json
  {
    "id": 1,
    "name": "John Doe",
    "email": "john@example.com"
  }
  ```

### Performance Criteria
- Passwords must be hashed using bcrypt or Argon2
- Token validation should occur in < 50ms per request
- Authentication service must handle 100 concurrent logins per second

---

## 2. Property Management

### Overview
This module allows hosts to create, update, and manage their property listings, including descriptions, pricing, and locations.

### API Endpoints

#### Create Property
- **Endpoint:** `POST /api/properties`
- **Input (JSON):**
  ```json
  {
    "title": "Luxury Apartment",
    "description": "2-bedroom apartment in Lagos",
    "price_per_night": 150,
    "location_id": 2,
    "user_id": 1
  }
  ```

- **Validation Rules:**
  - `title`: required, max 100 chars
  - `description`: required, max 500 chars
  - `price_per_night`: required, numeric, min 1
  - `location_id`: must exist in locations table
  - `user_id`: must exist and be a host

- **Output (JSON):**
  ```json
  {
    "id": 10,
    "title": "Luxury Apartment",
    "price_per_night": 150,
    "location": "Lagos",
    "host": "John Doe"
  }
  ```

#### Get All Properties
- **Endpoint:** `GET /api/properties`
- **Output (JSON):**
  ```json
  [
    {
      "id": 10,
      "title": "Luxury Apartment",
      "price_per_night": 150,
      "location": "Lagos"
    },
    {
      "id": 11,
      "title": "Beach House",
      "price_per_night": 300,
      "location": "Accra"
    }
  ]
  ```

- **Filters:**
  - `?location=Lagos`
  - `?price_min=100&price_max=200`
  - `?sort=price_asc`

#### Update Property
- **Endpoint:** `PUT /api/properties/{id}`
- **Input (JSON):**
  ```json
  {
    "price_per_night": 180,
    "description": "Updated description"
  }
  ```

- **Output (JSON):**
  ```json
  {
    "id": 10,
    "price_per_night": 180,
    "description": "Updated description"
  }
  ```

### Performance Criteria
- Must handle search/filter in < 500ms for 10k properties
- Properties should be indexed by location and price for fast queries
- A host cannot own more than 100 active listings (validation rule)

---

## 3. Booking System

### Overview
This module manages property reservations, ensuring that users can book properties without conflicts and with secure payment tracking.

### API Endpoints

#### Create Booking
- **Endpoint:** `POST /api/bookings`
- **Input (JSON):**
  ```json
  {
    "user_id": 2,
    "property_id": 10,
    "start_date": "2025-09-05",
    "end_date": "2025-09-10"
  }
  ```

- **Validation Rules:**
  - `user_id`: must exist, must not be the property owner
  - `property_id`: must exist and be available
  - Dates: start_date < end_date, cannot overlap with existing confirmed bookings

- **Output (JSON):**
  ```json
  {
    "id": 100,
    "status": "pending",
    "user": "Jane Doe",
    "property": "Luxury Apartment",
    "start_date": "2025-09-05",
    "end_date": "2025-09-10"
  }
  ```

#### Confirm Booking (with Payment)
- **Endpoint:** `POST /api/bookings/{id}/confirm`
- **Input (JSON):**
  ```json
  {
    "payment_method": "credit_card",
    "amount": 750
  }
  ```

- **Output (JSON):**
  ```json
  {
    "id": 100,
    "status": "confirmed",
    "payment_status": "successful"
  }
  ```

#### Get User Bookings
- **Endpoint:** `GET /api/users/{id}/bookings`
- **Output (JSON):**
  ```json
  [
    {
      "id": 100,
      "property": "Luxury Apartment",
      "status": "confirmed",
      "start_date": "2025-09-05",
      "end_date": "2025-09-10"
    }
  ]
  ```

### Performance Criteria
- Must prevent double-booking via database transaction locks
- Response time for booking check < 200ms
- System should scale to 1,000 concurrent bookings without errors
