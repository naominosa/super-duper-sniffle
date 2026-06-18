# Omie's Swift Logistics 🏍️

A high-performance, user-friendly logistics web application designed for the Nigerian market. Enables users to instantly book dispatch riders for deliveries with real-time tracking, AI-powered support, and seamless payment integration.

## 🎯 Project Vision

Modeled after platforms like Bolt, Omie's Swift Logistics brings frictionless dispatch rider booking to local logistics with:
- **Instant booking** with minimal clicks (3-step process)
- **Real-time tracking** with integrated mapping
- **24/7 AI customer service** with human touch
- **Dynamic pricing** and transparent cost calculation
- **Secure payments** and instant rider notifications

---

## 📋 Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Tech Stack](#tech-stack)
3. [Project Structure](#project-structure)
4. [UX Flow](#ux-flow)
5. [Design System](#design-system)
6. [AI & Chatbot Strategy](#ai--chatbot-strategy)
7. [Getting Started](#getting-started)
8. [Deployment](#deployment)

---

## 🏗️ Architecture Overview

### High-Level System Design

```
┌─────────────────────────────────────────────────────────────┐
│                     CLIENT LAYER (Web)                       │
│  React 18 + TypeScript + Tailwind CSS + Framer Motion      │
└────────────────┬────────────────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────────────────┐
│              API GATEWAY & SERVICE LAYER                     │
│  Node.js/Express + GraphQL + REST endpoints                │
└────────────────┬────────────────────────────────────────────┘
                 │
    ┌────────────┼────────────┬──────────────┐
    │            │            │              │
┌───▼──┐  ┌──────▼──┐  ┌─────▼────┐  ┌───────▼─────┐
│Auth  │  │Booking  │  │Tracking  │  │ Payments    │
│Svc   │  │Engine   │  │ Service  │  │ Service     │
└───┬──┘  └──────┬──┘  └─────┬────┘  └───────┬─────┘
    │            │            │              │
    └────────────┼────────────┼──────────────┘
                 │
┌────────────────▼────────────────────────────────────────────┐
│              DATABASE & CACHE LAYER                          │
│  Supabase/PostgreSQL + Redis + Mapbox SDK                  │
└─────────────────────────────────────────────────────────────┘
                 │
    ┌────────────┼────────────┐
    │            │            │
┌───▼────┐  ┌───▼─────┐  ┌──▼────┐
│ AI/ML  │  │ Real-   │  │WebSocket
│ Engine │  │ time    │  │ Server
│(Claude)│  │ Updates │  │(Socket.io)
└────────┘  └─────────┘  └────────┘
```

### Core Services

| Service | Purpose | Tech |
|---------|---------|------|
| **Authentication** | User login, role management | Firebase Auth / Supabase |
| **Booking Engine** | Real-time rider matching, pricing | Node.js + Bull queue |
| **Tracking Service** | Live GPS tracking, ETAs | WebSocket + Mapbox |
| **Payment Service** | Secure transactions | Stripe / Paystack |
| **AI Chatbot** | 24/7 support with escalation | Claude API + custom context |
| **Notification Hub** | SMS/Email/Push notifications | Twilio + Firebase Cloud Messaging |

---

## 💻 Tech Stack

### Frontend
- **Framework**: React 18 with TypeScript
- **Styling**: Tailwind CSS + Shadcn/ui components
- **State Management**: Zustand (lightweight) or Redux Toolkit
- **Mapping**: Mapbox GL JS (real-time tracking)
- **Animations**: Framer Motion (smooth UI transitions)
- **HTTP Client**: TanStack Query + Axios
- **Build Tool**: Vite (fast development)

### Backend
- **Runtime**: Node.js (v18+)
- **Framework**: Express.js or Fastify
- **API**: GraphQL (Apollo) + REST endpoints
- **Task Queue**: Bull/Redis for background jobs
- **Real-time**: Socket.io (rider updates, notifications)
- **Authentication**: JWT + OAuth (Google, Apple)

### Database & Storage
- **Primary DB**: PostgreSQL (Supabase)
- **Cache**: Redis (session management, rate limiting)
- **File Storage**: Supabase Storage / AWS S3
- **Search**: Elasticsearch (rider search, history)

### External Services
- **Maps**: Mapbox API (tracking, routing)
- **AI/Chatbot**: Claude API 3.5 Sonnet (context-aware support)
- **Payments**: Stripe / Paystack (Nigeria-optimized)
- **SMS/Push**: Twilio + Firebase Cloud Messaging
- **Analytics**: PostHog / Mixpanel

### DevOps & Deployment
- **Hosting**: Vercel (frontend) + Railway/Render (backend)
- **CI/CD**: GitHub Actions
- **Monitoring**: Sentry + DataDog
- **Containerization**: Docker + Docker Compose

---

## 📁 Project Structure

```
omies-swift-logistics/
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── Map/
│   │   │   │   ├── LiveTrackingMap.tsx
│   │   │   │   └── PickupDropoffSelector.tsx
│   │   │   ├── Booking/
│   │   │   │   ├── BookingFlow.tsx
│   │   │   │   ├── RiderSelection.tsx
│   │   │   │   └── ConfirmationScreen.tsx
│   │   │   ├── Chat/
│   │   │   │   ├── AIAssistant.tsx
│   │   │   │   ├── ConversationBubble.tsx
│   │   │   │   └── EscalationModal.tsx
│   │   │   └── Common/
│   │   │       ├── Header.tsx
│   │   │       ├── Navigation.tsx
│   │   │       └── LoadingStates.tsx
│   │   ├── pages/
│   │   │   ├── HomePage.tsx
│   │   │   ├── BookingPage.tsx
│   │   │   ├── OrderTracking.tsx
│   │   │   ├── OrderHistory.tsx
│   │   │   └── ProfileSettings.tsx
│   │   ├── hooks/
│   │   │   ├── useBooking.ts
│   │   │   ├── useTracking.ts
│   │   │   ├── useChat.ts
│   │   │   └── usePayment.ts
│   │   ├── store/
│   │   │   ├── authStore.ts
│   │   │   ├── bookingStore.ts
│   │   │   └── uiStore.ts
│   │   ├── utils/
│   │   │   ├── api.ts
│   │   │   ├── validators.ts
│   │   │   ├── formatters.ts
│   │   │   └── constants.ts
│   │   ├── styles/
│   │   │   ├── tailwind.config.js
│   │   │   └── globals.css
│   │   ├── App.tsx
│   │   └── main.tsx
│   ├── public/
│   ├── package.json
│   ├── vite.config.ts
│   └── tsconfig.json
│
├── backend/
│   ├── src/
│   │   ├── services/
│   │   │   ├── authService.ts
│   │   │   ├── bookingService.ts
│   │   │   ├── riderService.ts
│   │   │   ├── paymentService.ts
│   │   │   ├── notificationService.ts
│   │   │   └── aiChatbotService.ts
│   │   ├── routes/
│   │   │   ├── auth.ts
│   │   │   ├── bookings.ts
│   │   │   ├── riders.ts
│   │   │   ├── payments.ts
│   │   │   ├── tracking.ts
│   │   │   └── support.ts
│   │   ├── middleware/
│   │   │   ├── auth.ts
│   │   │   ├── errorHandler.ts
│   │   │   ├── rateLimiter.ts
│   │   │   └── validation.ts
│   │   ├── models/
│   │   │   ├── User.ts
│   │   │   ├── Booking.ts
│   │   │   ├── Rider.ts
│   │   │   ├── Payment.ts
│   │   │   └── ChatSession.ts
│   │   ├── jobs/
│   │   │   ├── matchRiders.ts
│   │   │   ├── sendNotifications.ts
│   │   │   ├── updateTracking.ts
│   │   │   └── escalateSupport.ts
│   │   ├── utils/
│   │   │   ├── db.ts
│   │   │   ├── redis.ts
│   │   │   ├── logger.ts
│   │   │   └── validators.ts
│   │   ├── websocket/
│   │   │   ├── handlers.ts
│   │   │   └── namespaces.ts
│   │   ├── app.ts
│   │   └── server.ts
│   ├── .env.example
│   ├── package.json
│   ├── tsconfig.json
│   └── docker-compose.yml
│
├── docs/
│   ├── ARCHITECTURE.md
│   ├── API_DOCUMENTATION.md
│   ├── DESIGN_SYSTEM.md
│   ├── AI_STRATEGY.md
│   ├── DATABASE_SCHEMA.md
│   └── DEPLOYMENT_GUIDE.md
│
├── .github/
│   └── workflows/
│       ├── frontend-ci.yml
│       ├── backend-ci.yml
│       └── deploy.yml
│
└── README.md
```

---

## 🎯 UX Flow

### Primary User Journey: Book a Rider (3 Steps)

```
┌─────────────────┐
│   Open App      │
│   (Home)        │
└────────┬────────┘
         │
         ▼
┌─────────────────────────────────────────┐
│  STEP 1: SET LOCATION & DESTINATION     │
│  ┌─────────────────────────────────────┐│
│  │ • Interactive map with current      ││
│  │   location pinned                   ││
│  │ • Search bar: "Pick a location"     ││
│  │ • "Use Current Location" button     ││
│  │ • Tap map to select pickup point    ││
│  │ • Search/Enter destination address  ││
│  └─────────────────────────────────────┘│
│  [CONTINUE] ───────►                    │
└─────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────��───┐
│  STEP 2: CONFIRM DETAILS & PRICING      │
│  ┌─────────────────────────────────────┐│
│  │ From: [Address] - [Distance Icon]   ││
│  │ To:   [Address]                     ││
│  │                                     ││
│  │ Item Type:  [Dropdown]              ││
│  │ Item Weight: [Slider] kg            ││
│  │                                     ││
│  │ Estimated Fare: ₦2,450              ││
│  │ • Base: ₦1,200                      ││
│  │ • Distance: ₦800 (5km)              ││
│  │ • Surge: ₦450 (Peak hours)          ││
│  │                                     ││
│  │ Preferred Rider: [Rider Profile]    ││
│  └─────────────────────────────────────┘│
│  [CONFIRM & PROCEED] ───────►           │
└─────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────┐
│  STEP 3: PAYMENT & CONFIRMATION         │
│  ┌─────────────────────────────────────┐│
│  │ Total: ₦2,450                       ││
│  │ Payment Method: [Saved Card] ✓      ││
│  │ (or add new payment method)          ││
│  │                                     ││
│  │ ☐ Add tip (optional)                ││
│  │ ☐ Schedule for later                ││
│  │                                     ││
│  │ Special Notes:                      ││
│  │ [Fragile items / Handle with care]  ││
│  └─────────────────────────────────────┘│
│  [BOOK RIDER] ───────►                  │
└─────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────┐
│  CONFIRMATION SCREEN                    │
│  ┌─────────────────────────────────────┐│
│  │ ✓ Booking Confirmed!                ││
│  │ Order ID: #OS-2024-0452             ││
│  │                                     ││
│  │ Rider arriving in: 4 mins           ││
│  │ [Live Map with Rider Location]      ││
│  │                                     ││
│  │ Driver: Chukwu M.                   ││
│  │ Rating: ⭐⭐⭐⭐⭐ (247 reviews)      ││
│  │ Phone: [Call] [Message]             ││
│  │                                     ││
│  │ [Chat with Support]                 ││
│  └─────────────────────────────────────┘│
└─────────────────────────────────────────┘
```

---

## 🎨 Design System

See `docs/DESIGN_SYSTEM.md` for complete color palette, typography, components, and guidelines.

### Color Palette Preview

| Color | Hex | Usage |
|-------|-----|-------|
| Primary Blue | `#1F5FFF` | CTAs, active states |
| Secondary Yellow | `#FFD700` | Highlights, badges |
| Success Green | `#00D084` | Confirmations |
| Warning Orange | `#FF6B35` | Alerts, surge pricing |
| Danger Red | `#E63946` | Errors |

---

## 🤖 AI & Chatbot Strategy

See `docs/AI_STRATEGY.md` for complete implementation details.

**Key Features:**
- Context-aware Claude 3.5 Sonnet integration
- Real-time sentiment analysis with smart escalation
- Human agent handoff within 2 minutes
- Proactive support (delay alerts, payment recovery)
- Nigerian English with empathetic tone

---

## 🚀 Getting Started

### Prerequisites
- Node.js 18+
- PostgreSQL 14+
- Redis 7+
- Mapbox API key
- Claude API key (Anthropic)
- Stripe/Paystack API keys

### Installation

**Frontend:**
```bash
cd frontend
npm install
npm run dev
```

**Backend:**
```bash
cd backend
npm install
cp .env.example .env
npm run dev
```

**Database Setup:**
```bash
npm run migrate
npm run seed
```

---

## 📦 Deployment

- **Frontend**: Vercel (automatic deploys on push to main)
- **Backend**: Railway or Render (Docker-based)
- **Database**: Supabase (managed PostgreSQL)
- **CDN**: Cloudflare (for static assets)

See `docs/DEPLOYMENT_GUIDE.md` for detailed steps.

---

## 📚 Documentation

- `docs/ARCHITECTURE.md` - Detailed system design
- `docs/API_DOCUMENTATION.md` - REST/GraphQL endpoints
- `docs/DESIGN_SYSTEM.md` - Complete UI component specs
- `docs/AI_STRATEGY.md` - Chatbot implementation guide
- `docs/DATABASE_SCHEMA.md` - Entity relationships
- `docs/DEPLOYMENT_GUIDE.md` - Production checklist

---

## 📞 Support

For questions or contributions, contact the team at hello@omiesswift.ng

---

**Status**: 🚧 In Development | **Last Updated**: June 2026