# SnipURL – MERN Stack URL Shortener

**Transform long URLs into short, shareable links.**

SnipURL is a full-stack URL shortening application built with MongoDB, Express.js, React.js, and Node.js. It lets users generate compact links, reuse existing short codes for duplicate URLs, track link clicks, and access recently created links through a responsive interface.

## Architecture at a Glance

SnipURL connects a React frontend to an Express API backed by MongoDB. The browser also uses `localStorage` to retain recent links.

### URL Shortening Workflow

When a user submits a long URL, the backend validates it, checks for duplicates, and either returns an existing short code or creates a new record.

### Redirection and Click Tracking Workflow

When a visitor opens a short link, the backend looks up its short code, increments the click counter, and responds with an HTTP 302 redirect. Unknown codes return a 404 response.

## Features

- **Instant URL shortening:** Convert valid `http://` and `https://` URLs into short links.
- **Unique short codes:** Generate seven-character alphanumeric identifiers using Nano ID.
- **Duplicate detection:** Return the existing short code when a URL has already been shortened.
- **Click tracking:** Increment the click counter whenever a short link is accessed.
- **Automatic redirection:** Redirect visitors to the original destination using HTTP 302.
- **Recent links:** Keep the five most recently generated links in browser `localStorage`.
- **Copy to clipboard:** Copy a generated short URL with visual feedback.
- **Loading states:** Show a loading indicator while API requests are processing.
- **Error handling:** Return a 404 response when a short code doesn't exist.
- **Responsive design:** Provide a mobile-first experience across desktop, tablet, and mobile screens.

## Technology Stack

| Layer | Technologies |
|---|---|
| Frontend | React 18, Vite, React Router v6 |
| Styling | Tailwind CSS v3, Google Fonts (Inter) |
| HTTP client | Axios |
| Backend | Node.js, Express.js |
| Database | MongoDB, Mongoose |
| Short-code generation | Nano ID |
| Browser storage | localStorage |
| Development | npm, Nodemon |

## How It Works

1. The user enters a valid long URL in the React application.
2. The frontend sends the URL to `POST /api/url/shorten`.
3. The backend validates the URL and checks whether it already exists.
4. If it exists, the backend returns its existing short code.
5. Otherwise, the backend generates a seven-character Nano ID and saves the URL in MongoDB.
6. The frontend displays the generated short URL and allows the user to copy it.
7. When someone opens the short URL, the backend looks up the code, increments its click counter, and redirects the visitor to the original URL.

```text
User enters URL
      |
      v
React frontend
      |
      v
POST /api/url/shorten
      |
      v
Validate URL
      |
      v
Check MongoDB for duplicate
      |
      +------ Existing URL ------> Return existing short code
      |
      +------ New URL -----------> Generate Nano ID
                                      |
                                      v
                                 Save in MongoDB
                                      |
                                      v
                              Return shortened URL
                                      |
                                      v
                              Display result in UI

Visitor opens short URL
      |
      v
GET /api/url/:code
      |
      v
Find original URL in MongoDB
      |
      v
Increment click counter
      |
      v
HTTP 302 redirect
```

## Project Structure

```text
SnipURL/
├── backend/
│   ├── config/
│   │   └── db.js
│   ├── controllers/
│   │   └── urlController.js
│   ├── models/
│   │   └── Url.js
│   ├── routes/
│   │   └── urlRoutes.js
│   ├── .env
│   ├── server.js
│   └── package.json
│
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── UrlForm.jsx
│   │   │   ├── ShortUrlResult.jsx
│   │   │   └── RecentLinks.jsx
│   │   ├── pages/
│   │   │   ├── Home.jsx
│   │   │   └── RedirectHandler.jsx
│   │   ├── services/
│   │   │   └── api.js
│   │   ├── App.jsx
│   │   ├── main.jsx
│   │   └── index.css
│   ├── index.html
│   ├── tailwind.config.js
│   ├── postcss.config.js
│   ├── vite.config.js
│   └── package.json
│
└── README.md
```

## Getting Started

### Prerequisites

- Node.js v18 or later
- npm
- MongoDB running locally, or a MongoDB Atlas connection string
- Git

### 1. Clone the repository

```bash
git clone https://github.com/anuj-yd/SnipURL.git
cd SnipURL
```

### 2. Set up the backend

```bash
cd backend
npm install
```

Create a `.env` file inside the `backend` directory:

```env
PORT=5000
MONGO_URI=mongodb://localhost:27017/urlshortener
BASE_URL=http://localhost:5000
```

Start the backend:

```bash
# Development mode (requires the dev script to be configured)
npm run dev

# Production mode
npm start
```

The backend should be available at `http://localhost:5000`.

### 3. Set up the frontend

Open a second terminal:

```bash
cd frontend
npm install
npm run dev
```

Vite will display the local frontend URL, typically `http://localhost:5173`.

> **Note:** If Tailwind CSS isn't configured in the project yet, install and configure the version compatible with the existing Tailwind v3 setup. The configuration files listed in the project structure may already be present.

## API Reference

### Shorten a URL

`POST /api/url/shorten`

Creates a short link for a valid URL. If duplicate detection is enabled for the submitted URL, the existing short code is returned instead of creating another record.

**Request body**

```json
{
  "longUrl": "https://example.com/very/long/path"
}
```

**Example success response**

```json
{
  "shortUrl": "http://localhost:5000/api/url/aB3dE7f",
  "shortCode": "aB3dE7f",
  "longUrl": "https://example.com/very/long/path",
  "clicks": 0,
  "createdAt": "2026-11-03T10:00:00.000Z"
}
```

The example response illustrates the expected response shape. The actual HTTP status code and field names should match the backend implementation.

### Redirect using a short code

`GET /api/url/:code`

Looks up the supplied short code, increments its click count, and redirects the visitor to the original URL.

**Example request**

```http
GET /api/url/aB3dE7f
```

**Expected outcomes**

| Status | Meaning |
|---|---|
| 302 | Redirect to the original URL |
| 404 | No URL exists for the supplied short code |
| 500 | Unexpected server error |

**Example 404 response**

```json
{
  "error": "Short URL not found."
}
```

## Environment Variables

| Variable | Description | Local example |
|---|---|---|
| `PORT` | Port used by the Express server | `5000` |
| `MONGO_URI` | MongoDB connection string | `mongodb://localhost:27017/urlshortener` |
| `BASE_URL` | Base URL used to construct short links | `http://localhost:5000` |

**Security:** Don't commit real credentials or production secrets. Add `.env` to `.gitignore` and provide a `.env.example` containing placeholder values for contributors.

## Deployment

A possible deployment setup is:

| Component | Platform options |
|---|---|
| Frontend | Vercel, Netlify |
| Backend | Render, Railway |
| Database | MongoDB Atlas |

Before deploying:

1. Create a production MongoDB database and configure its connection string.
2. Set the backend environment variables in the hosting provider.
3. Set `BASE_URL` to the deployed backend's public URL.
4. Configure the frontend API base URL to point to the deployed backend.
5. Configure CORS to allow requests from the deployed frontend domain.
6. Deploy the frontend and verify that URL creation and redirection work end to end.

Example CORS configuration (adapt it to the existing `server.js` implementation):

```js
app.use(cors({
  origin: "https://your-frontend-domain.com"
}));
```

## Future Enhancements

The following improvements could extend the project's functionality and scalability:

- **Base62 short-code generation:** Explore a custom Base62 encoding approach for compact, deterministic identifiers. This is a proposed enhancement; the current implementation uses Nano ID.
- **User authentication:** Add JWT-based authentication and user-specific URL management.
- **Analytics dashboard:** Display click statistics, traffic trends, and link performance.
- **QR code generation:** Create downloadable QR codes for shortened links.
- **Custom aliases:** Let users choose memorable short codes.
- **Link expiration:** Support configurable expiration dates for short URLs.
- **Rate limiting:** Protect API endpoints from abuse and excessive requests.
- **Redis caching:** Cache frequently accessed short-code lookups to improve redirect performance.
- **URL management:** Allow authenticated users to view, edit, and delete their links.
- **Docker support:** Containerize the frontend and backend to make local setup and deployment more consistent.

## Learning Outcomes

Working on SnipURL provides practical experience with:

- Building and organizing a full-stack MERN application.
- Designing and integrating REST APIs.
- Modeling and querying data with MongoDB and Mongoose.
- Understanding HTTP redirects and status codes.
- Handling asynchronous frontend-to-backend communication.
- Managing browser-side data with localStorage.
- Building reusable React components and client-side routes.
- Structuring backend code into routes, controllers, models, and configuration.

## Author

**Anuj Kumar Yadav**  
B.Tech, Computer Science and Engineering  
Lovely Professional University

- GitHub: https://github.com/anuj-yd
- Project repository: https://github.com/anuj-yd/SnipURL

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.
