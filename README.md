# Assignment 03 - SQL Query Writing (Football Ticket Booking System)

Hey, this is my assignment-03 submission. This one is about writing SQL queries on a Football Ticket Booking System database. The table structure (Users, Matches, Bookings) and the sample data were given to us as a template, but I filled in the data types, constraints, and wrote all the queries myself.

## What's in this database

Three tables, and they're connected to each other:

- **Users** - the people using the app (fans and ticket managers)
- **Matches** - the football matches you can book tickets for
- **Bookings** - the actual ticket bookings, connects Users and Matches together

Bookings has `user_id` and `match_id` as foreign keys, so that's how everything links up.

## The Queries

### 1. Champions League matches that are still available

```sql
SELECT match_id, fixture, base_ticket_price 
FROM Matches 
WHERE tournament_category = 'Champions League' AND match_status = 'Available';
```

Just a simple WHERE with two conditions. Nothing fancy, just filtering by category and status at the same time.

### 2. Search users by name (case-insensitive)

```sql
SELECT user_id, full_name, email 
FROM Users 
WHERE full_name ILIKE 'Tanvir%' OR full_name ILIKE '%Haque%';
```

I used `ILIKE` here instead of `LIKE` because the question said case-insensitive, and ILIKE ignores case in Postgres. The `%` at the end of `'Tanvir%'` means "starts with Tanvir", and wrapping `Haque` with `%` on both sides means "contains Haque anywhere in the name".

### 3. Bookings with missing payment status

```sql
SELECT booking_id, user_id, match_id, COALESCE(payment_status, 'Action Required') AS systematic_status 
FROM Bookings 
WHERE payment_status IS NULL;
```

Learned about `COALESCE` for this one — it basically says "if this value is NULL, show me this other thing instead." So instead of just showing NULL, it shows 'Action Required'. Still had to filter with `WHERE payment_status IS NULL` too, since the question wanted only the missing ones.

### 4. Booking details with user name and match fixture

```sql
SELECT booking_id, full_name, fixture, total_cost 
FROM Bookings 
INNER JOIN Users USING(user_id) 
INNER JOIN Matches USING(match_id);
```

First time joining three tables together in one query. Used `INNER JOIN` twice since I needed data from Users and Matches, both connected through Bookings. `USING(user_id)` is a shortcut instead of writing the full `ON Bookings.user_id = Users.user_id` thing, works because the column name is the same in both tables.

### 5. All users with their bookings (even fans with zero bookings)

```sql
SELECT user_id, full_name, booking_id 
FROM Users 
LEFT JOIN Bookings USING(user_id);
```

This one tripped me up at first — I originally tried `FULL JOIN` but that was wrong. The question wants ALL users to show up, even if they never booked a ticket, so I needed `LEFT JOIN` starting from Users. FULL JOIN would've also dragged in extra stuff from Bookings that didn't matter here. Good lesson on picking the right join direction depending on which table you actually want "everything" from.

### 6. Bookings above the average total cost

```sql
SELECT booking_id, match_id, total_cost 
FROM Bookings 
WHERE total_cost > (SELECT AVG(total_cost) FROM Bookings);
```

I messed this one up first too. My first attempt used `GROUP BY booking_id HAVING total_cost > AVG(total_cost)` which doesn't make sense because each booking_id is basically its own group of one row, so "average" inside that group is meaningless. What I actually needed was ONE average calculated across the whole table, and then compare every row to that single number. That's what the subquery `(SELECT AVG(total_cost) FROM Bookings)` does — it runs first, gets one number, and then the outer query just filters normally with WHERE. No GROUP BY needed at all here.

### 7. Top 2 matches by price, skipping the most expensive one

```sql
SELECT match_id, fixture, base_ticket_price 
FROM Matches 
ORDER BY base_ticket_price DESC LIMIT 2 OFFSET 1;
```

`ORDER BY ... DESC` sorts highest price first. `OFFSET 1` skips the first row (the most expensive match), and `LIMIT 2` grabs the next two after that. Kind of like pagination, skip-then-take.

## Things I learned doing this assignment

- `ILIKE` vs `LIKE` for case-insensitive search
- `COALESCE` for handling NULL values nicely
- Joining more than 2 tables at once
- The difference between `LEFT JOIN` and `FULL JOIN` and why direction matters
- Why `GROUP BY` + `HAVING` isn't the same as using a subquery to compare against one overall average
- `LIMIT` + `OFFSET` together for skipping rows

That's it for this assignment. Still learning, so if something looks a bit rough that's probably why 😅
