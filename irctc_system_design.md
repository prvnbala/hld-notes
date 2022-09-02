# IRCTC System design

> In a interview situation, make sure that you are leading the design discussion.

## 1. Requirement gathering
What is going to be our MVP (Minimum Viable Product)?

- Search for trains 
  - From source to destination
  - Response should list trains, respective arrival & departure times, ticket cost, etc.

- Check seat availability
  - Given a train_id, source and destination, date, class -> check availability of seats.

- Book seats
  - train_id, source, destination, date, class, passenger details

- Accept payments
  - Using a payment gateway.

### Future requirements:
- Consider tatkal scenario
- Consider train scheduling
- Cancellation of tickets

## 2. Scale estimation
- Traffic estimation: 
    - In case of highest traffic (queries per second) - how many server do I need?
- Storage estimation: 
    - How much data I need to persist for a reasonable period of time (like 5 or 10 years).
    - Helps in determining need for sharding? We need sharding for high volumes of data.
    - Determines db choice between SQL and NoSQL. NoSQL could be relatively better in sharding.
- Usage pattern: 
    - Read-heavy / write-heavy / mixed-workload system?
    - Determines many things like caching strategy, db choice.

### Assumptions of estimation: 
#### Traffic
- No. of trains running: 5000 trains per day
- No. of bogeys in each train : 20
- No. of seats in each bogey : 84
- We might also consider that there could be 20% bookings that can go waitlisted : 1.2
- No. of seats that could be booked = 6000 * 20 * 1.2= 10 million per day. 
- We could think, most booking happen (ex. 60%) during a peak time in a day like 10 AM to 11 AM. 
- Therefore, peak traffic could be 6 million per hour - booking (write requests).
- People could also search for 3 to 4 trains for availability before booking, so 24 million read requests per hours. 

#### Storage:
- Details to storage for each booking: passenger_name (128B), passenger_id(8B), date(8B), time(8B), train_id(8B), class (1B), seat_no (4B), pnr (8B), status (4B). 
- Total memory per record = 200 Bytes.
- Per day storage requirement = 10 million* 200 Byte = 2 GB.
- Ticket booking will happen only for present and future. Past data can be moved to archival databases.
- If we allow 90 day advance booking, we need to track data for 90 days only. 
- Storage needed for 90 days = 90 * 2GB = 180 GB. If we add buffer of factor 2, still requirement is 360 GB. 
- Therefore no sharding needed(less than 2TB), can work with SQL databases.


> Traffic estimation: 6 million writes/hr and 24 million reads/hr, high concurrency.

> Storage estimation: SQL store, 360 GB, no sharding.

> Workload type: Mixed workload, high read, high write.

## 3. Design goals:
- Search trains
    - highly available
    - Read : High
- Train - check availability
    - eventually consistent
    - Read : Moderate
- Train - book seat
    - highly consistent
    - Write : Moderate

## 4. System design:
### Search trains:
- The number of trains and the train routes mostly remain constant. Changes are infrequent. 
- If each train has 100 stops also, train-stop mapping table can at max have 500,000 entries. 
- This can be easily stored in __SQL database__, or even a __cache__. 
- So, given src, destination, date, etc. -> searching available trains can be done very quickly.

### Check availability
- 

