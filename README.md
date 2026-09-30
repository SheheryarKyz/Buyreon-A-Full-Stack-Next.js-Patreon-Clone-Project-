BUYREON — FULL PROJECT DOCUMENTATION
==========================================

Project: Buyreon
Project Type: Full-Stack Creator Crowdfunding Platform
Primary Purpose: Allow creators to build public profiles and receive financial support from fans.
Current Payment Environment: Safepay Sandbox
Database: MongoDB
Framework: Next.js
Language: JavaScript

1. PROJECT OVERVIEW
-------------------

Buyreon is a full-stack crowdfunding platform for creators, inspired by platforms such as Patreon.

Creators can authenticate with GitHub, manage a public creator profile, configure Safepay payment information, and receive financial support from visitors.

Visitors can visit creator pages, enter their name and message, choose a payment amount, and complete payment through Safepay.

The project was built as a hands-on full-stack application covering frontend UI, authentication, sessions, database operations, server-side logic, forms, dynamic routing, and third-party payment integration.


2. TECHNOLOGY STACK
-------------------

Frontend:
- Next.js
- React
- JavaScript
- Tailwind CSS
- React Hook Form

Backend:
- Next.js Server Actions
- Next.js API Routes
- Node.js
- Mongoose

Database:
- MongoDB

Authentication:
- NextAuth
- GitHub OAuth
- SessionProvider

Payments:
- Safepay
- @sfpy/node-core

Development:
- npm
- Git
- GitHub
- VS Code


3. PROJECT STRUCTURE
--------------------

BUYREON/
|
├── actions/
│   └── useractions.js
|
├── app/
│   ├── [username]/
│   │   └── page.js
│   ├── api/
│   │   └── auth/
│   │       └── [...nextauth]/
│   │           └── route.js
│   ├── dashboard/
│   │   └── page.js
│   ├── login/
│   │   └── page.js
│   ├── globals.css
│   ├── layout.js
│   └── page.js
|
├── components/
│   ├── Footer.js
│   ├── Navbar.js
│   ├── PaymentPage.js
│   └── SessionWrapper.js
|
├── db/
│   └── connectDb.js
|
├── models/
│   ├── Payment.js
│   └── User.js
|
├── public/
├── .env.local
├── next.config.mjs
├── package.json
└── README.md


4. APPLICATION ROUTES
---------------------

/
Home page.

 /login
Authentication page.

 /dashboard
Authenticated creator dashboard.

 /[username]
Dynamic public creator page. Each creator gets a URL based on their username.

Example:
/shaheryark56


5. ROOT LAYOUT
--------------

The root layout contains the shared application structure:

- Geist fonts
- Global CSS
- SessionWrapper
- Navbar
- Main page content
- Footer

The Navbar was made fixed so it remains visible while scrolling. The page content receives top padding to compensate for the fixed navigation bar.


6. NAVBAR
---------

The Navbar is a Client Component because it uses authentication state and client-side navigation.

It contains:
- Buyreon logo
- Home
- About
- Projects
- Login/Sign Up when logged out
- Dashboard/Your Page/Sign Out when logged in

The About link points to the Discover Buyreon section on the homepage:

/#discover-buyreon

A separate About page is not required.


7. AUTHENTICATION
-----------------

Buyreon uses NextAuth with GitHub OAuth.

Authentication route:

app/api/auth/[...nextauth]/route.js

The flow is:

User
  ↓
Login
  ↓
GitHub OAuth
  ↓
NextAuth
  ↓
signIn callback
  ↓
MongoDB user lookup
  ↓
Create user if necessary
  ↓
Session
  ↓
Authenticated application


8. USER CREATION
----------------

When a user signs in through GitHub, the signIn callback searches MongoDB using the user's email.

If no matching user exists, a new User document is created.

The initial username is derived from the user's email address.

The creator can then update their profile through the dashboard.


9. SESSION HANDLING
-------------------

SessionWrapper provides NextAuth's SessionProvider.

The Navbar reads the authentication session to determine whether the user is logged in.

The session callback retrieves the MongoDB user and assigns the database username to:

session.user.name

This username is then used to navigate to the creator's public page.


10. USER MODEL
--------------

File:

models/User.js

Fields:

email
name
username
profilePicture
coverPicture
safepayPublicKey
safepaySecretKey
createdAt
updateAt

The username is used as the creator's public URL identifier.


11. DATABASE CONNECTION
-----------------------

File:

db/connectDb.js

Local MongoDB connection:

mongodb://localhost:27017/buyreon

Mongoose is used to connect the Next.js application to MongoDB.


12. DASHBOARD
------------

The dashboard is for authenticated creators.

It uses:
- React Hook Form
- NextAuth session
- Server Actions
- MongoDB

Creators can update:
- Name
- Email
- Username
- Profile picture
- Cover picture
- Safepay public key
- Safepay secret key

The update is performed through updateProfile() in actions/useractions.js.

The update function also checks for duplicate usernames.


13. SERVER ACTIONS
------------------

File:

actions/useractions.js

Main actions:

initiate()
Creates a Safepay payment session and saves payment information.

fetchuser()
Retrieves a creator from MongoDB.

updateProfile()
Updates creator information in MongoDB.

These functions allow client UI components to interact with server-side functionality without exposing database credentials or payment secrets to the browser.


14. CREATOR PUBLIC PAGE
----------------------

File:

app/[username]/page.js

The dynamic page:

1. Receives username from the URL.
2. Connects to MongoDB.
3. Finds the creator.
4. Handles a missing creator.
5. Retrieves completed payments.
6. Converts payment data into supporter information.
7. Passes safe creator data to PaymentPage.

Safepay secret credentials are not passed to the client component.


15. PAYMENT PAGE
---------------

File:

components/PaymentPage.js

The payment form uses React Hook Form.

Supporters can enter:
- Name
- Message
- Amount

There is a Pay button and quick payment buttons for:

PKR 1,000
PKR 2,000
PKR 3,000

The form calls initiate() with the selected amount, creator username, and form information.


16. PAYMENT FLOW
----------------

The payment flow is:

Supporter
  ↓
Creator Page
  ↓
Payment Form
  ↓
Server Action
  ↓
Find Creator in MongoDB
  ↓
Retrieve Creator's Safepay Credentials
  ↓
Create Safepay Payment Session
  ↓
Create Tracker
  ↓
Create Authentication Token
  ↓
Generate Safepay Checkout URL
  ↓
Redirect Supporter to Safepay
  ↓
Payment Completed
  ↓
Return to Buyreon
  ↓
Payment Status Processed
  ↓
MongoDB
  ↓
Completed Payment Appears on Creator Page


17. SAFEPAY INTEGRATION
-----------------------

Buyreon currently uses the Safepay sandbox environment.

The amount entered by the supporter is in PKR.

The amount is converted to paisa before being sent to Safepay:

100 PKR = 10,000 paisa

Creator-specific Safepay credentials are retrieved from MongoDB on the server.

The installed Safepay SDK is @sfpy/node-core.


18. PAYMENT MODEL
-----------------

File:

models/Payment.js

Fields:

name
to_user
oid
message
amount
createdAt
updatedAt
done

name:
Supporter's name.

to_user:
Username of the creator receiving the payment.

oid:
Safepay tracker/payment identifier.

message:
Optional supporter message.

amount:
Payment amount in PKR.

done:
Whether the payment has been completed.


19. SUPPORTER DISPLAY
---------------------

Only completed payments are displayed publicly.

Payments are queried using:

to_user: username
done: true

They are sorted by:

createdAt: -1

Newest completed payments therefore appear first.

Each supporter entry displays:
- Name
- Amount
- Optional message


20. HOME PAGE
-------------

The homepage contains:

Hero:
Introduces Buyreon and provides navigation.

Built for creators:
Explains:
- Fans Want to Help
- Turn Support Into Funding
- Build Your Community

Discover Buyreon:
Provides additional platform information and serves as the About section.

The Navbar About link points to this section.


21. UI DESIGN
-------------

Buyreon uses a dark visual style with:

- Black backgrounds
- Purple/blue gradients
- White primary text
- Gray secondary text
- Rounded cards
- Subtle borders
- Hover effects
- Responsive layouts
- Gradient buttons

Tailwind CSS is used throughout the interface.


22. ENVIRONMENT VARIABLES
-------------------------

Example:

GITHUB_ID=your_github_client_id
GITHUB_SECRET=your_github_client_secret

NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=your_nextauth_secret

NEXT_PUBLIC_URL=http://localhost:3000

Actual credentials must never be committed to GitHub.

The .env.local file should remain private.


23. LOCAL SETUP
--------------

Install dependencies:

npm install

Make sure MongoDB is running locally.

Create .env.local and configure the required environment variables.

Then start Next.js:

npm run dev

Open:

http://localhost:3000


24. GITHUB OAUTH
---------------

A GitHub OAuth application is required for login.

The GitHub client ID and secret are supplied through:

GITHUB_ID
GITHUB_SECRET

The callback URL must be configured correctly for the local Buyreon installation.


25. IMPORTANT DEVELOPMENT ISSUES SOLVED
---------------------------------------

Mongoose:
An older connection option, useNewUrlParser, caused an unsupported-option error with the installed Mongoose version. Removing it resolved the problem.

Safepay SDK:
The installed SDK version required:

safepay.client.passport.create()

instead of:

safepay.auth.passport.create()

Safepay checkout:
The implementation uses:

safepay.checkout.createCheckoutUrl()

MongoDB schema:
Safepay credential fields had to be added to the User schema before they could be stored correctly.

Mongoose model caching:
After changing the User schema, restarting the Next.js development server was required so the updated model definition would be loaded.

Client/server serialization:
MongoDB user documents were converted to plain objects before being passed to Client Components.


26. SECURITY
------------

Sensitive values should remain server-side.

Important secrets include:
- GitHub client secret
- NextAuth secret
- Safepay secret credentials

These values should never be exposed in Client Components.

Creator Safepay credentials are currently stored in MongoDB. A production financial application would require additional security measures such as secure credential storage/encryption and stronger payment verification.

Buyreon is currently a learning and portfolio project and should not be treated as a production-ready financial platform without further security and payment-verification work.


27. CURRENT PROJECT STATUS
--------------------------

Buyreon has reached the stage of a functional first serious full-stack project.

The project demonstrates:

- Frontend development
- Backend development
- MongoDB integration
- Mongoose
- Authentication
- OAuth
- Session management
- CRUD operations
- Dynamic routing
- Server Actions
- API integration
- Payment integration
- Form handling
- Responsive UI
- Client/server communication

The main application flow is:

Login
  ↓
Dashboard
  ↓
Update Creator Profile
  ↓
Public Creator Page
  ↓
Supporter Payment
  ↓
Safepay
  ↓
Payment Completion
  ↓
MongoDB
  ↓
Supporter Display


28. FUTURE IMPROVEMENTS
-----------------------

Potential future additions include:

- Creator posts
- Membership tiers
- Recurring subscriptions
- Creator analytics
- Payment history dashboard
- Email notifications
- Comments
- Likes
- Creator discovery
- Improved payment verification
- Production webhook handling
- Cloud MongoDB
- Production deployment
- Improved error handling
- Additional authentication providers
- Admin functionality


29. LEARNING OUTCOMES
--------------------

Buyreon provided hands-on experience with a complete full-stack workflow.

Frontend:
- React components
- JSX
- Tailwind CSS
- Client Components
- React Hook Form
- Responsive design

Next.js:
- App Router
- Layouts
- Dynamic routes
- Server Components
- Client Components
- Server Actions
- API routes

Backend:
- Server-side JavaScript
- Database operations
- Authentication callbacks
- Third-party API integration

Database:
- MongoDB
- Mongoose
- Schemas
- Models
- Queries
- Updates
- Document creation
- Filtering
- Sorting

Authentication:
- OAuth
- GitHub authentication
- Sessions
- Protected pages

Payments:
- Safepay
- Payment sessions
- Trackers
- Checkout redirects
- Payment completion

Development:
- Debugging
- Environment variables
- SDK compatibility
- Client/server boundaries
- Database persistence
- Form state management


30. PORTFOLIO DESCRIPTION
-------------------------

Recommended project description:

"Buyreon is a full-stack creator crowdfunding platform built with Next.js, React, MongoDB, Mongoose, NextAuth, Tailwind CSS, and Safepay. It allows creators to manage public profiles and receive financial support from fans through integrated payments."


31. FINAL SUMMARY
-----------------

Buyreon is a complete first full-stack project combining:

React / Next.js
        ↓
Server Actions / API Routes
        ↓
MongoDB / Mongoose
        ↓
Safepay

It demonstrates the ability to build a complete application across frontend, backend, database, authentication, and third-party service integration.

The project is suitable as a portfolio project and provides a foundation for developing more advanced full-stack applications.
