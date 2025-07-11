# Project Structure and Flow (Inferred from Tutorial)

This document outlines the assumed project structure and data flow for the Full Stack Food Delivery Website, based on common MERN stack practices and the tutorial's description. This will serve as a reference for future interactions.

## Frontend (React JS)

*   **`src/components/`**: Reusable UI components.
    *   `Navbar.jsx`
    *   `Footer.jsx`
    *   `FoodItemCard.jsx`
    *   `CartItem.jsx`
    *   ... (other common UI elements)
*   **`src/pages/`**: Top-level components representing different application views/routes.
    *   `Home.jsx`
    *   `Menu.jsx` (or `FoodDisplay.jsx`)
    *   `Cart.jsx`
    *   `PlaceOrder.jsx`
    *   `LoginRegister.jsx` (or separate `Login.jsx`, `Register.jsx`)
    *   `MyOrders.jsx`
    *   ... (other main views)
*   **`src/context/`** or **`src/redux/`**: State management for global application data.
    *   `StoreContext.jsx` (for React Context API)
    *   `store.js` (for Redux, with reducers, actions)
    *   Manages: user authentication status, cart items, food list, etc.
*   **`src/assets/`**: Static assets like images, icons, logos.
    *   `logo.png`
    *   `food_images/`
    *   `icons/`
*   **`src/api/`** or **`src/services/`**: Functions responsible for making API calls to the backend.
    *   `authService.js`
    *   `foodService.js`
    *   `orderService.js`
    *   `paymentService.js`
*   **`src/App.jsx`**: The main application component, typically handles routing using `react-router-dom`.
*   **`src/index.css`**, **`src/App.css`**: Global and component-specific styling.
*   **`src/main.jsx`**: Entry point for the React application.

## Backend (Node JS, Express, MongoDB, Stripe)

*   **`server.js`** or **`app.js`**: The main entry point for the Express application.
    *   Sets up the Express server.
    *   Connects to the MongoDB database.
    *   Mounts API routes.
    *   Handles global middleware.
*   **`config/`**: Configuration files for sensitive data and settings.
    *   `db.js` (MongoDB connection)
    *   `jwt.js` (JWT secret)
    *   `stripe.js` (Stripe API keys)
    *   `cors.js` (CORS settings)
*   **`models/`**: Defines Mongoose schemas for MongoDB collections.
    *   `User.js` (e.g., name, email, password, cart, orders)
    *   `Food.js` (e.g., name, description, price, category, image)
    *   `Order.js` (e.g., user, items, amount, address, status, paymentId)
*   **`routes/`**: Defines the API endpoints and links them to controller functions.
    *   `userRoutes.js` (`/api/user/register`, `/api/user/login`)
    *   `foodRoutes.js` (`/api/food/list`, `/api/food/add`)
    *   `cartRoutes.js` (`/api/cart/add`, `/api/cart/remove`, `/api/cart/get`)
    *   `orderRoutes.js` (`/api/order/place`, `/api/order/userorders`)
    *   `paymentRoutes.js` (`/api/payment/create-intent`)
*   **`controllers/`**: Contains the business logic for handling requests for each route.
    *   `userController.js` (registerUser, loginUser)
    *   `foodController.js` (addFood, listFood, removeFood)
    *   `cartController.js` (addToCart, removeFromCart, getCart)
    *   `orderController.js` (placeOrder, getUserOrders)
    *   `paymentController.js` (createPaymentIntent)
*   **`middleware/`**: Functions that execute before route handlers.
    *   `authMiddleware.js` (verifies JWT token)
    *   `errorMiddleware.js` (global error handling)
*   **`utils/`**: Helper functions.
    *   `jwtUtils.js` (generate/verify JWT)
    *   `stripeUtils.js` (Stripe API interactions)

## Overall Application Flow

1.  **User Authentication:**
    *   **Frontend:** User submits login/registration form data.
    *   **Frontend:** Sends `POST` request to `/api/user/register` or `/api/user/login`.
    *   **Backend:** Validates input, hashes password (for register), checks credentials (for login).
    *   **Backend:** If successful, generates a JSON Web Token (JWT) and sends it back with user data.
    *   **Frontend:** Stores the JWT (e.g., in `localStorage`) and includes it in the `Authorization` header for subsequent authenticated requests.

2.  **Food Listing & Display:**
    *   **Frontend:** On `Home` or `Menu` page load, sends `GET` request to `/api/food/list`.
    *   **Backend:** Fetches all food items from the `Food` collection in MongoDB.
    *   **Backend:** Sends the list of food items to the frontend.
    *   **Frontend:** Renders `FoodItemCard` components for each food item.

3.  **Cart Management:**
    *   **Frontend:** User clicks "Add to Cart" or "Remove from Cart" on a `FoodItemCard`.
    *   **Frontend:** Sends `POST` or `DELETE` request to `/api/cart/add` or `/api/cart/remove` with food item ID and quantity, including JWT for authentication.
    *   **Backend:** Updates the user's cart in the `User` model (or a separate `Cart` model) in MongoDB.
    *   **Frontend:** Updates the cart display (e.g., cart icon count, cart page).

4.  **Order Placement:**
    *   **Frontend:** User proceeds to "Place Order" page, fills in delivery details.
    *   **Frontend:** Sends `POST` request to `/api/order/place` with cart items, total amount, and delivery address, including JWT.
    *   **Backend:** Creates a new `Order` document in MongoDB, linking it to the user and cart items.
    *   **Backend:** Clears the user's cart.

5.  **Stripe Payment Integration:**
    *   **Frontend:** After placing an order, user proceeds to payment.
    *   **Frontend:** Sends `POST` request to `/api/payment/create-intent` with the order amount.
    *   **Backend:** Uses the Stripe Node.js library to create a `PaymentIntent` with Stripe.
    *   **Backend:** Sends the `client_secret` from the `PaymentIntent` back to the frontend.
    *   **Frontend:** Uses Stripe.js (client-side library) and the `client_secret` to confirm the payment on the client-side.
    *   **Stripe:** Processes the payment.
    *   **Backend (Webhook/Confirmation):** Receives confirmation from Stripe (or polls) and updates the `Order` status in MongoDB (e.g., "pending" to "paid").

6.  **My Orders:**
    *   **Frontend:** User navigates to "My Orders" page.
    *   **Frontend:** Sends `GET` request to `/api/order/userorders`, including JWT.
    *   **Backend:** Fetches all orders associated with the authenticated user from MongoDB.
    *   **Backend:** Sends the list of orders to the frontend.
    *   **Frontend:** Displays the user's order history.

7.  **Admin Dashboard (Optional, but common in such tutorials):**
    *   Separate set of routes and components for administrators.
    *   Allows adding/removing food items, viewing/managing all orders, managing users.
    *   Requires additional authentication/authorization middleware to restrict access to admin users only.
