# Airbnb Clone Backend - Feature Requirements Specification
## 1. User Authentication System
### Functional Requirements
- Users can register with email/password or OAuth providers (Google, Facebook)
- Users can log in and receive a JWT token
- Users can reset forgotten passwords
- Users can update their profiles
- JWT tokens expire after 24 hours and can be refreshed

### API Endpoints
#### POST /api/auth/register
#### Input:
```json55
{
"email": "user@example.com",
"password": "SecurePassword123!",
"firstName": "John",
"lastName": "Doe",
"role": "guest" // or "host"
}
```

#### Output (Success - 201 Created):
```json5
{
"message": "User registered successfully",
"userId": "12345",
"token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

#### Output (Error - 400 Bad Request):
```json5
{
"error": "Validation failed",
"details": ["Email is required", "Password must be at least 8 characters"]
}
```

#### POST /api/auth/login
#### Input:
```json5
{
"email": "user@example.com",
"password": "SecurePassword123!"
}
```

#### Output (Success - 200 OK):
```json5
{
"message": "Login successful",
"token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
"user": {
"id": "12345",
"email": "user@example.com",
"firstName": "John",
"lastName": "Doe",
"role": "guest"
 }
}
```

#### POST /api/auth/forgot-password
#### Input:
```json5
{
"email": "user@example.com"
}
```

#### Output (Success - 200 OK):
```json5
{
"message": "Password reset email sent"
}
```

### Validation Rules
- Email must be valid format
- Password must be at least 8 characters with at least one uppercase, one lowercase, one number, and one special character
- First and last name must be at least 2 characters
- Role must be either "guest" or "host"

### Performance Criteria
- Registration API response time < 500ms
- Login API response time < 300ms
- Support up to 1000 concurrent authentication requests
- Password hashing using bcrypt with salt rounds = 12

---

## 2. Property Management System
### Functional Requirements
- Hosts can create, read, update, and delete property listings
- Properties have details (title, description, location, price, amenities, availability)
- Guests can search and filter properties
- Property images are stored in cloud storage
- Only property owners can modify their listings

### API Endpoints
#### POST /api/properties
#### Input:
```json5
{
"title": "Beachfront Villa with Ocean View",
"description": "Beautiful villa with private beach access...",
"location": {
"address": "123 Beach Road",
"city": "Miami",
"state": "FL",
"country": "USA",
"zipCode": "33139"
},
"pricePerNight": 250,
"bedroomCount": 3,
"bathroomCount": 2,
"maxGuests": 6,
"amenities": ["wifi", "pool", "airConditioning", "kitchen"],
"availability": [
{
"startDate": "2023-08-01",
"endDate": "2023-08-31"
}
]
}
```

#### Headers:
```text
Authorization: Bearer <JWT_TOKEN>
```

#### Output (Success - 201 Created):
```json5
{
"message": "Property created successfully",
"propertyId": "prop_12345"
}
```

#### GET /api/properties?location=Miami&minPrice=100&maxPrice=300&guests=4&amenities=wifi,pool
#### Output (Success - 200 OK):
```json5
{
"properties": [
{
"id": "prop_12345",
"title": "Beachfront Villa with Ocean View",
"location": {
"city": "Miami",
"state": "FL"
},
"pricePerNight": 250,
"bedroomCount": 3,
"bathroomCount": 2,
"maxGuests": 6,
"amenities": ["wifi", "pool", "airConditioning", "kitchen"],
"rating": 4.8,
"reviewCount": 42,
"images": ["https://bucket.s3.amazonaws.com/image1.jpg"]
    }
],
"pagination": {
"total": 125,
"page": 1,
"limit": 20,
"totalPages": 7
 }
}
```

#### PUT /api/properties/:id
#### Input:
```json5
{
"pricePerNight": 275,
"amenities": ["wifi", "pool", "airConditioning", "kitchen", "parking"]
}
```

#### Output (Success - 200 OK):
```json5
{
"message": "Property updated successfully"
}
```

### Validation Rules
- Title must be between 10-100 characters
- Description must be between 50-2000 characters
- Price must be a positive number
- Location must include valid address components
- Amenities must be from predefined list
- Availability dates must be valid and not in the past

### Performance Criteria
- Property search response time < 300ms with filters
- Property creation API response time < 800ms (including image processing)
- Support up to 5000 property records with efficient indexing
- Image uploads optimized with compression and resizing

---

## 3. Booking System
### Functional Requirements
- Guests can book available properties for specific dates
- Prevent double bookings with date validation
- Guests and hosts can cancel bookings according to policy
- Booking status transitions (pending, confirmed, canceled, completed)
- Automatic payout to hosts after completed stays

### API Endpoints
#### POST /api/bookings
#### Input:
```json5
{
"propertyId": "prop_12345",
"checkInDate": "2023-08-15",
"checkOutDate": "2023-08-20",
"guestCount": 4,
"totalPrice": 1250,
"paymentMethodId": "pm_1ABCDEFGHIJK" // From Stripe
}
```

#### Headers:
```text
Authorization: Bearer <JWT_TOKEN>
```

#### Output (Success - 201 Created):
```json5
{
"message": "Booking created successfully",
"bookingId": "book_67890",
"status": "confirmed",
"paymentStatus": "succeeded"
}
```

#### GET /api/bookings/:id
#### Output (Success - 200 OK):
```json5
{
"id": "book_67890",
"propertyId": "prop_12345",
"guestId": "user_12345",
"checkInDate": "2023-08-15",
"checkOutDate": "2023-08-20",
"guestCount": 4,
"totalPrice": 1250,
"status": "confirmed",
"paymentStatus": "succeeded",
"createdAt": "2023-07-10T14:30:00Z",
"propertyDetails": {
"title": "Beachfront Villa with Ocean View",
"images": ["https://bucket.s3.amazonaws.com/image1.jpg"]
  }
}
```

#### POST /api/bookings/:id/cancel
#### Output (Success - 200 OK):
```json5
{
"message": "Booking canceled successfully",
"refundAmount": 1125, // Depending on cancellation policy
"cancellationFee": 125
}
```

### Validation Rules
- Check-in date must be at least 24 hours in the future
- Check-out date must be after check-in date
- Guest count must not exceed property maximum
- Total price must match calculated (nights × pricePerNight)
- Property must be available for the selected dates

### Business Rules
- 100% refund if canceled 48+ hours before check-in
- 50% refund if canceled 24-48 hours before check-in
- No refund if canceled less than 24 hours before check-in
- Hosts receive payout 24 hours after successful check-out
- Automatic status change to "completed" after check-out date
- Performance Criteria
- Booking creation API response time < 600ms (including payment processing)
- Date availability check response time < 100ms
- Support up to 100 concurrent booking requests
- Payment processing failure rate < 0.1%

---

## Additional Technical Specifications
### Database Schema Highlights
#### Users Table:
- `id`, `email`, `password_hash`, `first_name`, `last_name`, `role`, `created_at`, `updated_at`

#### Properties Table:
- `id`,`host_id`, `title`, `description`, `address`, `city`, `state`, `country`, `zip_code`, `price_per_night`, `bedroom_count`, `bathroom_count`, `max_guests`, `amenities`, `created_at`, `updated_at`

#### Bookings Table:
- `id`, `property_id`, `guest_id`, `check_in_date`, `check_out_date`, `guest_count`, `total_price`, `status`, `payment_status`, `created_at`, `updated_at`


### Security Measures
- JWT tokens with 24-hour expiration
- Password hashing with bcrypt
- SQL injection prevention with parameterized queries
- XSS protection with input sanitization
- Rate limiting on authentication endpoints
- Payment data never stored (using Stripe tokens)

### Error Handling
- Consistent error response format
- Appropriate HTTP status codes
- Detailed error messages for client-side handling
- Logging of all errors for monitoring

