# 📘 Use Case Documentation – Property Booking Platform

This document provides a developer-oriented overview of the core use cases for the Property Booking Platform. It is derived from the features and functionality architecture and is intended to guide system design, implementation, and validation.

---

## Actors

| Actor | Description |
|-------|--------------|
| **Guest** | User interested in browsing and booking listed properties. |
| **Host** | Property owner/manager responsible for creating and managing listings. |
| **Admin** | Platform administrator responsible for monitoring and managing users & listings. |

---

## Use Cases Overview

The platform’s use cases are grouped into the following domains:

1. **User Management**
2. **Property Listing Management**
3. **Search & Booking Management**
4. **Payment Integration**
5. **Reviews & Ratings**
6. **Admin Dashboard Management**

---

## Use Case Specifications

### 1. User Management

#### 1.1 Register User
**Primary Actor:** Guest / Host / Admin  
**Trigger:** User chooses to sign up.  

**Preconditions:**
- User is not already registered.

**Postconditions:**
- New user profile is created and stored.
- Verification email/SMS may be sent.

**Basic Flow:**
1. User submits registration details.
2. System validates input.
3. System stores user details and assigns a role.
4. Confirmation message returned to the user.

**Alternative Flow:**
- Invalid details → System displays error.

---

#### 1.2 Login User
**Primary Actor:** Guest / Host / Admin  
**Trigger:** User attempts to sign in.

**Preconditions:**
- User must have registered.
- Credentials must match stored records.

**Postconditions:**
- User is authenticated and granted access.

**Basic Flow:**
1. User enters email and password.
2. System verifies credentials.
3. User is granted access to their dashboard.

**Alternate Flow:**
- Wrong credentials → Access denied.

---

### 2. Property Listing Management (Host)

#### 2.1 Add Property
**Primary Actor:** Host  
**Trigger:** Host selects "Add Property".

**Preconditions:**
- Host must be logged in.

**Postconditions:**
- New property listing is created and stored.

**Basic Flow:**
1. Host provides property details (images, description, pricing, location, amenities).
2. System validates details.
3. System saves property and sets listing to *active*.

---

#### 2.2 Edit Property
**Primary Actor:** Host  
**Trigger:** Host selects a property to edit.

**Preconditions:**
- Host owns the listing.

**Postconditions:**
- Property details updated.

---

#### 2.3 Delete Property
**Primary Actor:** Host  
**Trigger:** Host selects "Delete Property".

**Preconditions:**
- Host must own the property.

**Postconditions:**
- Property is removed from listings.

---

### 3. Search & Booking Management (Guest)

#### 3.1 Search Properties
**Primary Actor:** Guest  
**Trigger:** Guest opens search page or submits filters.

**Basic Flow:**
1. Guest enters search query or applies filters.
2. System fetches matching listings.
3. System displays results.

---

#### 3.2 Book Property
**Primary Actor:** Guest  
**Trigger:** Guest selects property and proceeds to booking.

**Preconditions:**
- Guest must be logged in.

**Postconditions:**
- Booking record created with *pending payment* status.

**Basic Flow:**
1. Guest selects dates and property.
2. System checks availability.
3. System creates booking record.

---

### 4. Payment Integration

#### 4.1 Make Payment
**Primary Actor:** Guest  
**Trigger:** Guest confirms booking payment.

**Preconditions:**
- Booking exists with pending status.

**Postconditions:**
- Payment recorded.
- Booking status updated to *confirmed*.

**Basic Flow:**
1. System redirects to payment gateway.
2. Payment Service processes payment.
3. System verifies payment response.
4. Booking marked as confirmed.

**Alternate Flow:**
- Payment failure → Booking remains pending or canceled.

---

### 5. Reviews & Ratings

#### 5.1 Post Review and Rating (Guest)
**Primary Actor:** Guest  
**Trigger:** Guest selects to add review after completing stay.

**Preconditions:**
- Guest has a confirmed and completed booking.

**Postconditions:**
- Review stored and visible on property page.

---

#### 5.2 Reply to Reviews (Host)
**Primary Actor:** Host  
**Trigger:** Host chooses to reply to a review.

**Preconditions:**
- Guest review exists for the property.

**Postconditions:**
- Response saved and displayed.

---

### 6. Admin Dashboard Management

#### 6.1 Manage Users
**Primary Actor:** Admin  
**Trigger:** Admin selects "Manage Users".

**Basic Flow:**
- Admin can view, suspend, activate, or delete users.

---

#### 6.2 Manage Properties
**Primary Actor:** Admin, Hosts  
**Trigger:** Admin selects "Manage Properties".

**Basic Flow:**
- Admin can approve, suspend, or remove property listings.

---

## System Cross-Cutting Concerns

| Concern | Description |
|--------|--------------|
| **Security & Authentication** | Role-based access: Guest, Host, Admin |
| **Data Validation & Sanitization** | Mandatory for user and property input |
| **Notifications** | On booking, payment, and review events |
| **Logging & Monitoring** | For audit & analytics |

---
