# System Design: Parking Lot

## Problem Statement

Design a system to manage a parking lot. The system should be able to allocate parking spaces to vehicles, handle payments, track available spots, and ensure smooth operations of the parking facility.

### Requirements

#### Functional Requirements:
1. **Vehicle Entry**: Allow vehicles to enter the parking lot and assign them a parking spot.
2. **Vehicle Exit**: Handle vehicle exit by calculating parking charges and freeing up the parking spot.
3. **Track Available Spots**: Keep track of available parking spots in real-time.
4. **Handle Multiple Types of Vehicles**: Support different types of vehicles, such as cars, bikes, trucks, etc.
5. **Payments**: Calculate parking charges based on the time spent in the lot.
6. **Parking Ticket**: Generate a ticket upon entry that contains essential details (entry time, vehicle details, spot, etc.).

#### Non-Functional Requirements:
1. **Scalability**: The system should handle multiple parking lots, each with varying capacities.
2. **Fault Tolerance**: Ensure the system can handle node failures without losing data (e.g., parking history).
3. **Low Latency**: Ensure quick entry and exit processing to prevent bottlenecks.
4. **Real-Time Spot Updates**: Continuously update the status of parking spots across the system.

---

## Components of the System

The Parking Lot system can be broken down into the following components:

1. **Parking Spot Manager**: Manages parking spots, assigns spots to vehicles, and tracks available spots.
2. **Ticketing System**: Issues parking tickets with vehicle details, entry time, and allocated spot.
3. **Payment Processor**: Handles payments when a vehicle exits the parking lot.
4. **Parking Lot Manager**: Oversees multiple parking lots and manages the addition/removal of parking spots.
5. **Parking History Manager**: Tracks parking history for each vehicle for auditing and reporting.

---

## High-Level Design

1. **User Interface (UI)**: Allows attendants or users to input vehicle details and receive parking spot assignments. The interface also displays real-time updates of parking spot availability.
2. **API Layer**: Exposes APIs for:
   - Parking spot allocation (`POST /allocateSpot`)
   - Payment handling (`POST /payTicket`)
   - Vehicle exit and spot release (`POST /exitLot`)
3. **Service Layer**:
   - **Spot Manager**: Handles parking spot allocation, tracking of available and occupied spots.
   - **Payment Service**: Calculates parking fees based on the time spent and processes payments.
   - **Ticket Service**: Generates and tracks tickets for vehicles entering and exiting the lot.
4. **Storage Layer**:
   - **Database**: Stores parking lot data, spot availability, parking history, and ticket information.
   - **Cache (Optional)**: Cache frequently accessed data like available spots to reduce database load.

---

## Detailed Design

### 1. **Parking Spot Allocation**

The system should allocate spots to vehicles based on the type of vehicle (car, bike, truck) and the available spots in the parking lot. Different types of vehicles may require different spot sizes.

#### Spot Types:
- **Small**: For motorcycles and small vehicles.
- **Medium**: For standard cars.
- **Large**: For trucks or large vehicles.

#### Spot Allocation Process:
1. When a vehicle arrives, the system checks the vehicle type.
2. The system queries available spots of the appropriate size.
3. The spot is reserved for the vehicle, and a parking ticket is generated.

---

### 2. **Parking Ticket**

A parking ticket is issued upon entry. It contains the following information:
- **Ticket ID**: Unique identifier for the ticket.
- **Vehicle Information**: License plate number and vehicle type.
- **Entry Time**: Timestamp of when the vehicle entered.
- **Parking Spot**: The spot assigned to the vehicle.

#### Example Ticket:
```json
{
  "ticketId": "T12345",
  "vehicleNumber": "AB-1234",
  "vehicleType": "Car",
  "entryTime": "2024-10-01T09:00:00Z",
  "parkingSpot": "M12"
}
```

---

### 3. **Payments and Parking Charges**

The payment system should calculate the parking fee based on the time spent in the parking lot. The system charges vehicles based on an hourly rate.

#### Parking Fee Calculation:
- **Hourly Rate**: $5 for cars, $2 for bikes, $10 for trucks.
- **Overtime Fees**: For extended stays, a surcharge may be added.

Example fee calculation:
- A car that stays for 3.5 hours will be charged for 4 hours: `4 hours x $5 = $20`.

When a vehicle exits the lot:
1. The system calculates the parking fee based on the entry and exit timestamps.
2. The user pays the fee using a payment method (cash, card, mobile payment).
3. The spot is marked as available once payment is confirmed.

---

### 4. **Vehicle Exit and Spot Release**

When a vehicle exits the parking lot:
1. The user presents the parking ticket.
2. The system retrieves the entry time from the ticket.
3. The total parking fee is calculated, and the user pays.
4. The system marks the parking spot as **available**.
5. The exit time and payment information are stored in the database.

---

### 5. **Database Schema**

The database will store information about parking lots, spots, tickets, and payments.

#### Parking Spot Table:
| Field         | Type          | Description                              |
|---------------|---------------|------------------------------------------|
| `spot_id`     | String (PK)   | Unique identifier for the parking spot.  |
| `type`        | String        | The type of spot (small, medium, large). |
| `status`      | String        | Available or occupied.                   |
| `lot_id`      | String (FK)   | ID of the parking lot.                   |

#### Parking Ticket Table:
| Field         | Type          | Description                               |
|---------------|---------------|-------------------------------------------|
| `ticket_id`   | String (PK)   | Unique identifier for the parking ticket. |
| `vehicle_no`  | String        | License plate number of the vehicle.      |
| `entry_time`  | Timestamp     | Time the vehicle entered.                 |
| `exit_time`   | Timestamp     | Time the vehicle exited.                  |
| `spot_id`     | String (FK)   | The parking spot assigned.                |
| `payment`     | Decimal       | Total payment made for parking.           |

#### Parking Lot Table:
| Field         | Type          | Description                               |
|---------------|---------------|-------------------------------------------|
| `lot_id`      | String (PK)   | Unique identifier for the parking lot.    |
| `total_spots` | Integer       | Total number of parking spots.            |
| `available_spots` | Integer   | Number of currently available spots.      |

---

## Key Challenges

### 1. **Scalability**
- As the number of parking lots and vehicles grows, the system needs to handle more requests and data.
    - **Solution**: Use horizontal scaling and database sharding to distribute the load across multiple servers.

### 2. **Fault Tolerance**
- The system should handle failures gracefully, ensuring that parking information is not lost in case of crashes.
    - **Solution**: Use backups and failover mechanisms, such as storing critical information in distributed databases (e.g., Cassandra).

### 3. **Real-Time Spot Updates**
- Ensuring that the system updates parking spot availability in real-time can be challenging, especially under high load.
    - **Solution**: Use in-memory databases like **Redis** for faster access to parking spot availability data.

---

## Advanced Features (Optional)

### 1. **Parking Reservations**
- Allow users to reserve parking spots in advance through a mobile app.

### 2. **Parking Analytics**
- Provide reports on parking lot utilization, peak hours, and revenue generated over time.

### 3. **License Plate Recognition**
- Use license plate recognition cameras to automate vehicle entry and exit, making the process faster and reducing human intervention.

### 4. **Dynamic Pricing**
- Adjust parking fees dynamically based on demand (e.g., higher prices during peak hours).

---

## Data Flow

### 1. **Vehicle Entry**
- The user enters the parking lot.
- The parking spot manager assigns an available spot based on the vehicle type.
- A parking ticket is generated with the vehicle details, spot, and entry time.
- The parking spot is marked as **occupied**.

### 2. **Vehicle Exit**
- The user presents the parking ticket at the exit.
- The system calculates the parking fee based on the time spent.
- The user makes the payment.
- The parking spot is marked as **available** and the exit time is recorded.

---

## Scaling the System

### 1. **Distributed Parking Lots**
- Each parking lot operates as an independent unit but shares information with the central system.
- Use **microservices architecture** to separate components such as ticketing, payments, and spot management.

### 2. **Load Balancing**
- Use load balancers to distribute API requests across multiple servers to prevent bottlenecks and ensure high availability.

### 3. **Cache Frequently Accessed Data**
- Use a cache to store frequently accessed data, such as available parking spots, for faster access and reduced database load.

---

## Conclusion

Designing a parking lot system requires careful consideration of real-time updates, spot allocation, payment processing,

and scalability. By leveraging components such as ticketing services, parking spot management, and payments, this system can handle large numbers of vehicles efficiently while providing a seamless user experience.

---

## Further Reading

- [Parking Lot Management Systems](https://example-link.com/parking-lot-management)
- [Scalable Microservices for Parking Lots](https://example-link.com/microservices-parking-lot)
- [How to Design a Parking Lot System](https://example-link.com/design-parking-lot-system)