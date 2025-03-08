# Ticketmaster
- Platforms: ticketmaster, bookmyshow
- Online ticketing system to sell movie tickets
- Customers can browse through moviews currently being played
  - And book seats anywhere anytime

## Requirements
- Functional Requirements
  - List cities where the affiliated cinemas are located
    - Once the user selects the city, display the movies that are available there
  - After selecting the movie
    - Display the cinemas running that movie and the available show times
  - Show the seating arrangement and the available seats
    - User can select multiple seats based on their preference and book them
  - After selecting the seats
    - Put a hold on the seats for 5 mins before finalizing and making payment
  - User should be able to wait if there is a chance that the seats might become available
    - For example, when the holds by other users expire
    - Waiting customers should be server in a fair, first come, first server manner
- Non-Functional Requirements
  - Highly concurrent
    - There will be multiple booking requests for the same seat at the same time
    - The service should handle this gracefully and fairly
  - There will be financial transactions
    - So system should be secure & database should be ACID compliant
  - Highly scalable and highly available
    - Since the traffic would spike on popular and much-awaited movie releases

## Considerations
- There should not be any partial order, either book all the tickets or nothing
- Fairness is mandatory for the system
- To stop system abuse, restrict users from booking more than 10 seats at a time

## Estimation
- Traffic
  - Page views: 3 B/month
  - Tickets sold: 10 M/month
- Business Size
  - Number of cities: 500
  - Cinemas per city: 10
  - Seats in each cinema: 2000
  - Shows per day: 2
- Storage
  - Database fields
    - For each booking: 50 bytes
    - For each movie: 25 bytes
    - For each cinema: 25 bytes
  - Daily Storage: 500 cities * 10 cinemas * 2000 seats * 2 shows * 100 bytes = 2 GB/day
  - Storage for 5 years: 3.6 TB

## API
- api_key can be used for throttling
- search_movies
  - api_key, keyword, city, zipcode, lat, long, radius, start_time, end_time
  - include_spell_check, results_per_page, sorting_order
- reserve_seats (returns whether reservation succeeded or failed)
  - api_key, session_id, movie_id, show_id, seat_numbers

## Database
- Relations
  - Each city can have multiple cinemas
  - Each cinema will have multiple halls
  - Each movie will have many shows
  - Each show will have multiple bookings
  - Each user can have multiple bookings
- Schema
  - User (id, name, email, phone)
  - Cinema (id, address_id, name, total_cinema_halls)
  - Address (id, city, state, zipcode)
  - Hall (id, cinema_id, regular_seat_capacity, premium_seat_capacity)
  - Seat (id, hall_id, seat_number, seat_type)
  - Show (id, hall_id, movie_id, start_time, end_time)
  - ShowSeat (id, seat_id, show_id, booking_id, status)
  - Movie (id, title, description, duration, genre, language, release_date, country)
  - Pricing (id, show_id, regular_price, premium_price)
  - Booking (id, user_id, show_id, number_of_seats, payment_id, status)
  - Payment (id, booking_id, transaction_id, coupon_id, payment_method, status, amount)

# Component Design
## Workflow
- User searches for a movie and selects a movie
- User is shown the available shows and selects a show
- User specifies the number of seats to be reserved
- If the required number of seats are available, user is shown the seat map
  - User selects the seats and the system tries to reserve them
    - If seats are reserved successfully
      - User gets five minutes to pay
      - After payment, booking is marked complete
      - If payment is not successful, seats become available for other users
    - Else if the specified seats are no longer available
      - Redirect to the seat map to choose different seats
    - Else if the show is full
      - If there are hold orders present
        - Redirect to the waiting page
        - If the seats become available, redirect to the seat map
        - If the seats get booked or there are fewer seats available/hold
          - Show the error message and redirect to the seat map
        - If the user cancels or the request timeouts
          - Show the error message and redirect to the movie search page
      - Else show the error message and redirect to the movie search page

## Reservation Tracker
- We need to track
  - All the active reservations that haven’t been booked yet
    - And remove any expired reservations from the system
  - All the waiting customers
    - If the seats become available, notify the longest waiting user to choose seats

### Active Reservation Service
- Each active reservation will have a expiry time
  - Show a timer to user for a better user experience
  - The timer can be a little out of sync, so add a buffer of 5s
- We can keep active reservations of a show in memory in addition to the database
  - We can use data structures like Linked Hash Map or Tree Map
    - That allows to jump to any reservation to remove it when the booking is complete
    - { show_id: linked_hash_map { booking_id, created_at } }
  - Head of the map will always point to the oldest reservation
    - So that it can be expired after the timeout (FCFS)
- Database
  - Store the reservation in the Booking table
  - And keep track of the expiry in a timestamp column
  - Status can have these values: reserved, booked, expired
- After the seats have been booked or the request expires
  - Update the database and remove the reservation from the map
  - And send a signal to the Waiting User Service

### Waiting User Service
- Just like Active Reservation Service
  - We can keep all the waiting users of a show in memory in a Linked Hash Map
    - { show_id: linked_hash_map { user_id, wait_start_time }
  - And the head of the map will point to the longest waiting user (FCFS)
  - When a user cancels request, we can jump and remove it from the map
- Clients can use long polling for keeping themselves updated about the reservation status
  - When the seats become available, the server can use this request to notify the user

## Concurrency
- No two users should be able to book the same seat
- We can use transactions in the SQL database to avoid any clashes
- E.g. Use Transaction Isolation Levels to lock the rows before updating
  ```
  SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
  BEGIN TRANSACTION;
  select * from show_seats
  where show_id = 99 and show_seat_id in (54, 55, 56) and status = 'available';
  -- If the number of rows returned above is 3, i.e. all the 3 seats are reserved
  -- return success else failure
  update show_seats ...;
  update bookings ...;
  COMMIT TRANSACTION;
  ```
- Reading rows within a transaction gets a write lock on them
  - And can’t be updated by anyone else
- Serializable is the highest isolation level
  - And guarantees safety from dirty, non-repeatable and phantom reads
- Once the transaction is successful
  - We can start tracking the reservation in Active Reservation Service

## Reservation Overview
### On Expiration
- Whenever a reservation is expired, the server holding it will carry these actions
- Update database to mark the booking expired (or remove it)
- Update the status of seats in show_seats table
- Remove the reservation from the Linked Hash Map
- Notify the user that their reservation has expired
- Figure out the Waiting User Service servers holding the waiting users
  - By using the consistent hashing scheme
- Broadcast a message to all Waiting User Service servers
  - To figure out the longest waiting user
- If the seats become available, request these server to process the user request

### On Success
- Whenever a reservation is successful, following things will happen
- The Waiting User Service holding the reservation
  - Sends a message to all the servers holding waiting users of that show
  - To expire all waiting users who need more seats than the available seats
- Upon receiving the above message, all the servers holding the waiting users
  - Will query the database to find how many free seats are available now
    - Using a database cache would help to run this query only once
  - Expire all waiting users wanting more than the available seats
    - The server will iterate through the Linked HashMap of all the waiting users

# Scalability
## Fault Tolerance
- What happens when ActiveReservationsService or WaitingUsersService crashes
  - If Active Reservation Service crashes
    - We can read all the active reservations from Booking table
    - Or we can use a master-slave configuration
  - If Waiting User Service crashes
    - We are not storing the waiting users in the database
    - So the only option is have a master-slave configuration
- Similarly, we can have a master-slave setup for the databases

## Database Partitioning
- Based on Movie Id
  - All the shows of a movie will be on a single server
  - For a very popular movie, this could cause an overload on that server
- Based on Show Id
  - The load gets distributed among different servers

## Load Balancing
- The web servers will manage all the active user sessions
  - And the communication with the users
- We can use Consistent Hashing based on the show id
  - To allocate servers for Active Reservation Service and Waiting User Service
  - This way all reservations and waiting users of a particular show
    - Will be handled by a certain set of servers
- Let’s assume for load balancing
  - The consistent hashing allocates three servers for any show
  - So when the reservation expires
    - Only the server holding that reservation will carry out the required actions
