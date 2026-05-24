# Stripe API Integration

A clean and secure backend integration with Stripe for handling payments, subscriptions, and webhook-based event processing in modern web applications.

This project demonstrates how to connect Stripe with a backend system to manage real-world payment flows.

## Features

- Secure Stripe payment integration.
- Support for one-time payments.
- Subscription handling.
- Webhook event processing.
- Clean backend API structure.
- Environment-based configuration with no hardcoded secrets.
- Ready for production extension.

## Tech Stack

- Backend: Node.js / Django / Flask.
- Stripe API.
- REST API architecture.
- Environment variables (`.env`).

## Project Structure

```text
project-root/
├── core/ or app/
│   ├── views/ or routes/
│   ├── services/
│   └── stripe/
├── .env
├── requirements.txt / package.json
└── README.md
```

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/Briankiboi/StripeAPIIntegration.git
cd StripeAPIIntegration
```

### 2. Create a virtual environment

If the backend is Python:

```bash
python -m venv venv
source venv/bin/activate
```

### 3. Install dependencies

For Python:

```bash
pip install -r requirements.txt
```

For Node.js:

```bash
npm install
```

## Stripe Setup

### 1. Create a Stripe account

- Go to [Stripe](https://stripe.com).
- Create an account.
- Navigate to **Developers → API Keys**.

### 2. Get API keys

You will get:
- Publishable Key.
- Secret Key.

### 3. Add environment variables

Create a `.env` file:

```env
STRIPE_SECRET_KEY=your_secret_key_here
STRIPE_PUBLIC_KEY=your_public_key_here
```

## Running the Project

Start the server:

For Python:

```bash
python manage.py runserver
```

For Node.js:

```bash
npm run dev
```

## API Flow

### 1. Create PaymentIntent

The client sends a request:

```http
POST /create-payment-intent
```

The server:
- Creates a Stripe PaymentIntent.
- Returns the client secret.

Stripe recommends creating the PaymentIntent on the server and passing the client secret to the client [web:25][web:27].

### 2. Confirm Payment

The frontend confirms payment using the Stripe SDK.

- Card details are handled securely by Stripe.
- No raw card data is stored on the backend.

### 3. Webhooks

Stripe sends events such as:
- `payment_intent.succeeded`
- `payment_intent.payment_failed`
- `customer.subscription.created`

These are handled at:

```http
/webhooks/stripe
```

Stripe recommends verifying webhook signatures using the `Stripe-Signature` header and your endpoint secret [web:33][web:30].

## Example Code

### Create PaymentIntent

```python
import os
import stripe

stripe.api_key = os.getenv("STRIPE_SECRET_KEY")

def create_payment(amount):
    intent = stripe.PaymentIntent.create(
        amount=amount,
        currency="usd",
    )
    return intent.client_secret
```

Stripe’s PaymentIntent API is the recommended flow for collecting payments securely [web:25][web:27].

### Webhook Handler

```python
from flask import request
import os
import stripe

endpoint_secret = os.getenv("STRIPE_WEBHOOK_SECRET")

def stripe_webhook():
    payload = request.data
    sig_header = request.headers.get("Stripe-Signature")

    try:
        event = stripe.Webhook.construct_event(
            payload=payload,
            sig_header=sig_header,
            secret=endpoint_secret,
        )
    except Exception as e:
        return {"error": str(e)}, 400

    if event["type"] == "payment_intent.succeeded":
        print("Payment successful")

    return {"status": "success"}, 200
```

Stripe recommends verifying webhook signatures and using the raw request body when constructing the event [web:33][web:30].

## Security Best Practices

- Never expose Stripe secret keys in frontend code.
- Always use `.env` for secrets.
- Validate webhook signatures.
- Use HTTPS in production.
- Do not store card data on your server.

## Testing Stripe

Use Stripe test cards such as:

- Card Number: `4242 4242 4242 4242`
- Expiry: Any future date.
- CVC: Any 3 digits.

Stripe provides a dedicated testing environment and documentation for test workflows [web:31].

## Common Issues

### 1. Invalid API key
- Check your `.env` file.
- Restart the server after changes.

### 2. Webhook not triggering
- Ensure the endpoint is publicly accessible.
- Use the Stripe CLI for testing.

### 3. Payment fails
- Use test mode keys.
- Check the currency format.

## Future Improvements

- Add subscription billing support.
- Add an admin dashboard for transactions.
- Add a payment history database.
- Add multi-currency support.
- Add email receipts after payment.

## License

MIT
