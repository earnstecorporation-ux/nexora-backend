# NEXORA Razorpay Payment Backend

This is the secure backend service for **NEXORA Android app** that handles Razorpay order creation and HMAC-SHA256 signature verification.

## Security Architecture

1. **No Secret in the Client**: The Android APK only interacts with your public Key ID (`rzp_test_...`) and this backend. The `RAZORPAY_KEY_SECRET` remains strictly confidential on this server.
2. **Server-Side Order Creation**: Orders are created using official Razorpay APIs with a fixed amount of ₹49 INR (4900 paise).
3. **Cryptographic Signature Verification**: When Razorpay Checkout finishes on the Android device, the app submits `order_id`, `payment_id`, and `signature` to `/api/payment/verify-signature`. Premium is unlocked **only** when the server validates the SHA-256 HMAC signature.

---

## Quick Setup (Local Testing)

1. **Install dependencies**:
   ```bash
   cd backend
   npm install
   ```

2. **Configure environment**:
   Copy `.env.example` to `.env`:
   ```bash
   cp .env.example .env
   ```
   Open `.env` and fill in:
   - `RAZORPAY_KEY_ID`: Your Test Key ID from Razorpay Dashboard (starts with `rzp_test_`).
   - `RAZORPAY_KEY_SECRET`: Your Test Key Secret from Razorpay Dashboard.

3. **Start the server**:
   ```bash
   npm start
   ```
   The server runs at `http://localhost:3000`.

4. **Android Emulator Access**:
   When testing via the Android emulator, use `http://10.0.2.2:3000` as the `PAYMENT_BACKEND_URL` in the Android `.env` file or in the app's Settings screen.

---

## Production Deployment (Render, Railway, or Vercel)

### Option A: Render.com (Free Web Service)
1. Push your repository to GitHub.
2. Log in to [Render](https://render.com) and create a **New Web Service**.
3. Select your repository and set Root Directory to `backend`.
4. Build Command: `npm install`
5. Start Command: `node server.js`
6. Add Environment Variables:
   - `RAZORPAY_KEY_ID`
   - `RAZORPAY_KEY_SECRET`
7. Copy your deployed service URL (e.g., `https://nexora-backend.onrender.com`).
8. In the Android project's `.env`, set `PAYMENT_BACKEND_URL=https://nexora-backend.onrender.com`.

### Option B: Railway.app
1. Go to [Railway](https://railway.app) and create a **New Project from GitHub Repo**.
2. Set Root Directory to `/backend`.
3. Set the Environment Variables (`RAZORPAY_KEY_ID`, `RAZORPAY_KEY_SECRET`).
4. Generate a public domain and set it as `PAYMENT_BACKEND_URL` in the Android app.

---

## API Endpoints

### 1. Health Check
- **GET** `/api/health`
- Response:
  ```json
  {
    "status": "ok",
    "service": "NEXORA Payment Backend",
    "mode": "TEST",
    "keyConfigured": true
  }
  ```

### 2. Create Order
- **POST** `/api/payment/create-order`
- Body:
  ```json
  {
    "receipt": "nexora_receipt_12345"
  }
  ```
- Response:
  ```json
  {
    "success": true,
    "orderId": "order_EKwxwAgItmmXdp",
    "amount": 4900,
    "currency": "INR",
    "keyId": "rzp_test_..."
  }
  ```

### 3. Verify Signature
- **POST** `/api/payment/verify-signature`
- Body:
  ```json
  {
    "orderId": "order_EKwxwAgItmmXdp",
    "paymentId": "pay_29QQoUBi66xm2f",
    "signature": "9ef4dff0de670b3c79a4e02c..."
  }
  ```
- Response:
  ```json
  {
    "verified": true,
    "status": "PAID",
    "orderId": "order_EKwxwAgItmmXdp",
    "paymentId": "pay_29QQoUBi66xm2f",
    "amount": 4900,
    "currency": "INR",
    "entitlement": "LIFETIME_PREMIUM",
    "entitlementToken": "...",
    "verifiedAt": 1713333333333
  }
  ```
