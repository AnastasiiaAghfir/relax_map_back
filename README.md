# RelaxMap — Backend API

> **Note:** The backend is hosted on Render's free plan, which goes to sleep after a period of inactivity.
> The first request may take up to a minute while the server wakes up — after that, the site works at normal speed.

> A server platform for cataloguing places to relax: creating locations, categorising them by type and region, and a user rating and review system.

## 1. About the project

**RelaxMap Backend** is a REST API built with Node.js and Express that powers the client application for finding and rating places to relax.

Business goals the system addresses:

- centralised management of a location database with photos, descriptions, coordinates and ratings;
- structuring content by location type and by region of Ukraine;
- building trust through verified user reviews;
- managing user profiles and user-created content.

Database: MongoDB (Mongoose). Authorisation: session-based with token rotation.

## 2. Key features

**Users and access:**

- sign-up and login with email / password;
- session-based token refresh and secure logout;
- user profile and viewing other users' profiles;
- updating name and avatar.

**Locations:**

- public catalogue with pagination, search, filters and sorting;
- detailed location card;
- creating and editing locations by authorised users.

**Feedback:**

- 1 to 5 rating and a text review;
- viewing reviews for a location;
- aggregated rating and review counter.

**Reference data:**

- list of location types: `GET /api/categories/types`;
- list of regions: `GET /api/categories/regions`.

## 3. Tech stack

| Category       | Technology                               |
| -------------- | ---------------------------------------- |
| Runtime        | Node.js, ES Modules                      |
| HTTP Framework | Express 5                                |
| DB / ODM       | MongoDB, Mongoose 9                      |
| Auth / Crypto  | bcrypt, crypto.randomUUID                |
| Validation     | Celebrate, Joi                           |
| Files          | Multer, Cloudinary SDK v2                |
| Security       | Helmet, CORS, cookie-parser, http-errors |
| Configuration  | dotenv                                   |
| Logging        | pino-http, pino-pretty                   |
| Code quality   | ESLint, Prettier, EditorConfig           |
| Dev mode       | Nodemon                                  |

## 4. Project architecture

```text
src/
├── server.js              # Express setup, CORS, routes, error handling
├── db/connectToMongoDB.js # MongoDB connection
├── routes/                # endpoint definitions
├── controllers/           # request business logic
├── services/              # sessions, users, categories
├── models/                # Mongoose schemas
├── validations/           # Celebrate Joi schemas
├── middleware/            # authenticate, multer, logger, errorHandler
├── utils/                 # Cloudinary, Mongoose helpers
└── constants/             # token lifetimes, regex, sorting
```

Principles: thin routes, input validation, a service layer for reuse, centralised error handling.

## 5. Quick start

**Requirements:** Node.js 20+, MongoDB 6+, a Cloudinary account, npm 10+.

```bash
# 1. Clone
git clone https://github.com/stanislave-droid/relax_map.git
cd relax_map_back

# 2. Install dependencies
npm install

# 3. Configure the environment
cp .env.example .env
# fill in the values as described in section 6

# 4. Run in development mode
npm run dev

# 5. Run in production mode
npm start
```

Default server: `https://relax-map-back-z38k.onrender.com/`.

Check the reference endpoints:

```bash
curl https://relax-map-back-z38k.onrender.com/api/categories/types
curl "https://relax-map-back-z38k.onrender.com/api/locations?page=1&limit=10"
```

## 6. Environment variables

`.env` file (see `.env.example`):

| Variable                | Purpose                          | Required |
| ----------------------- | -------------------------------- | -------- |
| `PORT`                  | HTTP server port, e.g. `3000`    | Yes      |
| `NODE_ENV`              | `development` / `production`     | Yes      |
| `MONGO_URL`             | MongoDB connection string        | Yes      |
| `FRONTEND_DOMAIN`       | Allowed frontend CORS origin     | Yes      |
| `CLOUDINARY_CLOUD_NAME` | Cloudinary cloud name            | Yes      |
| `CLOUDINARY_API_KEY`    | Cloudinary API key               | Yes      |
| `CLOUDINARY_API_SECRET` | Cloudinary API secret            | Yes      |

> The `.env` file is not stored in the repository. In production, use your hosting platform's secrets manager.

## 7. Scripts

| Command       | Action                                      |
| ------------- | ------------------------------------------- |
| `npm start`   | Production start: `node src/server.js`      |
| `npm run dev` | Development start with auto-reload          |
| `npm test`    | Not configured yet (planned)                |

## 8. API overview

Base URL: `/api`. Data format: JSON. Authorisation: `sessionId`, `accessToken`, `refreshToken` cookies.

### 8.1 Authentication — `/api/auth`

| Method | Endpoint    | Access  | Request body                |
| ------ | ----------- | ------- | --------------------------- |
| POST   | `/register` | Public  | `{ name, email, password }` |
| POST   | `/login`    | Public  | `{ email, password }`       |
| POST   | `/refresh`  | Cookie  | —                           |
| POST   | `/logout`   | Private | —                           |

Sign-up example:

```bash
curl -X POST https://relax-map-back-z38k.onrender.com/api/auth/register \
  -H "Content-Type: application/json" \
  -c cookies.txt \
  -d '{"name":"Olena","email":"olena@example.com","password":"SecurePass123"}'
```

### 8.2 Locations — `/api/locations`

| Method | Endpoint       | Access  | Description                                                        |
| ------ | -------------- | ------- | ------------------------------------------------------------------ |
| GET    | `/`            | Public  | List: `page, limit, region, type, search, sortBy, sortDirection`   |
| GET    | `/:locationId` | Public  | Location details                                                   |
| POST   | `/`            | Private | Create + `multipart/form-data image`                               |
| PATCH  | `/:locationId` | Private | Update fields and photo                                            |

Search example:

```bash
curl "https://relax-map-back-z38k.onrender.com/api/locations?search=Карпати&region=Івано-Франківська&type=Гори&page=1&limit=10&sortBy=rate&sortDirection=desc"
```

### 8.3 Users — `/api/users`

| Method | Endpoint             | Access  | Description                              |
| ------ | -------------------- | ------- | ---------------------------------------- |
| GET    | `/current`           | Private | Current user                             |
| GET    | `/:userId`           | Public  | Public profile                           |
| GET    | `/:userId/locations` | Public  | Author's locations with pagination       |
| PATCH  | `/me/avatar`         | Private | Update `name` and `avatar` (multipart)   |

### 8.4 Categories — `/api/categories`

| Method | Endpoint   | Description    |
| ------ | ---------- | -------------- |
| GET    | `/types`   | Location types |
| GET    | `/regions` | Regions        |

### 8.5 Feedback — `/api/feedbacks`

| Method | Endpoint                      | Access  | Description                  |
| ------ | ----------------------------- | ------- | ---------------------------- |
| GET    | `/?page=1&limit=3`            | Public  | All reviews                  |
| GET    | `/:locationId?page=1&limit=3` | Public  | Reviews for a location       |
| POST   | `/:locationId`                | Private | `{ rate: 1-5, description }` |

Successful responses: `200 OK`; creation: `201 Created`. Errors are returned in the standard `http-errors` and Celebrate format.

## 9. Authentication and security

- Passwords hashed with bcrypt (10 rounds).
- Access token: 15 minutes. Refresh token: 1 day, with rotation (the old session is deleted).
- Cookies: `httpOnly, Secure, SameSite=None`.
- Header protection with Helmet. CORS is limited to `FRONTEND_DOMAIN` with `credentials: true`.
- Email validated by regex, password length 8–128 characters, ObjectId via a Mongoose helper.
- The password is excluded from API responses at the model level.

Production recommendations: HTTPS is required; add rate limiting on `/api/auth`; audit CORS domains.

## 10. Data models

- **User:** `name, email (unique), password (hash), avatarUrl, articlesAmount`, timestamps.
- **Location:** `image, name (3-96), description (20-6000), locationType, region, rate (0-5), ownerId, feedbacksId[], feedbacksCount, coordinates {lat, lon}`, timestamps.
- **Feedback:** `userName, ownerId, rate (1-5), description, locationId`, timestamps.
- **Category:** `type, slug, shortDescription`.
- **Region:** `region, slug, level, note`.
- **Session:** `userId, accessToken, refreshToken, accessTokenValidUntil, refreshTokenValidUntil`.

## 11. File uploads

Multer in memory storage mode → the buffer is sent to Cloudinary:

- avatars: `relax-map/avatars`, 500×500, `fill`, `gravity:auto`;
- location photos: `relax-map/locations`, 750×500;
- auto-optimisation: `fetch_format:auto, quality:auto`.

Allowed file types for location updates: `image/jpeg, image/jpg, image/png`.

## 12. Error handling and logging

- `notFoundHandler` — for unknown routes (404).
- `celebrate errors()` — validation errors (400).
- `errorHandler` — centralised error format.
- `pino-http` — structured request logging.

## 13. Deployment

1. Set the environment variables on your hosting platform (Render / Railway / VPS).
2. Allow MongoDB Atlas access by IP or VPC.
3. Set up HTTPS (required for `Secure` cookies).
4. Start command: `npm install && npm start`.
5. Health check: `GET /api/categories/types` should return `200`.

## 14. Project team

The project was built by a team of 12 members.

- **[stanislave-droid](https://github.com/stanislave-droid)** — repository owner, Team Lead. Users, locations, authentication, Cloudinary / Multer, sorting and fixes.
- **[Ellen-HI](https://github.com/Ellen-HI)** — session refresh, session model and service, session fixes, user update.
- **[LiliaMamutova](https://github.com/LiliaMamutova)** — user model, sign-up error messages.
- **[Viktor-8888](https://github.com/Viktor-8888)** — current user profile.
- **[Roman-Bieloshchuk](https://github.com/Roman-Bieloshchuk)** — logout.
- **[HyriaRoman](https://github.com/HyriaRoman)** — user's locations.
- **[OlgaDovgal](https://github.com/OlgaDovgal)** — creating locations, getting a location by ID, controller fixes.
- **[oleksandr0681](https://github.com/oleksandr0681)** — location update validation, Multer setup, locations router.
- **[KateB713](https://github.com/KateB713)** — category types and category response format.
- **[LaraMach](https://github.com/LaraMach)** — regions and review counter.
- **[AnastasiiaAghfir](https://github.com/AnastasiiaAghfir)** — getting reviews.
- **[Guijeen](https://github.com/Guijeen)** — getting the latest reviews, creating a new location review.
