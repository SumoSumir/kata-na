# Functional Requirements

Functional requirements define the specific behaviors and functions of the system. They describe what the system must do to meet user and business needs.

---

### Customer-Facing Features

-   **User Registration & Profile Management:**
    -   Users must be able to register, log in, and manage their profiles.
    -   KYC (Know Your Customer) document upload and verification.
    -   Management of payment methods.
    -   View ride history and receipts.

-   **Vehicle Discovery & Booking:**
    -   Display a real-time map of available vehicles.
    -   Search and filter vehicles by type, proximity, battery level, and price.
    -   Use conversational AI Assistant for booking your ride via text or voice.
    -   Support Auto booking by vehicle type, time in stipulated price ranges.
    -   Receive personalized vehicle and route recommendations.
    -   Book vehicles for immediate use (scooters/eBikes) or schedule in advance (cars/vans).
    -   Receive booking confirmations via push notifications and email.

-   **Rental & Vehicle Interaction:**
    -   Unlock vehicles using QR code or NFC.
    -   Perform mandatory pre-ride vehicle condition verification via photo upload.
    -   Lock vehicles to end a rental period.
    -   Perform post-ride vehicle condition verification.

-   **Pricing & Payments:**
    -   View dynamic, real-time pricing before booking.
    -   Receive and accept AI-driven relocation incentives (discounts/credits).
    -   Automated payment processing upon ride completion.
    -   View detailed cost breakdowns and receipts.

-   **In-Ride Experience & Support:**
    -   View real-time trip progress and telemetry on the map.
    -   Access in-app navigation.
    -   Report issues or request support via a conversational AI assistant.
    -   Receive safety alerts (e.g., geofence boundaries, low battery).

-   **Feedback & Gamification:**
    -   Rate rides and provide feedback on vehicle condition.
    -   Track and redeem rewards earned from relocation incentives.

---

### Staff-Facing & Operational Features (Operations Dashboard)

-   **Fleet Management:**
    -   View a real-time dashboard of the entire fleet's status, location, and health.
    -   Manually track and manage individual vehicles.
    -   Define and manage geofenced operational zones and parking spots.

-   **AI-Driven Task Management:**
    -   Receive an AI-prioritized list of daily tasks (battery swaps, rebalancing, maintenance).
    -   View optimized routes for completing assigned tasks.
    -   Track task progress and log completion.

-   **Maintenance & Repair:**
    -   Receive predictive maintenance alerts with failure diagnostics.
    -   Manage the lifecycle of maintenance tickets from creation to resolution.
    -   Log parts used and actions taken for each repair.
    -   Mark vehicles as "out of service" or "available".

-   **Incident Management:**
    -   Receive real-time alerts for detected collisions or accidents.
    -   Access incident details, including telemetry snapshots and customer information.
    -   Dispatch field staff to incident locations.

-   **User & Dispute Management:**
    -   Review customer-submitted damage photos and resolve disputes.
    -   Handle customer support escalations from the AI assistant.
    -   Issue refunds or apply fines based on ride reviews.

---

### System & Core Functional Requirements

-   **AI/ML & Data Services:**
    -   **Demand Forecasting:** The system must generate demand predictions for specific zones and time intervals.
    -   **Dynamic Pricing:** The system must calculate and apply dynamic pricing based on supply, demand, and other factors.
    -   **Predictive Maintenance:** The system must analyze telemetry to predict component failures.
    -   **Damage Detection:** The system must automatically detect and classify vehicle damage from user-submitted photos.
    -   **Relocation Incentives:** The system must identify opportunities and calculate optimal incentives for user-led vehicle rebalancing.
    -   **Conversational AI:** The system must understand and respond to user queries via a chat/voice interface.

-   **Telemetry & IoT:**
    -   Ingest and process high-frequency telemetry data from vehicles (GPS, battery, IMU).
    -   Execute edge-based ML models for real-time collision detection.
    -   Securely authenticate and communicate with IoT devices.
    -   Support over-the-air (OTA) updates for vehicle firmware and ML models.

-   **Administration & Configuration (Admin Portal):**
    -   Configure system-wide settings (e.g., pricing rules, geofences).
    -   Manage user roles and permissions.
    -   Monitor system health and performance through dedicated dashboards.
    -   Onboard new regions, vehicle types, and external API integrations.

-   **Authentication & Authorization:**
    -   The system must secure all endpoints and services.
    -   Implement role-based access control (RBAC) for customer, staff, and admin roles.
