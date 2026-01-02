# Airbnb / Wanderlust

A full-stack Airbnb-style listings application built with Express 5, MongoDB, EJS views, Mapbox maps, and Cloudinary media storage. Users can sign up, log in, create listings with geocoded locations, upload photos, and collect reviews.

## Features

- User signup/login with Passport (local strategy) and persistent Mongo-backed sessions.
- Listings CRUD with owner-only edit/delete, image uploads to Cloudinary, and server-side validation via Joi.
- Geocoding of listing locations using Mapbox; map display on listing pages with pins and popups.
- Reviews with rating + comment, author-only deletion, and cascading cleanup when a listing is removed.
- Flash messaging, form validation helpers, and centralized error handling.

## Tech Stack

- Runtime: Node.js 22 (see `engines` in `package.json`).
- Server: Express 5, EJS + ejs-mate layouts, method-override.
- Data: MongoDB (Mongoose 9 models for users, listings, reviews) with connect-mongo session store.
- Auth: passport, passport-local, passport-local-mongoose.
- Uploads: multer + multer-storage-cloudinary -> Cloudinary.
- Maps/Geocoding: Mapbox SDK + client-side mapbox-gl.
- Validation & utilities: Joi, custom middleware, ExpressError wrapper.

## Project Structure

```
app.js                # Express app entry
cloudConfig.js        # Cloudinary + multer storage
middleware.js         # Auth/ownership/validation guards
schema.js             # Joi schemas
controllers/          # Route handlers (listings, reviews, users)
routes/               # Express routers
models/               # Mongoose schemas (Listing, Review, User)
public/               # Static assets (CSS/JS)
views/                # EJS templates (layouts, partials, pages)
init/                 # Local seed script and sample data
utils/                # ExpressError + async wrapper
```

## Environment Variables

Create a `.env` file in the project root (the app loads it when `NODE_ENV !== 'production'`).

```
ATLASDB_URL=mongodb+srv://<user>:<pass>@<cluster>/wanderlust
SECRET=<random-long-string>              # session + MongoStore secret
CLOUD_NAME=<cloudinary-cloud-name>
CLOUD_API_KEY=<cloudinary-api-key>
CLOUD_API_SECRET=<cloudinary-api-secret>
MAP_TOKEN=<mapbox-public-access-token>
```

Notes:

- `ATLASDB_URL` is used both for Mongoose and the Mongo session store in production.
- Session cookies are HTTP-only and last 7 days; `SECRET` must be set.
- Cloudinary uploads are stored under the `wanderlust_DEV` folder and accept jpeg/png/jpg.
- Mapbox token must allow geocoding and map tile access.

## Installation & Local Development

1. Install Node 22.x.
2. Install dependencies:
   ```bash
   npm install
   ```
3. Configure `.env` as above. For local Mongo, you can also set `ATLASDB_URL=mongodb://127.0.0.1:27017/wanderlust`.
4. (Optional) Seed sample listings locally:
   ```bash
   node init/index.js
   ```
   - The seed script targets `mongodb://127.0.0.1:27017/wanderlust` by default.
   - It assigns the `owner` field to a hardcoded user id; update `init/index.js` to a valid user id from your DB or create that user first.
5. Start the server:
   ```bash
   node app.js
   ```
   - The app binds to port `8000` (the log message currently prints 8080).
6. Visit `http://localhost:8000/listings`.

## Routing Overview

- Listings: `GET /listings` (index), `GET /listings/new`, `POST /listings`, `GET /listings/:id`, `GET /listings/:id/edit`, `PUT /listings/:id`, `DELETE /listings/:id`.
- Reviews: `POST /listings/:id/reviews`, `DELETE /listings/:id/reviews/:reviewId`.
- Auth: `GET/POST /signup`, `GET/POST /login`, `GET /logout`.

## Data Models (Mongoose)

- Listing: `title`, `description`, `image { url, filename }`, `price`, `location`, `country`, `geometry { type: 'Point', coordinates: [lng, lat] }`, `owner` (User ref), `reviews` (Review refs). Cascade deletes reviews on listing removal.
- Review: `comment`, `rating`, `author` (User ref), `createdAt` (default now).
- User: `email`, plus `username`, `hash`, `salt` via `passport-local-mongoose`.

## Middleware & Validation

- `isLoggedIn`, `isOwner`, `isReviewAuthor` gate actions that mutate listings/reviews.
- `validateListing` and `validateReview` enforce Joi schemas for required fields, min/max rating, and non-negative price.
- Central error handler renders `views/error.ejs` for thrown `ExpressError` instances.

## Maps & Media

- Creating/updating a listing geocodes `listing.location` via Mapbox and stores GeoJSON coordinates on the Listing document.
- Client-side map rendering uses `public/js/map.js` with the injected `mapToken` and `listing.geometry` from the view.
- Images are uploaded through multer to Cloudinary and stored on the Listing document (`image.url`, `image.filename`).

## Production Notes

- Set `NODE_ENV=production` to skip automatic dotenv loading.
- Use a hosted MongoDB (Atlas) for both data and session store; ensure network rules allow the app to connect.
- Provide robust `SECRET` and rotate it if compromised; invalidate sessions as needed.
- Serve behind a TLS-terminating proxy and enable trust proxy/cookie `secure` flags if required.
- Consider adding a `start` script (e.g., `"start": "node app.js"`) and a process manager like PM2 or a systemd unit for deployment.

## Testing

No automated tests are defined (`npm test` is a placeholder). Add coverage for route handlers, auth flows, and validation before relying on this in production.
