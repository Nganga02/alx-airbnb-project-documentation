# Airbnb Clone Backend – Features & Functionalities

This document outlines the core features and functionalities that the backend of the Airbnb Clone must support. It serves as a high-level system blueprint to guide development before coding begins. The goal is to clearly define what the system must do, for whom, and under what conditions.

---

## 1. User Management & Authentication

### Core Features
- **User Registration (Sign Up)**  
  Users can create accounts by providing required personal details.
- **User Login (Authentication)**  
  Secure login using email and password.
- **Password Encryption**  
  User passwords must be stored using a secure hashing method (e.g., bcrypt).
- **User Roles**  
  System supports multiple user roles:
  - Guest (Default)
  - Host
  - Admin
- **Profile Management**  
  Users can update profile information (phone number, name, etc.).
- **Session & Token Management**  
  Use JWT for secure access to protected routes.

---

## 2. Property Management (Host Functionality)

### Core Features
- **Add New Property Listing**  
  Hosts can list properties with details such as name, description, location, and price.
- **Update Property Information**  
  Ability to modify listing details.
- **Delete Property**  
  Hosts may remove their own listings.
- **View Own Listed Properties**  
  Hosts can view and manage all their listings.
- **Property Search & Browsing**  
  All users can view available properties with filtering and sorting options (price, location, rating, etc.)

---

## 3. Search and filtering (System functionality for guests)

### Core functionalities
- **Guests can implement searches for properties using filters**
  Guests can use filters like location, price, amenities, number of guests to search for the preferred arpartment
- **Optimising searches**
  The system should optimise search criteria for the users

--- 

## 4. Booking System

### Core Features
- **Create a Booking**  
  Guests can book a property for specific dates.
- **Check Availability**  
  System must prevent double-booking of the same dates for the same property.
- **Manage Bookings**  
  Guests and Hosts can view booking details.
- **Modify or Cancel Booking**  
  Must follow rules based on booking status.
- **Booking Statuses**
  - Pending  
  - Confirmed  
  - Canceled

---

## 5. Payments

### Core Features
- **Payment Processing Integration**  
  Handle payments using providers such as PayPal, Stripe, or Credit Card.
- **Payment Validation**  
  Only valid bookings can be paid for.
- **Payment Record Keeping**  
  System stores amount, payment method, and timestamps.
- **Refund Handling (Future Enhancement)**  
  Optional future feature for cancellations.
- **Support multiple currencies**
  System should support payments using different currencies
- **Implementing secure payment gateways**
  This should handle: upfront payments by guests and automatic payouts to hosts after a booking is completed.

---

## 6. Reviews & Ratings

### Core Features
- **Leave a Review**  
  Guests can review properties after a completed stay. Should be linked to a booking to prevent abuse.
- **Rating System**  
  Ratings on a scale of 1 to 5.
- **View Reviews**  
  Users can read reviews for each property.
- **Host Response**  
  Hosts can reply to reviews (optional future feature).
- **Link reviews to a booking**
  The system should link reviews to a system to prevent abuse.

---

## 7. Notification system

### Core functionality
- **e-mail and in-app notification**
  The system should implement notifications for:
    - booking confirmation
    - cancellation
    - payment updates
### Advanced functionalities
- **SMS notification**
  Hosts can receive offline sms of booking confirmed and payment receipts.

---

## 8. Admin dashboard

### Core functionality
- **Monitoring and managing**
  The admin user should be to monitor and manage:
    + Users 
    + Listing
    + Booking
    + Payments

---

## 7. Messaging System(Additional functionality)

### Core Features
- **User-to-User Messaging**  
  Guests and Hosts can communicate within the platform.
- **Send & Receive Messages**  
  Store sender, receiver, and message body.
- **Message History**  
  Retrieve list of messages between two users.

---

## 8. Security & Access Control

### Core Features
- **Role-Based Access Control**  
  Restrict actions based on user role.
- **Authentication Required for Protected Actions**  
  Such as booking, listing property, messaging, etc.
- **Input Validation & Sanitization**  
  Protect against security vulnerabilities (XSS, SQL injection).

---

## Summary Table

| Feature Category       | Key Actors             | Core Objective                                      |
|------------------------|-------------------------|----------------------------------------------------|
| User Management        | Guest, Host, Admin      | Secure access & identity management               |
| Property Listing Management     | Host                    | Publish, update & manage property listings         |
| Booking System          | Guest, Host             | Make and manage reservations                        |
| Payments                | Guest                   | Process secure payments                             |
| Reviews                 | Guest, Host             | Rate and assess stays                               |
| Notification            | Guest, Host             | e-mail and in-app notifications                     |
| Admin dashboard         | Admin                   | monitoring and managening modules                   |
| Messaging               | Guest, Host             | Direct communication                                |
| Security                | All Users               | Protect system and ensure safe access               |
---


