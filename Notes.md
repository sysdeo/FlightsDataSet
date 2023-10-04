`Total delayed flights by day of week`

INSERT INTO nflights (day_of_week, nday)
SELECT day_of_week, COUNT(*)
FROM flights
WHERE arrival_delay > 0
GROUP BY day_of_week
ORDER BY day_of_week;


---

`Created table delay type to count delays by:`

`Department, Flight Number, Origin, Destination, Time of day`


`Note: this is the table of which the majority of charts come from`

CREATE TABLE delay_type(
arrival_delay NUMERIC,
air_system_delay NUMERIC,
security_delay NUMERIC,
airline_delay NUMERIC,
late_aircraft NUMERIC,
weather_delay NUMERIC,
airline TEXT,
flight_number NUMERIC(4),
origin_airport TEXT,
destination_airport TEXT);

----

`Inserting into 1 of the main tables specific columns to further process delays over 0 minutes`

INSERT INTO delay_type(arrival_delay, air_system_delay, security_delay, airline_delay, late_aircraft, weather_delay, airline, flight_number, origin_airport, destination_airport)
SELECT arrival_delay, air_system_delay, security_delay, airline_delay, late_aircraft_delay, weather_delay, airline, flight_number, origin_airport, destination_airport
FROM flights
WHERE arrival_delay > 0;


---

`Creating a table showing delays with over 0 for 6 sections of the day including:`

`Midnight, Dawn, Morning, Noon, Afternoon, Dusk`


CREATE TABLE six_way_delays (
    id SERIAL PRIMARY KEY,
    airline TEXT,
    flight_number TEXT,
    origin_airport TEXT,
    destination_airport TEXT,
    delay_minutes INT,
    time_of_day TEXT
    delay_count_at_destination INTEGER;
);

---

`Populating six_way_delays table with segmented data`

INSERT INTO six_way_delays (airline, flight_number, origin_airport, destination_airport, delay_minutes, time_of_day)
SELECT
    airline,
    flight_number,
    origin_airport,
    destination_airport,
    arrival_delay,
    CASE
        WHEN EXTRACT(HOUR FROM TIME '00:00' + ((scheduled_departure / 100) * INTERVAL '1 hour') + ((scheduled_departure % 100) * INTERVAL '1 minute')) BETWEEN 0 AND 3 THEN 'Midnight to Dawn'
        WHEN EXTRACT(HOUR FROM TIME '00:00' + ((scheduled_departure / 100) * INTERVAL '1 hour') + ((scheduled_departure % 100) * INTERVAL '1 minute')) BETWEEN 4 AND 7 THEN 'Dawn to Morning'
        WHEN EXTRACT(HOUR FROM TIME '00:00' + ((scheduled_departure / 100) * INTERVAL '1 hour') + ((scheduled_departure % 100) * INTERVAL '1 minute')) BETWEEN 8 AND 11 THEN 'Morning to Noon'
        WHEN EXTRACT(HOUR FROM TIME '00:00' + ((scheduled_departure / 100) * INTERVAL '1 hour') + ((scheduled_departure % 100) * INTERVAL '1 minute')) BETWEEN 12 AND 15 THEN 'Noon to Afternoon'
        WHEN EXTRACT(HOUR FROM TIME '00:00' + ((scheduled_departure / 100) * INTERVAL '1 hour') + ((scheduled_departure % 100) * INTERVAL '1 minute')) BETWEEN 16 AND 19 THEN 'Afternoon to Dusk'
        WHEN EXTRACT(HOUR FROM TIME '00:00' + ((scheduled_departure / 100) * INTERVAL '1 hour') + ((scheduled_departure % 100) * INTERVAL '1 minute')) BETWEEN 20 AND 23 THEN 'Dusk to Midnight'
    END
FROM
    flights
WHERE
    arrival_delay > 0;


----

`Populating the table with segmented data`

INSERT INTO six_way_delays (airline, flight_number, origin_airport, destination_airport, delay_minutes, time_of_day)

SELECT

airline,

flight_number,

origin_airport,

destination_airport,

arrival_delay,

CASE

WHEN EXTRACT(HOUR FROM TIME '00:00' + ((scheduled_departure / 100) * interval '1 hour') + ((scheduled_departure % 100) * interval '1 minute')) BETWEEN 0 AND 3 THEN 'Midnight to Dawn'

WHEN EXTRACT(HOUR FROM TIME '00:00' + ((scheduled_departure / 100) * interval '1 hour') + ((scheduled_departure % 100) * interval '1 minute')) BETWEEN 4 AND 7 THEN 'Dawn to Morning'

WHEN EXTRACT(HOUR FROM TIME '00:00' + ((scheduled_departure / 100) * interval '1 hour') + ((scheduled_departure % 100) * interval '1 minute')) BETWEEN 8 AND 11 THEN 'Morning to Noon'

WHEN EXTRACT(HOUR FROM TIME '00:00' + ((scheduled_departure / 100) * interval '1 hour') + ((scheduled_departure % 100) * interval '1 minute')) BETWEEN 12 AND 15 THEN 'Noon to Afternoon'

WHEN EXTRACT(HOUR FROM TIME '00:00' + ((scheduled_departure / 100) * interval '1 hour') + ((scheduled_departure % 100) * interval '1 minute')) BETWEEN 16 AND 19 THEN 'Afternoon to Dusk'

WHEN EXTRACT(HOUR FROM TIME '00:00' + ((scheduled_departure / 100) * interval '1 hour') + ((scheduled_departure % 100) * interval '1 minute')) BETWEEN 20 AND 23 THEN 'Dusk to Midnight'

END

FROM

flights

WHERE

arrival_delay > 0;`

----

`Counting unique delays by Airport`

UPDATE six_way_delays
SET delay_count_at_destination = subquery.delay_count
FROM (
    SELECT destination_airport, COUNT(*) as delay_count
    FROM six_way_delays
    GROUP BY destination_airport
) AS subquery
WHERE six_way_delays.destination_airport = subquery.destination_airport;

---

`rounding up numbers to limit to a singular decimal (eg: 26.1, 36.2)`

UPDATE seasonal_delays
SET spring = ROUND(CAST(spring AS NUMERIC), 1);

----
