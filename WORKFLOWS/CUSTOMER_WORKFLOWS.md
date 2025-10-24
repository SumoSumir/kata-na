### Customer Workflows

#### **Workflow: Vehicle Discovery and Booking**
```
┌─────────────────────────────────────────────────────────────────┐
│ 1. DISCOVER & SEARCH                                            │
├─────────────────────────────────────────────────────────────────┤
│ • User opens mobile app                                         │
│ • App displays map with available vehicles (real-time)          │
│ • Filters: vehicle type, distance, price, battery level         │
│ • AI shows personalized recommendations based on history        │
│ • Dynamic pricing displayed (surge/discount indicators)         │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│ 2. BOOKING                                                      │
├─────────────────────────────────────────────────────────────────┤
│ Bikes/Scooters:                                                 │
│ • Book up to 30 min advance, open-ended duration (max 12h)      │
│                                                                 │
│ Cars/Vans:                                                      │
│ • Book up to 7 days advance, fixed duration                     │
│ • Select pickup location and estimated return time              │
│                                                                 │
│ • System reserves vehicle, locks inventory                      │
│ • Payment method pre-authorized (not charged yet)               │
│ • Booking confirmation sent via push + email                    │
│ • Optional: allow bookings via a short conversational chat      │
│   with the in‑app AI assistant                                  │
│ • Optional: opt in for relocation incentive after mentioning    │
|   your drop point                                               |
┌─────────────────────────────────────────────────────────────────┐
│ 3. UNLOCK & START RENTAL                                        │
├─────────────────────────────────────────────────────────────────┤
│ • User arrives at vehicle location                              │
│ • NFC tap via app to unlock.                                    │
│ • Vehicle unlocked remotely by backend                          │
│ • Camera-based pickup verification:                             │
│   - User takes photo of vehicle condition                       │
│   - AI detects pre-existing damage                              │
│   - Verification score saved for dispute resolution             │
│ • GPS tracking begins, rental timer starts                      │
│ • Event published: `bookings.started`                           │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│ 4. DURING RENTAL                                                │
├─────────────────────────────────────────────────────────────────┤
│ • Real-time GPS telemetry (30s intervals)                       │
│ • Battery level monitoring (60s intervals)                      │
│ • Collision detection (edge processing, immediate alerts)       │
│ • Dynamic pricing updates if rental extended                    │
│ • In-app navigation and assistance available                    │
│ • Optional vehicle relocation incentive offered                 │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│ 5. RETURN & VERIFICATION                                        │
├─────────────────────────────────────────────────────────────────┤
│ • User navigates to designated parking spot                     │
│ • For Cars/Vans: Plug into EV charger                          │
│ • User takes return photo via app                               │
│ • AI verifies:                                                  │
│   - Correct parking location (GPS + photo)                      │
│   - Vehicle plugged in (for Cars and vans)                      │
│   - New damage detected (compared to pickup photo)              │
│ • NFC tap to lock vehicle                                       │
│ • Rental timer stops                                            │
│ • Event published: `bookings.completed`                         │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│ 6. PAYMENT & FEEDBACK                                           │
├─────────────────────────────────────────────────────────────────┤
│ • Calculate total cost:                                         │
│   - Base rate × duration                                        │
│   - Dynamic pricing adjustments                                 │
│   - Fines (wrong location, late return, damage)                │
│ • Charge payment method                                         │
│ • Receipt sent via email                                        │
│ • In-app feedback prompt:                                       │
│   - Rating (1-5 stars)                                          │
│   - Vehicle condition report                                    │
│   - Freeform comments (analyzed by NLP)                         │
│ • Relocation incentive offered if applicable                    │
└─────────────────────────────────────────────────────────────────┘
```

#### **Workflow: AI-Driven Relocation Incentives**
```
┌─────────────────────────────────────────────────────────────────┐
│ SCENARIO: User completes rental in high-supply zone             │
├─────────────────────────────────────────────────────────────────┤
│ 1. AI detects imbalance:                                        │
│    - Current zone has 15 available scooters                     │
│    - Nearby high-demand zone has 2 available scooters           │
│                                                                 │
│ 2. Incentive engine calculates offer:                           │
│    - Distance: 2.5 km                                           │
│    - Urgency: High (peak morning commute)                       │
│    - Incentive: "50% off + €3 credit if you drop off in         │
│      Downtown by 9 AM"                                          │
│                                                                 │
│ 3. User accepts optional incentive:                             │
│    - App provides navigation to incentive dropoff zone          │
│    - Real-time progress tracking                                │
│                                                                 │
│ 4. Completion and reward:                                       │
│    - Dropoff verified in correct zone                           │
│    - Discount applied + credit added to account                 │
│    - Gamification: "You've completed 5 relocations this month!" │
└─────────────────────────────────────────────────────────────────┘
```
