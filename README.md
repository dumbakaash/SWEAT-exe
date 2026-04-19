Here's a complete description for SWEAT.EXE:

---

# SWEAT.EXE — AI Fitness OS

## What It Is

SWEAT.EXE is a full-stack AI-powered fitness web application built as a single HTML file. It combines a marketing landing page with a complete authenticated app — no page reloads, no separate files, no backend server required to run. Users go from the landing page to a fully functional fitness dashboard in seconds, with every feature powered by the Anthropic Claude API.

---

## How It Works — Full Flow

### 1. Landing Page
The user arrives at a Gen Z-styled dark-mode landing page with animated hero section, live profile card mockups, a stats strip (50K users, 4.9★, $5/mo, 94% hit goals), feature grid, how-it-works steps, pricing cards, testimonials, and a CTA section. Everything is scroll-animated with intersection observer reveals.

### 2. Sign Up / Login
Clicking any CTA opens an auth overlay with Login and Sign Up tabs. User data (name, email, hashed password, plan) is stored in localStorage under namespaced keys. Sessions persist across browser refreshes via a `session` key. No external database is needed — everything runs client-side.

### 3. Payment
The payment modal offers two tabs — **Stripe** (card) and **PayPal**. The card form validates card number length, expiry format, and CVV before processing. On success, the user's plan is upgraded to PRO and they are auto-logged into the app. To go fully live, you connect a backend that creates a Stripe `PaymentIntent` or a PayPal subscription order and returns the client secret.

### 4. App Launch
After auth or payment, the landing page fades out and the full dashboard app slides in — sidebar, topbar, and content area — all without any page navigation.

---

## App Modules

### 🏠 Dashboard
The home screen shows four live stat cards (streak, workouts logged, BMI, TDEE), a weekly activity bar chart that fills based on logged sessions, a quick-status panel with macro progress bars pulled from the user's saved profile, and a real-time activity feed showing the last 5 logged actions across all modules.

### 🧬 BMI + TDEE Calculator
The user enters age, gender, weight, height, activity level, and primary goal. The app calculates BMI using the standard formula, BMR using the Mifflin-St Jeor equation (separate male/female formulas), TDEE by multiplying BMR by an activity multiplier, target daily calories adjusted for goal (−500 for fat loss, +300 for muscle gain), and daily protein target based on body weight and goal. All results are saved to localStorage under the user's email key and shared across every other module. After calculation, the app calls the Claude API and streams a personalized health analysis directly into the UI.

### 💪 Workout Plan Generator
The user selects days per week (3–6), fitness level, available equipment, training split (Push/Pull/Legs, Upper/Lower, Full Body, Bro Split), primary focus, and any special notes like injuries or preferences. The app builds a detailed prompt using the user's saved BMI profile and sends it to Claude, which streams back a complete day-by-day workout plan with exercise names, sets, reps, rest times, and coaching tips per exercise. If BMI is above 30, the prompt automatically instructs Claude to use low-impact exercises only.

### 🥗 Meal Plan Generator
The user sets dietary restriction (vegan, gluten-free, keto, halal, etc.), meals per day, budget, cuisine preference, foods to avoid, and allergies. The app pulls the saved calorie and protein targets from the BMI profile and sends everything to Claude, which generates a complete one-day meal plan with exact ingredient amounts in grams, quick prep instructions, and a macro breakdown (protein, carbs, fat, calories) per meal plus a daily total summary.

### 🤖 AI Coach Chat
A full real-time chat interface with the Claude API. The coach is initialized with the user's profile as system context (BMI, goal, TDEE, target calories) so every answer is personalized. Messages stream token by token into the chat bubble as they are generated. The conversation history is maintained in memory for the full session so the coach remembers earlier messages. The sidebar has 8 quick-prompt buttons covering the most common fitness questions. There is a typing indicator, auto-resizing textarea, Enter to send, and a clear chat button.

### 📈 Progress Tracker
Two logging forms — one for body weight (with optional note) and one for individual exercises (sets, reps, weight, duration). Logged weight entries render as a live bar chart showing up to 16 entries, with the most recent bar highlighted in lime green. Exercise history shows in a sortable table. A separate AI Insights button sends the user's full weight history, change over time, and recent exercises to Claude, which streams back a progress assessment, what is working, what to adjust, and a two-week focus plan.

### ⚙️ Settings
Shows account info (name, email, plan, join date), API key management (add, test, change, remove), fitness profile summary with an update link, data summary showing counts of all logged entries, and billing management with cancel plan and upgrade buttons that update the user's plan state in real time.

---

## API Key System

Every AI feature requires an Anthropic API key entered by the user. The key is stored only in the user's browser localStorage — it is never sent to any third-party server. When the user saves a key, the app makes a live test call to the Anthropic API using the lightweight Claude Haiku model to confirm the key is valid before saving it. A green animated dot in the sidebar and topbar confirms the key is active. If no key is set, every AI feature shows a clear inline message directing the user to the API setup screen. The key can be removed at any time from Settings.

---

## Payment Integration

### Stripe
The Stripe JS SDK (`js.stripe.com/v3`) is loaded from the official CDN. The card form collects card number, expiry, CVV, and cardholder name with live formatting and client-side validation. In production, your backend creates a `PaymentIntent` with the amount and currency, returns the `client_secret` to the frontend, and you call `stripe.confirmCardPayment(clientSecret, {payment_method: {card: cardElement}})`. On confirmation, the webhook on your server upgrades the user's plan.

### PayPal
The PayPal tab in the payment modal handles the subscription flow. In production, you load the PayPal JS SDK with your client ID and plan ID, call `paypal.Buttons({createSubscription: (data, actions) => actions.subscription.create({plan_id: 'P-XXXX'}), onApprove: (data) => upgradeUser(data.subscriptionID)}).render('#paypal-container')`. The current implementation simulates the full approval flow with a 2.2-second processing delay and success screen.

---

## Tech Stack

**Frontend** — Vanilla HTML, CSS, and JavaScript. No frameworks, no build step, no npm. React is not used — the entire UI is rendered by lightweight DOM string injection functions that re-render on navigation. Google Fonts loads Bebas Neue, Syne, and Space Mono.

**AI** — Anthropic Claude API (`claude-haiku-4-5-20251001` for chat/test, `claude-haiku-4-5-20251001` for all generation). All API calls use server-sent events streaming so output renders token by token in real time.

**Payments** — Stripe JS v3 SDK (CDN) for card payments. PayPal JS SDK integration points are documented in the code.

**Storage** — Browser localStorage with namespaced keys per user email. All user data, session, profile, logs, weights, and exercise history live in the browser. No backend database.

**Auth** — Client-side only. Passwords are stored in localStorage (plaintext in this demo — in production, hash with bcrypt on a backend). Sessions persist via a JSON session key.

---

## Deployment

To deploy this as a real product you need three things added to the existing file:

**1. A backend server** (Node.js/Express or Python/FastAPI) that handles Stripe PaymentIntent creation, PayPal subscription creation, and Anthropic API calls with your secret key (so users never see your key — they only provide their own, or you proxy calls through your backend).

**2. Stripe secret key + webhook** on your server to listen for `payment_intent.succeeded` and `customer.subscription.created` events and update user plan status in a real database.

**3. A real database** (Supabase, Firebase, or PostgreSQL) to replace localStorage so data persists across devices and browsers.

The current file is fully functional as a demo, portfolio piece, or prototype that can be shown to investors or users immediately.

---

## File Stats

| Property | Value |
|---|---|
| File size | 105 KB |
| Lines of code | ~1,400 |
| Dependencies | Google Fonts, Stripe JS SDK |
| Pages / modules | 7 (Dashboard, BMI, Workout, Meals, Progress, Coach, Settings) |
| AI features | 6 (BMI analysis, workout gen, meal gen, coach chat, progress insights, exercise coaching) |
| Payment methods | 2 (Stripe card, PayPal) |
| Backend required | No (for demo) / Yes (for production payments) |
