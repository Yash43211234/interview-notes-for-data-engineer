# Ride-Sharing Data Warehouse Design

## 1. Introduction

A **Ride-Sharing Data Warehouse** is a centralized analytical system that stores and organizes data from a ride-booking platform.

It helps businesses analyze:

- Total rides and completed trips.
- Ride revenue and platform earnings.
- Driver performance and availability.
- Passenger behavior.
- Pickup and drop-off locations.
- Ride cancellations.
- Driver acceptance rates.
- Average trip duration and distance.
- Surge pricing and discounts.
- Payment success and refunds.

A Data Engineer builds pipelines to collect data from operational systems, transform it into reliable analytical datasets, and make it available for reporting.

### Example Business Scenario

Consider a ride-sharing platform that connects passengers with drivers.

The business team wants to answer:

1. How many rides are booked daily?
2. How many rides are completed?
3. What is the cancellation rate?
4. Which cities generate the most revenue?
5. Which drivers complete the most trips?
6. What is the average ride duration?
7. What is the average distance traveled?
8. Which pickup locations have the highest demand?
9. What percentage of ride requests are accepted?
10. How much does the platform earn from each completed ride?

The warehouse must support these questions while preserving the correct grain of every business process.

---

## 2. Understand the Source Systems

A ride-sharing platform may have multiple operational systems.

| Source System | Data Generated |
|---|---|
| Passenger System | Passenger profiles and account details |
| Driver System | Driver profiles and vehicle information |
| Ride Booking System | Ride requests and trip details |
| Driver Matching System | Driver search and assignment events |
| Location System | GPS coordinates and location events |
| Payment System | Payment attempts and transactions |
| Pricing System | Base fare, surge pricing, and fare calculation |
| Promotion System | Coupons and discounts |
| Rating System | Passenger and driver ratings |
| Support System | Complaints and support tickets |

### Example Source Tables

**passengers**

| passenger_id | passenger_name | city |
|---|---|---|
| P001 | Rahul | Kolkata |
| P002 | Priya | Delhi |

**drivers**

| driver_id | driver_name | city | vehicle_type |
|---|---|---|---|
| D001 | Amit | Kolkata | Sedan |
| D002 | Ravi | Delhi | SUV |

**rides**

| ride_id | passenger_id | request_time | ride_status |
|---|---|---|---|
| R001 | P001 | 2026-10-01 09:00 | Completed |
| R002 | P002 | 2026-10-01 10:00 | Cancelled |

**ride_assignments**

| assignment_id | ride_id | driver_id | assigned_at | assignment_status |
|---|---|---|---|---|
| A001 | R001 | D001 | 09:02 | Accepted |
| A002 | R002 | D002 | 10:03 | Rejected |

**payments**

| payment_id | ride_id | payment_amount | payment_status |
|---|---|---:|---|
| PAY001 | R001 | 250 | Success |
| PAY002 | R002 | 0 | Not Charged |

These are simplified examples. Real systems may contain multiple driver assignment attempts, payment attempts, GPS events, fare adjustments, and cancellation events for one ride.

---

## 3. Gather Business Requirements

Before designing the warehouse, identify the questions that business teams need to answer.

### Ride Analytics

- How many rides are requested daily?
- How many rides are completed?
- How many rides are cancelled?
- What is the average trip duration?
- What is the average trip distance?
- What are the peak booking hours?

### Passenger Analytics

- Who are the most frequent passengers?
- How many new passengers register each month?
- Which passengers have stopped using the platform?
- Which locations generate the most passenger demand?

### Driver Analytics

- Which drivers complete the most trips?
- What is the driver acceptance rate?
- What is the average driver rating?
- How much time do drivers spend waiting for requests?
- Which drivers have high cancellation rates?

### Revenue Analytics

- What is the total gross booking value?
- What is the platform's net revenue?
- How much is paid to drivers?
- What is the total discount amount?
- How much revenue is refunded?

### Location Analytics

- Which pickup areas have the highest demand?
- Which drop-off areas receive the most rides?
- Which locations have a shortage of available drivers?
- What is the demand-to-supply ratio?

### Pricing Analytics

- How frequently is surge pricing applied?
- What is the average surge multiplier?
- How much revenue is generated during peak hours?
- How do discounts affect ride bookings?

These requirements determine the fact tables, dimensions, measures, and KPIs.

---

## 4. Identify Business Processes

A ride-sharing platform contains several distinct business processes.

| Business Process | What It Represents |
|---|---|
| Ride Request | Passenger requesting a ride |
| Driver Assignment | Matching drivers to ride requests |
| Trip Execution | Actual ride from pickup to drop-off |
| Ride Status History | Changes in ride status |
| Fare Calculation | Calculating the ride fare |
| Payments | Payment attempts and transactions |
| Driver Availability | Driver availability over time |
| GPS Tracking | Location and movement events |
| Ratings | Passenger and driver feedback |
| Refunds | Money returned after a transaction |

Each process may require a separate fact table because the events and grains are different.

**Important:** A ride request, driver assignment, trip, and payment are not necessarily one-to-one relationships.

One ride may have multiple assignment attempts and payment attempts. Joining all these tables directly can multiply rows and produce incorrect totals.

---

## 5. Define the Grain of Each Fact Table

Grain defines exactly what one row represents.

It must be decided before selecting fact-table columns.

| Fact Table | Grain |
|---|---|
| `fact_ride_request` | One row per ride request |
| `fact_ride` | One row per completed trip or actual trip instance |
| `fact_driver_assignment` | One row per driver assignment attempt |
| `fact_ride_status_event` | One row per ride status event |
| `fact_payment` | One row per payment attempt or transaction |
| `fact_refund` | One row per refund transaction |
| `fact_driver_availability_snapshot` | One row per driver and snapshot time |
| `fact_gps_event` | One row per recorded GPS event |
| `fact_rating` | One row per submitted rating |

### Why Separate Grains Matter

Suppose one ride has:

- Three driver assignment attempts.
- Two payment attempts.
- Ten GPS events.
- Four ride status events.

Joining these tables directly by `ride_id` can create many combinations.

This can result in:
- Duplicate fare amounts.
- Inflated payment totals.
- Incorrect assignment counts.
- Misleading trip metrics.

**Rule:** Keep fact tables at their natural business-event grain. Aggregate them to compatible grains before combining measures.

---

## 6. Design the Dimension Tables

Dimensions store descriptive information used to filter, group, and analyze facts.

### 6.1 dim_passenger

Stores passenger information.

| passenger_key | passenger_id | passenger_name | city | state |
|---:|---|---|---|---|
| 101 | P001 | Rahul | Kolkata | West Bengal |
| 102 | P002 | Priya | Delhi | Delhi |

Possible attributes:
- Passenger name.
- City.
- State.
- Country.
- Registration date.
- Passenger segment.

Sensitive personal details should be restricted or excluded from analytical datasets where not needed.

### 6.2 dim_driver

Stores driver information.

| driver_key | driver_id | driver_name | city | vehicle_type |
|---:|---|---|---|---|
| 201 | D001 | Amit | Kolkata | Sedan |
| 202 | D002 | Ravi | Delhi | SUV |

Possible attributes:
- Driver identifier.
- Vehicle type.
- Vehicle category.
- Operating city.
- Driver status.
- Driver category.

### 6.3 dim_vehicle

Stores vehicle details.

| vehicle_key | vehicle_id | vehicle_type | vehicle_category |
|---:|---|---|---|
| 301 | V001 | Sedan | Standard |
| 302 | V002 | SUV | Premium |

Possible attributes:
- Vehicle type.
- Vehicle category.
- Manufacturer.
- Model.
- Fuel type.

### 6.4 dim_date

Stores calendar attributes.

| date_key | full_date | day | month | quarter | year |
|---:|---|---:|---|---|---:|
| 20261001 | 2026-10-01 | 1 | October | Q4 | 2026 |
| 20261002 | 2026-10-02 | 2 | October | Q4 | 2026 |

Possible attributes:
- Day.
- Week.
- Month.
- Quarter.
- Year.
- Weekend flag.
- Holiday flag.

### 6.5 dim_time

Useful for analyzing booking patterns by time of day.

| time_key | hour | minute | time_period |
|---:|---:|---:|---|
| 0900 | 9 | 0 | Morning |
| 1800 | 18 | 0 | Evening |

Possible attributes:
- Hour.
- Minute.
- Day period.
- Peak-hour indicator.

### 6.6 dim_location

Stores geographic information.

| location_key | area | city | state |
|---:|---|---|---|
| 401 | Salt Lake | Kolkata | West Bengal |
| 402 | Connaught Place | Delhi | Delhi |

A ride can have different location roles:

- Pickup location.
- Drop-off location.
- Driver's location.
- Driver's operating area.

The same location dimension can be reused through role-playing keys such as `pickup_location_key` and `dropoff_location_key`.

### 6.7 dim_ride_type

Stores ride categories.

| ride_type_key | ride_type | category |
|---:|---|---|
| 501 | Economy | Standard |
| 502 | Premium | Premium |
| 503 | Shared | Shared |

### 6.8 dim_payment_method

| payment_method_key | payment_method |
|---:|---|
| 601 | UPI |
| 602 | Credit Card |
| 603 | Cash |
| 604 | Wallet |

### 6.9 dim_ride_status

| ride_status_key | ride_status |
|---:|---|
| 701 | Requested |
| 702 | Accepted |
| 703 | Started |
| 704 | Completed |
| 705 | Cancelled |

### 6.10 dim_promotion

| promotion_key | coupon_code | promotion_type |
|---:|---|---|
| 801 | FIRST50 | First Ride |
| 802 | SAVE20 | Percentage Discount |

Dimensions should contain descriptive attributes. Frequently changing attributes may require SCD handling.

---

## 7. Design the Fact Tables

Fact tables store measurable business events at a defined grain.

### 7.1 fact_ride_request

**Grain:** One row per ride request.

Possible columns:

| Column | Purpose |
|---|---|
| ride_request_id | Request identifier |
| ride_id | Ride identifier |
| passenger_key | Passenger dimension key |
| pickup_location_key | Pickup location |
| dropoff_location_key | Drop-off location |
| request_date_key | Request date |
| request_time_key | Request time |
| ride_type_key | Requested ride category |
| requested_fare | Estimated fare |
| request_status | Current or final request outcome |

This fact supports ride demand and request-level analysis.

### 7.2 fact_ride

**Grain:** One row per completed trip instance.

Possible columns:
- `ride_id`
- `passenger_key`
- `driver_key`
- `vehicle_key`
- `pickup_location_key`
- `dropoff_location_key`
- `ride_type_key`
- `request_date_key`
- `pickup_date_key`
- `dropoff_date_key`
- `trip_distance`
- `trip_duration_minutes`
- `base_fare`
- `surge_amount`
- `discount_amount`
- `final_fare`

This fact supports completed-trip analysis.

If the platform records trip attempts that do not complete, model those in the request or trip-lifecycle process rather than silently excluding them from all reporting.

### 7.3 fact_driver_assignment

**Grain:** One row per driver assignment attempt.

Possible columns:
- `assignment_id`
- `ride_id`
- `driver_key`
- `assignment_date_key`
- `assignment_timestamp`
- `response_timestamp`
- `assignment_status`
- `response_duration_seconds`
- `rejection_reason_key`

This table supports:
- Driver acceptance rate.
- Assignment rejection rate.
- Average response time.
- Number of assignment attempts per ride.

### 7.4 fact_ride_status_event

**Grain:** One row per ride status event.

Possible columns:
- `ride_id`
- `ride_status_key`
- `event_date_key`
- `event_time_key`
- `event_timestamp`
- `event_sequence`

Example:

| ride_id | status | event_timestamp |
|---|---|---|
| R001 | Requested | 09:00 |
| R001 | Accepted | 09:02 |
| R001 | Started | 09:10 |
| R001 | Completed | 09:35 |

This table preserves the ride lifecycle.

### 7.5 fact_payment

**Grain:** One row per payment attempt or transaction.

Possible columns:
- `payment_id`
- `ride_id`
- `passenger_key`
- `payment_method_key`
- `payment_date_key`
- `payment_amount`
- `payment_status`
- `transaction_reference`

One ride may have multiple payment attempts. Failed attempts should not be counted as successful collections.

### 7.6 fact_refund

**Grain:** One row per refund transaction.

Possible columns:
- `refund_id`
- `payment_id`
- `ride_id`
- `refund_date_key`
- `refund_amount`
- `refund_reason_key`
- `refund_status`

A payment can have multiple partial refunds, depending on platform rules.

### 7.7 fact_driver_availability_snapshot

**Grain:** One row per driver and snapshot time.

Possible columns:
- `driver_key`
- `date_key`
- `time_key`
- `availability_status`
- `online_duration_minutes`
- `available_duration_minutes`
- `busy_duration_minutes`
- `location_key`

This fact supports driver utilization and supply analysis.

Availability balances and durations must be interpreted according to the snapshot or interval definition. Do not sum repeated cumulative snapshots as if they were independent durations.

### 7.8 fact_gps_event

**Grain:** One row per recorded GPS event.

Possible columns:
- `gps_event_id`
- `ride_id`
- `driver_key`
- `event_timestamp`
- `latitude`
- `longitude`
- `speed`
- `location_key`, if mapped to a geographic dimension

GPS data can be extremely high volume. Retention, sampling, aggregation, and access policies should be considered.

### 7.9 fact_rating

**Grain:** One row per submitted rating.

Possible columns:
- `rating_id`
- `ride_id`
- `passenger_key`
- `driver_key`
- `rating_date_key`
- `rating_value`
- `rating_subject`

An order may have separate driver and passenger feedback. Store the rating subject to distinguish them.

---

## 8. Ride-Sharing Warehouse Architecture

A practical architecture can use a medallion approach.

```text
                    Source Systems
                          |
       +------------------+------------------+
       |          |       |       |          |
   Passengers  Drivers  Rides  Payments   GPS Events
       |          |       |       |          |
       +----------+-------+-------+----------+
                          |
                          v
                    BRONZE LAYER
                  Raw Ingested Data
                          |
                          v
                    SILVER LAYER
             Cleaned and Standardized Data
                          |
             +------------+-------------+
             |            |             |
         Validation   Deduplication  Standardization
             |            |             |
             +------------+-------------+
                          |
                          v
                     GOLD LAYER
                 Dimensional Data Model
                          |
       +------------------+------------------+
       |          |       |       |          |
       v          v       v       v          v
    Ride       Driver  Payment  Location  Rating
    Facts      Facts   Facts    Analytics  Facts
       |          |       |       |          |
       +----------+-------+-------+----------+
                          |
                          v
                   BI / Analytics
                 Power BI / Tableau
```

### Bronze Layer

Stores raw or minimally processed source data.

Examples:
- Raw ride requests.
- Raw driver records.
- Raw assignment events.
- Raw payment events.
- Raw GPS events.

### Silver Layer

Cleans and standardizes data.

Examples:
- Validate schemas.
- Deduplicate events.
- Standardize timestamps.
- Validate ride and driver identifiers.
- Handle late-arriving events.
- Normalize status values.
- Apply business validation rules.

### Gold Layer

Stores business-ready dimensional models and aggregates.

Examples:
- Ride fact.
- Driver assignment fact.
- Payment fact.
- Driver availability snapshots.
- Passenger and driver dimensions.
- City-level and hourly demand aggregates.

**Important:** Bronze, Silver, and Gold are logical layers. They do not have to be separate physical systems or storage technologies.

---

## 9. Data Ingestion Strategy

Different source systems may require different ingestion methods.

| Source | Possible Ingestion Method |
|---|---|
| Passenger database | Incremental load or CDC |
| Driver database | Incremental load or CDC |
| Ride booking database | Incremental load or CDC |
| Driver assignment events | Event streaming |
| GPS events | Event streaming |
| Payment transactions | Event streaming or CDC |
| Ratings | Incremental load |
| Promotions | Incremental load |

### Full Load

Loads the complete source dataset.

Useful for:
- Initial ingestion.
- Small reference datasets.
- Controlled rebuilds.

### Incremental Load

Loads only new or changed records.

Possible approaches:
- Timestamp-based extraction.
- Increasing ID.
- CDC.
- File-based ingestion.

### Event-Based Ingestion

Ride status, driver assignment, and GPS events may be ingested continuously or in micro-batches.

Example ride lifecycle:

```text
Ride Requested
      |
      v
Driver Search
      |
      v
Driver Assigned
      |
      v
Driver Accepted
      |
      v
Trip Started
      |
      v
Trip Completed
      |
      v
Payment Processed
```

Not every request follows this exact sequence. A ride may be cancelled, rejected, reassigned, or fail before trip completion.

### Important Design Considerations

- Preserve event timestamps.
- Track ingestion timestamps separately.
- Handle duplicate events.
- Handle out-of-order events.
- Process late-arriving data.
- Make target writes retry-safe.
- Reconcile source and target counts and amounts.
- Maintain separate checkpoints for independent streaming queries.

**Important:** A batch extraction watermark and a Structured Streaming event-time watermark are different concepts.

---

## 10. Handling Ride Status History

A ride's current status does not provide the complete lifecycle.

For example, a completed ride may have passed through several events:

- Requested.
- Driver search started.
- Driver assigned.
- Driver accepted.
- Trip started.
- Trip completed.

A status-event fact preserves these transitions.

### Example: Latest Ride Status

```sql
WITH ranked_status AS (
    SELECT
        ride_id,
        status,
        event_timestamp,
        ROW_NUMBER() OVER (
            PARTITION BY ride_id
            ORDER BY event_timestamp DESC, event_sequence DESC
        ) AS rn
    FROM fact_ride_status_event
)
SELECT
    ride_id,
    status,
    event_timestamp
FROM ranked_status
WHERE rn = 1;
```

This assumes `event_sequence` provides a reliable tie-breaker. If the source has a stronger event ordering field, use that instead.

### Why Status History Matters

It helps calculate:
- Driver assignment time.
- Driver response time.
- Passenger waiting time.
- Trip duration.
- Cancellation stage.
- Time between ride lifecycle milestones.

---

## 11. Handling Slowly Changing Dimensions

Passenger, driver, and vehicle attributes may change over time.

For example, a driver may change their operating city or vehicle.

If historical reporting is required, SCD Type 2 can preserve the old and new dimension versions.

### Example: dim_driver

| driver_key | driver_id | city | vehicle_type | effective_from | effective_to | is_current |
|---:|---|---|---|---|---|---|
| 201 | D001 | Kolkata | Sedan | 2026-01-01 | 2026-07-01 | FALSE |
| 305 | D001 | Howrah | SUV | 2026-07-01 | NULL | TRUE |

A historical ride should reference the driver dimension version valid at the time of that ride if historical attribute reporting is required.

### SCD Strategies

- Type 0: Retain the original value.
- Type 1: Overwrite an existing value.
- Type 2: Insert a new version and preserve history.
- Type 3: Store a previous value in an additional column.

Use the appropriate strategy for each attribute based on reporting requirements.

---

## 12. Important Ride-Sharing KPIs

KPI definitions must be agreed upon with business teams. The numerator, denominator, filters, and time period must be explicit.

| KPI | Example Definition |
|---|---|
| Total Ride Requests | Count of distinct eligible ride requests |
| Completed Rides | Count of completed trip instances |
| Ride Cancellation Rate | Cancelled requests / eligible requests × 100 |
| Driver Acceptance Rate | Accepted assignment attempts / eligible assignment attempts × 100 |
| Average Trip Duration | Average duration of completed trips |
| Average Trip Distance | Average distance of completed trips |
| Average Fare | Average qualifying fare per completed trip |
| Platform Revenue | Platform-defined earnings from eligible rides |
| Driver Utilization | Busy time / eligible online time × 100 |
| Payment Success Rate | Successful payment attempts / eligible payment attempts × 100 |
| Average Driver Rating | Average valid driver rating |
| Demand-to-Supply Ratio | Ride requests / available drivers for a defined area and time window |

### Important KPI Considerations

- Define whether a ride request or completed trip is being counted.
- Decide how reassignment attempts affect driver acceptance rate.
- Define cancellation responsibility: passenger, driver, or platform.
- Decide whether cancelled trips are included in revenue calculations.
- Distinguish gross booking value from platform revenue.
- Define driver online time and busy time consistently.
- Use the same geographic area and time window for demand-to-supply calculations.

---

## 13. Example Analytical Queries

The following queries assume the fact and dimension columns described in this design. Date functions and syntax may vary by database.

### 13.1 Total Ride Requests

```sql
SELECT
    COUNT(DISTINCT ride_id) AS total_ride_requests
FROM fact_ride_request;
```

### 13.2 Completed Rides

```sql
SELECT
    COUNT(DISTINCT ride_id) AS completed_rides
FROM fact_ride
WHERE trip_status = 'Completed';
```

This assumes `fact_ride` contains a `trip_status` column and one row per trip instance.

### 13.3 Daily Ride Requests

```sql
SELECT
    d.full_date,
    COUNT(DISTINCT f.ride_id) AS total_requests
FROM fact_ride_request f
JOIN dim_date d
    ON f.request_date_key = d.date_key
GROUP BY
    d.full_date
ORDER BY
    d.full_date;
```

### 13.4 Top 5 Pickup Locations

```sql
SELECT
    l.area,
    l.city,
    COUNT(DISTINCT f.ride_id) AS total_requests
FROM fact_ride_request f
JOIN dim_location l
    ON f.pickup_location_key = l.location_key
GROUP BY
    l.area,
    l.city
ORDER BY
    total_requests DESC
LIMIT 5;
```

### 13.5 Average Trip Duration

```sql
SELECT
    AVG(trip_duration_minutes) AS avg_trip_duration
FROM fact_ride
WHERE trip_status = 'Completed';
```

### 13.6 Ride Cancellation Rate

```sql
SELECT
    COUNT(CASE
        WHEN request_status = 'Cancelled' THEN 1
    END) * 100.0
    / NULLIF(COUNT(*), 0) AS cancellation_rate
FROM fact_ride_request;
```

This assumes one row per ride request and that all rows belong to the eligible population.

### 13.7 Driver Acceptance Rate

```sql
SELECT
    COUNT(CASE
        WHEN assignment_status = 'Accepted' THEN 1
    END) * 100.0
    / NULLIF(COUNT(*), 0) AS driver_acceptance_rate
FROM fact_driver_assignment;
```

This measures accepted assignment attempts divided by all eligible assignment attempts. It is not the same as the percentage of ride requests that eventually receive a driver.

### 13.8 Top 10 Drivers by Completed Trips

```sql
SELECT
    d.driver_name,
    COUNT(DISTINCT f.ride_id) AS completed_trips
FROM fact_ride f
JOIN dim_driver d
    ON f.driver_key = d.driver_key
WHERE f.trip_status = 'Completed'
GROUP BY
    d.driver_name
ORDER BY
    completed_trips DESC
LIMIT 10;
```

### 13.9 Revenue by City

```sql
SELECT
    l.city,
    SUM(f.final_fare) AS total_fare
FROM fact_ride f
JOIN dim_location l
    ON f.pickup_location_key = l.location_key
WHERE f.trip_status = 'Completed'
GROUP BY
    l.city
ORDER BY
    total_fare DESC;
```

This reports completed-trip fare by pickup city. It is not necessarily platform revenue, because platform revenue may be only a commission or service fee.

### 13.10 Average Driver Rating

```sql
SELECT
    d.driver_name,
    AVG(r.rating_value) AS average_rating
FROM fact_rating r
JOIN dim_driver d
    ON r.driver_key = d.driver_key
WHERE r.rating_subject = 'Driver'
GROUP BY
    d.driver_name
ORDER BY
    average_rating DESC;
```

This assumes the rating fact distinguishes the subject of each rating.

---

## 14. Data Quality Checks

Data quality is critical because incorrect ride or event data can produce misleading KPIs.

### Important Checks

| Quality Check | Example |
|---|---|
| Completeness | Every valid ride request has a request timestamp |
| Uniqueness | Ride request IDs are unique |
| Validity | Trip distance is non-negative |
| Referential Integrity | Fact dimension keys resolve correctly |
| Timestamp Validity | Drop-off time is not before pickup time |
| Consistency | Ride status values follow standard definitions |
| Reconciliation | Source and warehouse ride counts match |
| Freshness | New ride events arrive within the expected SLA |

### Example: Check Duplicate Ride Requests

```sql
SELECT
    ride_id,
    COUNT(*) AS record_count
FROM fact_ride_request
GROUP BY ride_id
HAVING COUNT(*) > 1;
```

### Example: Check Invalid Trip Distance

```sql
SELECT *
FROM fact_ride
WHERE trip_distance < 0
   OR trip_distance IS NULL;
```

### Example: Check Invalid Trip Duration

```sql
SELECT *
FROM fact_ride
WHERE trip_duration_minutes < 0;
```

### Example: Check Missing Driver Keys

```sql
SELECT *
FROM fact_ride
WHERE driver_key IS NULL;
```

A missing driver may be valid for a requested ride that never received an assignment. The validation rule should reflect the process stage and business requirements.

---

## 15. Performance Optimization

Ride-sharing platforms can generate large volumes of trip, assignment, and GPS events.

Performance should be considered during warehouse and pipeline design.

### 15.1 Partitioning

Partition large datasets using appropriate, commonly filtered columns.

Examples:
- Request date.
- Trip date.
- Event date.

Avoid high-cardinality partition columns such as ride IDs, driver IDs, or GPS coordinates if they create excessive small partitions.

### 15.2 Columnar Storage

Formats such as Parquet are useful for analytical workloads because they support column-oriented reads and compression.

### 15.3 Predicate Pushdown and Partition Pruning

Apply filters that allow the query engine and storage format to reduce unnecessary reads where supported.

For example, a daily trip report should avoid scanning unrelated periods when the storage layout and query allow pruning.

### 15.4 File Management

Small files can increase metadata and scheduling overhead.

Use suitable file sizing, optimized writes, and compaction features supported by the platform.

### 15.5 Pre-Aggregation

Create hourly or daily aggregates for frequently accessed dashboards.

Examples:
- Hourly ride requests by city.
- Daily completed rides by driver.
- Daily revenue by city.
- Driver availability by area.
- Average trip duration by ride type.

### 15.6 GPS Data Optimization

GPS events can grow much faster than ordinary transactional tables.

Consider:
- Appropriate retention policies.
- Sampling where business requirements permit.
- Aggregated route or distance metrics.
- Separate storage and processing paths for high-volume location events.
- Restricted access to sensitive location data.

### 15.7 Query Optimization

- Select only required columns.
- Filter early where appropriate.
- Avoid unnecessary shuffles.
- Use suitable join strategies.
- Check execution plans and runtime metrics.
- Cache only data that is reused.

Optimization should be measured against actual workloads. There is no universal partition count or file size that works for every dataset.

---

## 16. Security and Governance

A ride-sharing warehouse may contain passenger information, driver details, payment references, and precise location data.

Important practices:

- Restrict access based on business roles.
- Avoid exposing unnecessary personal information.
- Protect precise GPS and location data.
- Mask sensitive fields where appropriate.
- Store credentials securely.
- Maintain audit logs.
- Track data lineage.
- Apply retention and deletion policies.
- Follow applicable privacy and regulatory requirements.

In Databricks environments, Unity Catalog can help manage access, governance, lineage, and data discovery.

Cloud storage permissions and catalog permissions may be separate layers.

---

## 17. End-to-End Project Explanation

This is a useful structure for explaining a ride-sharing warehouse in a Data Engineer interview.

### Project Objective

Build a centralized data warehouse to analyze ride requests, completed trips, drivers, passengers, payments, pricing, and geographic demand.

### Step 1: Source Ingestion

Collect data from:
- Passengers.
- Drivers.
- Ride requests.
- Driver assignments.
- Trip events.
- Payments.
- GPS events.
- Ratings.
- Promotions.

Use full or incremental ingestion according to source capabilities.

### Step 2: Bronze Processing

Store raw source data with ingestion metadata.

### Step 3: Silver Processing

- Validate schemas.
- Deduplicate source events.
- Standardize timestamps.
- Validate ride and driver identifiers.
- Handle late and out-of-order events.
- Normalize status values.
- Apply business validation rules.

### Step 4: Gold Modeling

Build:
- `fact_ride_request`
- `fact_ride`
- `fact_driver_assignment`
- `fact_ride_status_event`
- `fact_payment`
- `fact_refund`
- `fact_driver_availability_snapshot`
- `fact_gps_event`
- `fact_rating`
- Passenger, driver, vehicle, date, time, location, and ride-type dimensions.

### Step 5: Historical Tracking

Use SCD Type 1 or Type 2 according to historical reporting requirements.

### Step 6: KPI Calculation

Calculate:
- Ride requests.
- Completed rides.
- Cancellation rate.
- Driver acceptance rate.
- Average trip duration.
- Average trip distance.
- Platform revenue.
- Driver utilization.
- Demand-to-supply ratio.

### Step 7: Data Validation

Perform:
- Duplicate checks.
- Null checks.
- Referential integrity checks.
- Timestamp validation.
- Source-to-target reconciliation.
- KPI validation.

### Step 8: Reporting

Publish Gold datasets to BI tools for business dashboards.

### Step 9: Monitoring

Monitor:
- Pipeline failures.
- Data freshness.
- Processing duration.
- Data quality.
- Event volume anomalies.
- Reconciliation differences.

---

## 18. Common Interview Questions and Answers

### Q1. How would you design a ride-sharing data warehouse?

I would first gather business requirements and identify processes such as ride requests, driver assignments, trip execution, payments, refunds, GPS tracking, and ratings. Then I would define the grain of each fact table, identify dimensions and measures, and design the dimensional model. I would build ingestion and transformation pipelines, validate the data, and publish business-ready datasets.

### Q2. Why do we need separate fact tables?

Different business processes have different grains. Ride requests, driver assignments, completed trips, and payment attempts represent different events. Separate fact tables help prevent incorrect joins and double counting.

### Q3. What is the grain of a ride fact table?

A common grain is one row per completed trip instance. Ride requests should be modeled separately if the business needs to analyze requests that never become completed trips.

### Q4. How would you calculate driver acceptance rate?

I would define the eligible driver assignment attempts, count accepted assignments, and divide by all eligible attempts. I would distinguish this from the percentage of ride requests that eventually receive an accepted driver.

### Q5. How would you handle multiple drivers assigned to one ride?

I would maintain a separate driver assignment fact with one row per assignment attempt. This preserves assignment history and supports acceptance and rejection analysis.

### Q6. How would you calculate average trip duration?

I would define the start and end events first. For example, trip start to trip completion measures actual trip duration. Request time to drop-off measures the passenger's complete waiting and travel experience. Then I would calculate the average over the eligible trips.

### Q7. How would you handle late-arriving GPS events?

I would preserve event timestamps and ingestion timestamps, process late events safely, and update affected aggregates or derived trip metrics when necessary.

### Q8. How would you track ride status changes?

I would maintain an event-history table with one row per status event. This supports analysis of assignment time, waiting time, trip duration, and cancellation stages.

### Q9. What is the difference between gross booking value and platform revenue?

Gross booking value represents the total value of rides under a defined fare basis. Platform revenue represents the amount earned by the platform, such as commission and service fees, according to its accounting rules. They should not be treated as the same metric.

### Q10. How would you calculate demand-to-supply ratio?

I would define a geographic area and time window, count eligible ride requests, and divide by the number of available drivers under a consistent availability definition. The numerator and denominator must use compatible time and location boundaries.

### Q11. How would you optimize GPS data processing?

I would use an appropriate ingestion path, manage retention, avoid unnecessary columns, partition or cluster according to actual query patterns, and create aggregated route or trip metrics where detailed GPS points are not needed.

### Q12. How would you handle passenger address changes?

If historical address reporting is required, I would use SCD Type 2. Fact records would reference the passenger dimension version valid at the event time.

### Q13. What is the biggest risk when joining ride-sharing fact tables?

The biggest risk is joining facts with different grains. Multiple assignment attempts, payment attempts, and GPS events can multiply rows and inflate measures. I would aggregate each fact to a compatible grain before combining them.

### Q14. How would you handle cancelled rides?

I would retain ride requests and their final outcomes, including cancellation stage and responsible party where available. This allows cancellation analysis without incorrectly counting cancelled requests as completed trips.

### Q15. How would you monitor the pipeline?

I would monitor job execution, processing duration, data freshness, event volume, duplicate rates, late-event volume, source-to-target reconciliation, and business KPI anomalies.

---

## 19. Common Design Mistakes

| Mistake | Correct Approach |
|---|---|
| Combining requests, assignments, trips, and payments into one fact | Model separate business processes |
| Not defining fact-table grain | Define what one row represents |
| Joining all facts directly by ride ID | Check cardinality and aggregate to compatible grains |
| Counting assignment attempts as completed rides | Use the appropriate ride-level fact |
| Ignoring cancelled ride requests | Preserve request outcomes |
| Storing only current ride status | Preserve status-event history when required |
| Treating assignment attempts as unique drivers | Track each assignment separately |
| Counting failed payment attempts as collections | Define payment success rules |
| Treating gross booking value as platform revenue | Define financial measures separately |
| Ignoring late GPS events | Preserve event time and process corrections safely |
| Partitioning GPS data by ride ID | Use a layout suitable for actual query patterns |
| Assuming pipeline success means accurate data | Validate quality and reconciliation |

---

## 20. Quick Revision

### Main Source Systems

- Passengers.
- Drivers.
- Vehicles.
- Ride requests.
- Driver assignments.
- Trips.
- Payments.
- GPS events.
- Ratings.
- Promotions.

### Main Fact Tables

| Fact Table | Grain |
|---|---|
| `fact_ride_request` | One row per ride request |
| `fact_ride` | One row per completed trip instance |
| `fact_driver_assignment` | One row per assignment attempt |
| `fact_ride_status_event` | One row per status event |
| `fact_payment` | One row per payment attempt or transaction |
| `fact_refund` | One row per refund transaction |
| `fact_driver_availability_snapshot` | One row per driver and snapshot time |
| `fact_gps_event` | One row per GPS event |
| `fact_rating` | One row per submitted rating |

### Main Dimensions

- `dim_passenger`
- `dim_driver`
- `dim_vehicle`
- `dim_date`
- `dim_time`
- `dim_location`
- `dim_ride_type`
- `dim_payment_method`
- `dim_ride_status`
- `dim_promotion`

### Architecture

```text
Sources → Bronze → Silver → Gold → BI
```

### Most Important Concepts

1. Gather business requirements before designing tables.
2. Identify distinct business processes.
3. Define the grain of every fact table.
4. Separate ride requests from completed trips.
5. Model driver assignments separately.
6. Preserve ride status history when required.
7. Handle multiple payment attempts.
8. Handle late and out-of-order events.
9. Use conformed dimensions where appropriate.
10. Apply SCD based on historical reporting requirements.
11. Prevent double counting across different fact grains.
12. Optimize high-volume GPS data carefully.
13. Monitor pipeline health and data freshness.

### One-Minute Interview Answer

For a ride-sharing data warehouse, I would first identify source systems and business requirements related to passengers, drivers, ride requests, trips, payments, GPS events, and ratings.

I would define separate grains for each fact table. For example, the ride-request fact would have one row per request, the trip fact one row per completed trip, the driver-assignment fact one row per assignment attempt, and the payment fact one row per payment attempt.

I would create shared dimensions such as passenger, driver, vehicle, date, time, and location. I would preserve ride status history through an event-level fact table and use SCD Type 2 where historical attribute tracking is required.

I would build Bronze, Silver, and Gold processing layers, implement incremental ingestion, handle late and out-of-order events, and apply data quality checks.

Finally, I would publish business-ready datasets for KPIs such as completed rides, cancellation rate, driver acceptance rate, average trip duration, platform revenue, and demand-to-supply ratio. I would carefully manage fact-table grain to prevent double counting.

---

## 21. Self-Assessment

- [ ] Explain the objective of a ride-sharing data warehouse.
- [ ] Identify the source systems.
- [ ] Gather business requirements.
- [ ] Identify distinct business processes.
- [ ] Define the grain of each fact table.
- [ ] Design ride-request and trip facts.
- [ ] Design driver-assignment and status-event facts.
- [ ] Design payment and refund facts.
- [ ] Explain GPS event data.
- [ ] Identify conformed dimensions.
- [ ] Explain star schema design.
- [ ] Explain SCD Type 1 and Type 2.
- [ ] Explain incremental ingestion and CDC.
- [ ] Handle late and out-of-order events.
- [ ] Calculate ride cancellation and driver acceptance rates.
- [ ] Distinguish gross booking value from platform revenue.
- [ ] Identify fact-to-fact join risks.
- [ ] Explain data quality and reconciliation.
- [ ] Explain partitioning and query optimization.
- [ ] Explain security and governance.
- [ ] Present the end-to-end architecture in an interview.

**Interview readiness:** Given a ride-sharing business scenario, be able to identify source systems, business processes, fact-table grains, dimensions, measures, ingestion strategy, event history, KPIs, quality checks, and analytical outputs.