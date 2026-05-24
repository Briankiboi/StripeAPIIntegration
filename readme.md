# Stripe API Integration for Django

A clean and secure backend integration with Stripe for handling one-time payments, subscriptions, and webhook-based event processing in Django applications.

This project demonstrates how to connect Stripe with a Django backend to manage real-world payment flows safely and efficiently.

---

## Table of Contents

1. Overview
2. Features
3. Tech Stack
4. Project Structure
5. Prerequisites
6. Stripe Setup
7. Django Setup
8. Payment Flow
9. Webhook Handling
10. Testing
11. Security Best Practices
12. Common Issues
13. Future Improvements
14. License

---

## Overview

This integration allows your Django application to accept payments through Stripe while keeping sensitive payment logic on the backend.

It supports secure payment processing, webhook event handling, and a structure that can be extended for subscriptions and advanced billing workflows [web:25][web:27][web:43].

---

## Features

- Secure Stripe payment integration.
- Support for one-time payments.
- Subscription handling.
- Webhook event processing.
- Clean backend API structure.
- Environment-based configuration.
- Production-ready foundation.

---

## Tech Stack

- Backend: Django.
- Payments: Stripe API.
- API style: REST.
- Configuration: Environment variables with `.env`.
- Optional tools: `requests`, `python-dotenv`.

---

## Project Structure

```text
project-root/
├── app/
│   ├── views.py
│   ├── urls.py
│   ├── utils.py
│   ├── webhooks.py
│   └── services/
├── project_name/
│   ├── settings.py
│   └── urls.py
├── .env
├── requirements.txt
└── README.md
```

---

## Prerequisites

Before starting, make sure you have:

- A Stripe account.
- A Django project already created.
- Python installed.
- Stripe Python SDK installed.
- `python-dotenv` installed for environment variables.
- A Stripe test account for development [web:31][web:40].

---

## Stripe Setup

### 1. Create a Stripe account

- Go to [Stripe](https://stripe.com).
- Create or sign in to your account.
- Open the **Developers** section.
- Copy your API keys from the dashboard [web:25].

### 2. Get API keys

You will need:
- Publishable key.
- Secret key.
- Webhook signing secret.

### 3. Add environment variables

Create a `.env` file:

```env
STRIPE_SECRET_KEY=your_secret_key_here
STRIPE_PUBLIC_KEY=your_public_key_here
STRIPE_WEBHOOK_SECRET=your_webhook_secret_here
```

Do not hardcode these values in your source code.

---

## Django Setup

### 1. Install dependencies

```bash
pip install stripe python-dotenv
```

### 2. Load environment variables

In `settings.py` or a utility module:

```python
import os
from dotenv import load_dotenv

load_dotenv()

STRIPE_SECRET_KEY = os.getenv("STRIPE_SECRET_KEY")
STRIPE_PUBLIC_KEY = os.getenv("STRIPE_PUBLIC_KEY")
STRIPE_WEBHOOK_SECRET = os.getenv("STRIPE_WEBHOOK_SECRET")
```

### 3. Configure Stripe in Django

Create a small helper file, for example `app/stripe_utils.py`:

```python
import os
import stripe
from dotenv import load_dotenv

load_dotenv()

stripe.api_key = os.getenv("STRIPE_SECRET_KEY")
```

---

## Payment Flow

The recommended Stripe pattern is to create a PaymentIntent on the server, then confirm it on the client using Stripe.js or another frontend SDK [web:25][web:27].

### 1. Create a PaymentIntent

```python
import stripe
import os

stripe.api_key = os.getenv("STRIPE_SECRET_KEY")

def create_payment_intent(amount, currency="usd"):
    intent = stripe.PaymentIntent.create(
        amount=amount,
        currency=currency,
    )
    return intent.client_secret
```

### 2. Backend view for payment intent

```python
from django.http import JsonResponse
from django.views.decorators.csrf import csrf_exempt
from .stripe_utils import create_payment_intent

@csrf_exempt
def create_payment_view(request):
    if request.method != "POST":
        return JsonResponse({"error": "Method not allowed"}, status=405)

    amount = 1000
    client_secret = create_payment_intent(amount)

    return JsonResponse({
        "client_secret": client_secret
    })
```

### 3. Client confirmation

The frontend uses the returned client secret to confirm the payment securely.

Stripe handles card details, so your backend never stores raw card data [web:25][web:27].

---

## Webhook Handling

Webhooks are important for receiving asynchronous Stripe events such as successful payments, failed payments, or subscription updates [web:43].

### 1. Create a webhook endpoint

In Django, create a webhook view:

```python
import os
import stripe
from django.http import JsonResponse
from django.views.decorators.csrf import csrf_exempt

stripe.api_key = os.getenv("STRIPE_SECRET_KEY")
endpoint_secret = os.getenv("STRIPE_WEBHOOK_SECRET")

@csrf_exempt
def stripe_webhook(request):
    payload = request.body
    sig_header = request.META.get("HTTP_STRIPE_SIGNATURE")

    try:
        event = stripe.Webhook.construct_event(
            payload=payload,
            sig_header=sig_header,
            secret=endpoint_secret,
        )
    except ValueError:
        return JsonResponse({"error": "Invalid payload"}, status=400)
    except stripe.error.SignatureVerificationError:
        return JsonResponse({"error": "Invalid signature"}, status=400)

    if event["type"] == "payment_intent.succeeded":
        payment_intent = event["data"]["object"]
        print("Payment successful:", payment_intent["id"])

    elif event["type"] == "payment_intent.payment_failed":
        payment_intent = event["data"]["object"]
        print("Payment failed:", payment_intent["id"])

    return JsonResponse({"status": "success"})
```

Stripe recommends using the raw request body and validating the `Stripe-Signature` header with the endpoint signing secret [web:33][web:42].

### 2. Add webhook URL to Stripe Dashboard

Use a URL like:

```text
https://yourdomain.com/webhooks/stripe/
```

You can test locally using the Stripe CLI or by exposing your local server with a tunnel [web:31][web:43].

---

## Testing

### Test card details

Use the Stripe test card:

- Card number: `4242 4242 4242 4242`
- Expiry: Any future date.
- CVC: Any 3 digits.

Stripe provides dedicated testing tools and test card scenarios for development [web:31][web:40].

### Test webhook events

Use the Stripe dashboard or Stripe CLI to send test events to your webhook endpoint [web:31][web:43].

### Verify payment flow

1. Create PaymentIntent.
2. Confirm it on the client.
3. Check that the webhook receives the event.
4. Confirm logs or database updates.

---

## Security Best Practices

- Never expose the Stripe secret key in frontend code.
- Always store sensitive values in `.env`.
- Validate webhook signatures.
- Use HTTPS in production.
- Do not store card details on your server.
- Log failures carefully without exposing secrets.

Webhook verification is especially important because it confirms that events really came from Stripe [web:33][web:42].

---

## Common Issues

### Invalid API key
- Check your `.env` values.
- Restart the server after updating environment variables.

### Webhook not triggering
- Ensure the endpoint is publicly reachable.
- Confirm the webhook URL is correct.
- Check the Stripe dashboard or CLI setup.

### Signature verification error
- Use `request.body`, not `request.POST`.
- Make sure you are using the correct webhook signing secret.
- Confirm the `Stripe-Signature` header is being passed correctly [web:33][web:36][web:42].

### Payment fails
- Use test mode keys.
- Confirm the currency and amount format.
- Try the test card again.

---

## Future Improvements

- Add subscription billing support.
- Add a transaction dashboard.
- Store payment history in a database.
- Support multiple currencies.
- Send email receipts after successful payment.
- Add asynchronous processing with Celery.

---

## License

This project is licensed under the MIT License.
