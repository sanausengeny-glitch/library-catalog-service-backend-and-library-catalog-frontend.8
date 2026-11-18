# Library Catalog Service Backend and Frontend Admin Pages

Repository name: `library-catalog-service-backend-and-library-catalog-frontend.8`

This project provides a React catalog and an Express API for library books, borrowing, and space reservations.

## Setup

1. Install backend dependencies with `npm install`.
2. Copy `.env.example` to `.env` and set `MONGODB_URI`.
3. Seed the database with `npm run seed`.
4. Start the API with `npm start`.
5. In `client/`, install dependencies with `npm install`, then start the frontend with `npm run dev`.

The API listens on port `3000` by default and the Vite frontend proxies API requests to it.

## Borrow requests and fines

- A signed-in patron can submit a borrow request for a book on their reading list using `POST /api/borrow-requests` with a `bookId`.
- Requests start as `pending` and receive a due date 14 days after they are requested.
- Only a librarian can list requests or change a request's status with `PATCH /api/borrow-requests/:id`.
- A book with zero available copies is rejected by the server.
- When a librarian changes a request to `returned`, the server records the return time and calculates the fine. The rate is **$0.25 per overdue day**, charging each started 24-hour day. The amount is rounded to cents; any client-provided fine is ignored.

## Space bookings

- Signed-in patrons create bookings with `POST /api/bookings`, sending a space, date, start time, and end time.
- Before saving, the server checks existing bookings with the same space and date. A new booking conflicts if its start is earlier than an existing booking's end **and** its end is later than that booking's start. This catches partial overlaps as well as fully contained ranges; bookings that only touch at an endpoint are allowed.
- The librarian-only `GET /api/bookings` endpoint lists all bookings.

## Librarian admin pages

After signing in with a librarian account, the navigation provides:

- `/admin/books` — add, edit, and delete books through the Book API.
- `/admin/borrow-requests` — view requests and update their status; returned requests display the server-calculated fine.
- `/admin/bookings` — view all space bookings.

These pages show loading, error, and empty states and communicate with the backend rather than only changing local UI state.

## Seeded librarian account

- Username: `librarian1`
- Email: `librarian@library.test`
- Password: `LibrarianPass123!`

## API conventions

Authenticated routes accept the JWT from the `token` httpOnly cookie or an `Authorization: Bearer <token>` header. Successful JSON responses use `{ "success": true, "data": ... }`; errors include a message in `error.message`.

The catalog's existing `/books` and `/genres` routes remain available, with `/api/books` and `/api/genres` aliases for API clients.
