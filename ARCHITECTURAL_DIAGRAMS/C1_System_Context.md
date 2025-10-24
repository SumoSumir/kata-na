# C1: System Context Diagram

This diagram provides a high-level overview of the MobilityCorp platform and its interactions with external users and systems.

```
┌────────────────────────────────────────────────────────────────────────┐
│                         SYSTEM CONTEXT                                 │
└────────────────────────────────────────────────────────────────────────┘

         ┌─────────────┐                    ┌─────────────┐
         │  Customer   │                    │    Staff    │
         │  (Mobile    │                    │ (Operations │
         │   App)      │                    │  Dashboard) │
         └──────┬──────┘                    └──────┬──────┘
                │  ▲                               │  ▲
                │  │                               │  │
                │  │ Booking, Unlock,              │  │ Task Management,
                │  │ Payment, Feedback             │  │ Maintenance Logs
                │  │                               │  │
                │  └─ Smart Notifications,         │  └─ Alerts & Assignments
                │     Recommendations              │
                ▼                                  ▼
        ┌───────────────────────────────────────────────────┐
        │                                                   │
        │         MobilityCorp Platform                     │
        │  (Microservices + AI/ML + IoT + Events)           │
        │  ⭐ Intelligent Notification Service              │
        │                                                   │
        └───────────────────────────────────────────────────┘
                │              │              │
                │              │              │
        ┌───────▼──────┐  ┌────▼─────┐  ┌────▼─────────┐
        │   IoT        │  │ External │  │  Payment     │
        │  Vehicles    │  │   APIs   │  │  Gateway     │
        │ (Telemetry,  │  │ (Weather,│  │  (Stripe,    │
        │  GPS, NFC)   │  │  Events, │  │   Adyen)     │
        └──────────────┘  │ Holidays)│  └──────────────┘
                          └──────────┘
```

### Key External Systems:

-   **Customers:** Mobile app users booking and using vehicles.
-   **Staff:** Operations team managing the fleet via a web dashboard.
-   **IoT Vehicles:** The fleet of electric scooters, eBikes, cars, and vans with embedded sensors.
-   **External APIs:** Third-party services providing contextual data like weather, public holidays, and local events.
-   **Payment Gateway:** A third-party service (e.g., Stripe, Adyen) for processing customer payments.
