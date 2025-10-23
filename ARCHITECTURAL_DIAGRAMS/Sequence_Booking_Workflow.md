# Sequence Diagram: Vehicle Booking Workflow

This diagram shows the sequence of interactions between services for a successful vehicle booking and unlock workflow.

```
Customer App     API Gateway     Booking Svc     Vehicle Mgmt     Payment Svc         Kafka         Telemetry Svc     IoT Vehicle
       |                |                 |               |               |                 |                 |              |
       | POST /bookings |                 |               |               |                 |                 |              |
       |--------------->|                 |               |               |                 |                 |              |
       |                | createBooking   |               |               |                 |                 |              |
       |                |---------------->|               |               |                 |                 |              |
       |                |                 | reserveVehicle|               |                 |                 |              |
       |                |                 |-------------->|               |                 |                 |              |
       |                |                 |               | { success: true }               |                 |              |
       |                |                 |<--------------|               |                 |                 |              |
       |                |                 | preAuthorize  |               |                 |                 |              |
       |                |                 |------------------------------>|                 |                 |              |
       |                |                 |               |               | { success: true }                 |              |
       |                |                 |<------------------------------|                 |                 |              |
       | { CONFIRMED }  |                 |               |               |                 |                 |              |
       |<---------------|                 |               |               |                 |                 |              |
       |                |                 | Publish [booking.created]     |                 |                 |              |
       |                |                 |-------------------------------------------->|                     |              |
       |                |                 |               |               |                 |                 |              |
       |                |                 |               |               |                 |                 |              |
       | POST /unlock   |                 |               |               |                 |                 |              |
       |--------------->|                 |               |               |                 |                 |              |
       |                | unlockVehicle   |               |               |                 |                 |              |
       |                |---------------------------------------------------------------->|                   |              |
       |                |                 |               |               |                 |                 |              |
       |                |                 |               |               |                 | MQTT: 'UNLOCK'  |              |
       |                |                 |               |               |                 |---------------->|              |
       |                |                 |               |               |                 |                 |              |
       |                |                 |               |               |                 | MQTT: 'UNLOCKED'|              |
       |                |                 |               |               |                 |<----------------|              |
       |                |                 |               |               |                 |                 |              |
       |                |                 |               |               | Publish [vehicle.unlocked]        |              |
       |                |                 |               |               |                 |------------>|                 |              |
       |                |                 |               |               |                 |                 |              |
       | { SUCCESS }    |                 |               |               |                 |                 |              |
       |<-------------------------------------------------------------------------------|              |
       |                |                 |               |               |                 |                 |              |
```