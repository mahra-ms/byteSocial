# byteSocial

A Node.js/Express + MongoDB backend for a "Tinder for developers"-style social/matching app. It handles user authentication, profile management, connection requests (swipe-style interest/ignore, accept/reject), a real-time chat system via Socket.IO, and premium memberships powered by Razorpay.

## Features

- **Authentication** — signup, login, and logout with JWT stored in an HTTP-only cookie, passwords hashed with bcrypt
- **Profile management** — view and edit profile, change password
- **Discovery feed** — paginated feed of users excluding yourself and anyone you already have a connection request with
- **Connection requests** — send a request as `interested` or `ignored`, and review incoming requests as `accepted` or `rejected`
- **Connections** — list your accepted connections and received requests
- **Real-time chat** — Socket.IO powered chat rooms per user pair, with message persistence, delivery, and "seen" status
- **Premium memberships** — Razorpay order creation, webhook-verified payment capture, and premium status flag on the user
- **CORS-protected API** — configured for a specific frontend origin

## Tech Stack

- [Node.js](https://nodejs.org/) / [Express 5](https://expressjs.com/)
- [MongoDB](https://www.mongodb.com/) via [Mongoose](https://mongoosejs.com/)
- [Socket.IO](https://socket.io/) for real-time messaging
- [JWT](https://www.npmjs.com/package/jsonwebtoken) for authentication
- [bcrypt](https://www.npmjs.com/package/bcrypt) for password hashing
- [Razorpay](https://razorpay.com/) for payments
- [validator](https://www.npmjs.com/package/validator) for input validation
- [dotenv](https://www.npmjs.com/package/dotenv), [cors](https://www.npmjs.com/package/cors), [cookie-parser](https://www.npmjs.com/package/cookie-parser)

## Project Structure

```
byteSocia/
├── src/
│   ├── app.js                     # Express app entry point, route mounting, server startup
│   ├── config/
│   │   └── database.js            # MongoDB connection
│   ├── middleware/
│   │   └── auth.js                # JWT-based auth middleware (userAuth)
│   ├── models/
│   │   ├── user.js                # User schema
│   │   ├── connectionRequest.js   # Connection request schema
│   │   ├── chat.js                # Chat schema
│   │   ├── message.js             # Message schema
│   │   └── payment.js             # Payment schema
│   ├── router/
│   │   ├── auth.js                # /signUp, /logIn, /logOut
│   │   ├── profile.js             # /profile/view, /profile/edit, /profile/changePassword
│   │   ├── request.js             # /request/send, /request/review
│   │   ├── user.js                # /feed, /user/connections, /user/requests/received
│   │   ├── chat.js                # /chat, /chat/:targetUserId
│   │   └── payment.js             # /payment/create, /payment/webhook, /premium/verify
│   └── utils/
│       ├── constants.js           # Membership pricing
│       ├── razorpay.js            # Razorpay client instance
│       ├── socket.js              # Socket.IO event handlers
│       └── validation.js          # Signup/profile-edit validation
├── package.json
└── package-lock.json
```

## Getting Started

### Prerequisites

- [Node.js](https://nodejs.org/)
- A MongoDB database (local or hosted, e.g. MongoDB Atlas)
- A [Razorpay](https://razorpay.com/) account (test keys are fine for development)

### Installation

```bash
git clone https://github.com/mahra-ms/devTinder.git
cd devTinder
npm install
```

### Environment Variables

Create a `.env` file in the project root with the following:

```env
DB_CONNECTION_SECRET=your_mongodb_connection_string
JWT_SECRET=your_jwt_secret
RAZORPAY_KEY_ID=your_razorpay_key_id
RAZORPAY_KEY_SECRET=your_razorpay_key_secret
RAZORPAY_WEBHOOK_SECRET=your_razorpay_webhook_secret
```

### Running the server

```bash
# Development (auto-restart with nodemon)
npm run dev

# Production
npm start
```

The server starts on **port 3000** and connects to MongoDB before accepting requests. Socket.IO is initialized on the same HTTP server.

By default, CORS is configured to allow requests from `http://localhost:5173` and a deployed frontend origin — update this in `src/app.js` and `src/utils/socket.js` to match your own frontend URL.

## API Overview

### Auth
| Method | Endpoint | Description |
|---|---|---|
| POST | `/signUp` | Create a new user account |
| POST | `/logIn` | Log in and receive an auth cookie |
| POST | `/logOut` | Clear the auth cookie |

### Profile
| Method | Endpoint | Description |
|---|---|---|
| GET | `/profile/view` | Get the logged-in user's profile |
| PATCH | `/profile/edit` | Update allowed profile fields |
| PATCH | `/profile/changePassword` | Change password |

### Discovery & Connections
| Method | Endpoint | Description |
|---|---|---|
| GET | `/feed` | Paginated feed of discoverable users |
| POST | `/request/send/:status/:toUserId` | Send a request (`interested` or `ignored`) |
| POST | `/request/review/:status/:requestId` | Review a received request (`accepted` or `rejected`) |
| GET | `/user/requests/received` | List pending incoming requests |
| GET | `/user/connections` | List accepted connections |

### Chat
| Method | Endpoint | Description |
|---|---|---|
| GET | `/chat/:targetUserId` | Get or create a chat and fetch its message history |

Real-time messaging uses Socket.IO events: `joinChat`, `sendMessage`, `messageReceived`, `messageSeen`, `messagesSeen`.

### Payments
| Method | Endpoint | Description |
|---|---|---|
| POST | `/payment/create` | Create a Razorpay order for a membership plan (`plus` or `pro`) |
| POST | `/payment/webhook` | Razorpay webhook to verify and capture payment |
| GET | `/premium/verify` | Check the logged-in user's premium status |

Most endpoints require authentication via the `token` cookie set at login, enforced by the `userAuth` middleware.

## License

ISC
