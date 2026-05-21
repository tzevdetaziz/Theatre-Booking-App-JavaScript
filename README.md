# Theatre Booking System

A complete prototype for a mobile and distributed theatre reservation platform developed for **CN6035 – Mobile & Distributed Systems**.

The project includes:

- **Mobile frontend** built with **Expo / React Native**
- **REST API backend** built with **Express.js**
- **MariaDB / MySQL** relational database
- **JWT authentication** with access and refresh tokens
- **Reservation management** with seat selection and protected user routes

---

## 1. Project Overview

The system allows a user to:

- create an account
- log in securely
- browse available theatres
- browse shows and showtimes
- view available seats for a selected showtime
- create reservations
- update or cancel existing reservations
- view profile information
- view their own reservation history

The aim of the project is to demonstrate a working **distributed application** with a clear separation between:

- frontend presentation layer
- backend API/business logic layer
- database persistence layer

---

## 2. Technology Stack

### Frontend
- Expo
- React Native
- React Navigation
- Axios

### Backend
- Node.js
- Express.js
- express-validator
- bcryptjs
- jsonwebtoken
- mysql2
- cors
- dotenv

### Database
- MariaDB / MySQL

---

## 3. Project Structure

```text
theatre-booking-fullstack/
│
├── backend/
│   ├── src/
│   │   ├── config/
│   │   ├── controllers/
│   │   ├── middleware/
│   │   ├── routes/
│   │   ├── services/
│   │   ├── utils/
│   │   ├── app.js
│   │   └── server.js
│   ├── .env.example
│   └── package.json
│
├── mobile/
│   ├── src/
│   │   ├── components/
│   │   ├── context/
│   │   ├── navigation/
│   │   ├── screens/
│   │   └── services/
│   └── package.json
│
├── database/
│   ├── schema.sql
│   └── seed_demo.sql
│
├── docs/
├── postman/
└── README.md
```

---

## 4. Main Features

### Authentication
- user registration
- user login
- JWT access token generation
- refresh token support
- logout handling
- protected profile and reservation routes

### Theatre Browsing
- list available theatres
- filter theatres by name or location

### Show Discovery
- list shows
- filter by theatre
- filter by title
- filter by date

### Showtime and Seats
- retrieve showtimes for a selected show
- retrieve seat layout for a selected showtime
- identify reserved vs available seats

### Reservation Management
- create reservation for selected seats
- update a future reservation
- cancel a future reservation
- view current user's reservations

---

## 5. Database Design

The core database tables are:

- `users`
- `refresh_tokens`
- `theatres`
- `shows`
- `showtimes`
- `showtime_seats`
- `reservations`
- `reservation_seats`

### Relationship summary
- one **user** can have many **reservations**
- one **theatre** can host many **shows**
- one **show** can have many **showtimes**
- one **showtime** can have many **showtime seats**
- one **reservation** can include many seats through `reservation_seats`

This schema supports both authentication and the reservation lifecycle.

---

## 6. Backend Setup

### Requirements
Before running the backend, make sure you have installed:

- Node.js
- MariaDB or MySQL server

### Install backend dependencies
Open a terminal inside `backend/` and run:

```bash
npm install
```

### Configure environment variables
Create a `.env` file inside `backend/`.

Example:

```env
PORT=5000
DB_HOST=localhost
DB_PORT=3306
DB_NAME=theatre_booking
DB_USER=root
DB_PASSWORD=0000
JWT_ACCESS_SECRET=replace_with_a_long_random_secret
JWT_REFRESH_SECRET=replace_with_another_long_random_secret
ACCESS_TOKEN_EXPIRES_IN=15m
REFRESH_TOKEN_EXPIRES_IN_DAYS=7
CORS_ORIGIN=*
```

> Adjust `DB_PASSWORD` to match your own MariaDB / MySQL root password or database user password.

### Start backend
```bash
npm run dev
```

If everything is correct, the terminal should show messages similar to:

```text
MariaDB connection established.
Server listening on port 5000
```

### Health check
You can test that the backend is running at:

```text
http://localhost:5000/health
```

Expected response:

```json
{
  "success": true,
  "message": "API is running."
}
```

---

## 7. Database Setup

### Create the database
If the database does not already exist, create it:

```sql
CREATE DATABASE theatre_booking CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

### Import schema
Run the schema file into the `theatre_booking` database.

Example with MySQL CLI on Windows:

```powershell
Get-Content .\database\schema.sql | & "C:\Program Files\MySQL\MySQL Server 8.0\bin\mysql.exe" -u root -p theatre_booking
```

### Import seed data
To populate the UI with theatres, shows, showtimes and seats:

```powershell
Get-Content .\database\seed_demo.sql | & "C:\Program Files\MySQL\MySQL Server 8.0\bin\mysql.exe" -u root -p theatre_booking
```

After importing the seed file, the frontend will display realistic demo content.

---

## 8. Mobile Frontend Setup

### Install mobile dependencies
Open a terminal inside `mobile/` and run:

```bash
npm install
```

### Recommended additional Expo packages
If required by your local environment, install:

```bash
npx expo install expo-asset expo-secure-store react-dom react-native-web @expo/metro-runtime
```

### Start the frontend
```bash
npm start
```

or with clean cache:

```bash
npx expo start -c
```

---

## 9. API Base URL Notes

The frontend uses different API base URLs depending on platform.

### Web
```text
http://localhost:5000
```

### Android Emulator
```text
http://10.0.2.2:5000
```

### Real Device
If running from a physical phone on the same Wi-Fi network, use your computer's LAN IP, for example:

```text
http://192.168.1.63:5000
```

This setting can be adjusted in:

```text
mobile/src/services/api.js
```

---

## 10. Example API Endpoints

### Public Routes
- `GET /health`
- `POST /auth/register`
- `POST /auth/login`
- `POST /auth/refresh`
- `POST /auth/logout`
- `GET /theatres`
- `GET /shows`
- `GET /showtimes`
- `GET /seats?showtimeId=1`

### Protected Routes
- `GET /user/profile`
- `GET /user/reservations`
- `POST /reservations`
- `PUT /reservations/:id`
- `DELETE /reservations/:id`

---

## 11. Example User Flow

A typical user flow is:

1. Register a new account
2. Log in
3. Browse theatres
4. Select a show
5. View available showtimes
6. Choose seats
7. Create reservation
8. View reservation in **My Reservations**
9. Update or cancel reservation if the showtime is still in the future

---

## 12. Authentication and Security

The project includes several security-related design decisions:

- passwords are hashed with **bcrypt**
- protected routes require **JWT bearer token**
- refresh tokens are stored and managed separately
- logout revokes refresh token usage
- request payloads are validated with **express-validator**
- reservation logic prevents invalid seat selection

---

## 13. Reservation Logic

The reservation workflow is one of the most important parts of the system.

When a reservation is created:

1. the user must be authenticated
2. the backend checks that the selected seats exist
3. the backend checks that the seats belong to the selected showtime
4. the backend verifies that the seats are not already reserved
5. the total reservation cost is calculated
6. the reservation is inserted into the database
7. selected seats are marked as reserved

The update and cancellation logic only applies to **future** reservations, which protects the consistency of historical booking data.

---

## 14. Testing Summary

The following functional checks were performed during development:

- backend connection to MariaDB
- health endpoint response
- registration via API
- login via API and frontend
- retrieval of theatres, shows, showtimes and seats
- retrieval of user profile
- protected route access using token
- frontend rendering in web mode
- fixes for token storage behavior in web
- seed import to enrich UI content

---

## 15. Known Limitations

This project is a coursework prototype, so some production-level features are intentionally out of scope.

Current limitations include:

- no payment integration
- no admin dashboard
- no email verification
- no password reset
- no advanced concurrency protection beyond prototype-level transaction logic
- UI styling is functional rather than fully polished

---

## 16. Possible Future Improvements

Possible future enhancements include:

- payment gateway integration
- administrator panel for managing theatres and shows
- image upload and media management
- better seat map visualization
- email notifications and booking confirmation
- automated testing
- deployment to cloud infrastructure
- stronger concurrency control for high-volume booking scenarios

---

## 17. Demo Credentials

You may register your own account through the UI.

Example test user used during development:

```text
Email: test@example.com
Password: 12345678
```

---

## 18. Conclusion

Theatre Booking System is a complete distributed prototype that demonstrates:

- mobile frontend development
- REST API implementation
- JWT authentication
- relational database modeling
- reservation business logic
- frontend-backend integration


---
