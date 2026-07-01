 Notification System Design

# Stage 1

## Introduction

The Campus Notification System is designed to deliver important announcements to authenticated users in a secure and reliable manner. Notifications may include placement drives, examination schedules, event announcements, academic updates, and other campus-related information. The service follows RESTful API principles and exchanges data in JSON format to ensure seamless integration with frontend applications.

---

# Core Actions

The notification platform should support the following operations:

- Create a notification
- Retrieve all notifications for a user
- Retrieve a specific notification
- Update an existing notification
- Delete a notification
- Mark a notification as read
- Mark all notifications as read
- Retrieve unread notifications

---

# REST API Endpoints

## 1. Create Notification

### Endpoint

```http
POST /api/v1/notifications
```

### Description

Creates a new notification.

### Request Headers

```http
Authorization: Bearer <JWT>
Content-Type: application/json
Accept: application/json
```

### Request Body

```json
{
  "title": "Placement Drive",
  "message": "TCS recruitment drive on Friday at 10:00 AM.",
  "type": "PLACEMENT",
  "priority": "HIGH"
}
```

### Success Response

```json
{
  "id": 101,
  "message": "Notification created successfully."
}
```

### Status Code

```
201 Created
```

---

## 2. Get All Notifications

### Endpoint

```http
GET /api/v1/notifications
```

### Description

Returns all notifications available for the logged-in user.

### Request Headers

```http
Authorization: Bearer <JWT>
Accept: application/json
```

### Success Response

```json
{
  "notifications": [
    {
      "id": 101,
      "title": "Placement Drive",
      "message": "TCS recruitment drive on Friday.",
      "type": "PLACEMENT",
      "priority": "HIGH",
      "isRead": false,
      "createdAt": "2026-07-01T09:30:00Z"
    },
    {
      "id": 102,
      "title": "Exam Schedule",
      "message": "Mid-semester timetable has been released.",
      "type": "ACADEMIC",
      "priority": "MEDIUM",
      "isRead": true,
      "createdAt": "2026-06-29T08:00:00Z"
    }
  ]
}
```

### Status Code

```
200 OK
```

---

## 3. Get Notification By ID

### Endpoint

```http
GET /api/v1/notifications/{id}
```

### Description

Returns complete information for a single notification.

### Request Headers

```http
Authorization: Bearer <JWT>
Accept: application/json
```

### Success Response

```json
{
  "id": 101,
  "title": "Placement Drive",
  "message": "TCS recruitment drive on Friday at 10:00 AM.",
  "type": "PLACEMENT",
  "priority": "HIGH",
  "isRead": false,
  "createdAt": "2026-07-01T09:30:00Z"
}
```

### Status Code

```
200 OK
```

---

## 4. Update Notification

### Endpoint

```http
PUT /api/v1/notifications/{id}
```

### Description

Updates an existing notification.

### Request Body

```json
{
  "title": "Placement Drive Updated",
  "message": "The interview venue has changed to Seminar Hall 2.",
  "priority": "HIGH"
}
```

### Success Response

```json
{
  "message": "Notification updated successfully."
}
```

### Status Code

```
200 OK
```

---

## 5. Delete Notification

### Endpoint

```http
DELETE /api/v1/notifications/{id}
```

### Description

Deletes the specified notification.

### Request Headers

```http
Authorization: Bearer <JWT>
```

### Success Response

```json
{
  "message": "Notification deleted successfully."
}
```

### Status Code

```
204 No Content
```

---

## 6. Mark Notification as Read

### Endpoint

```http
PATCH /api/v1/notifications/{id}/read
```

### Description

Marks a notification as read.

### Success Response

```json
{
  "message": "Notification marked as read."
}
```

### Status Code

```
200 OK
```

---

## 7. Mark All Notifications as Read

### Endpoint

```http
PATCH /api/v1/notifications/read-all
```

### Description

Marks every unread notification of the logged-in user as read.

### Success Response

```json
{
  "message": "All notifications marked as read."
}
```

### Status Code

```
200 OK
```

---

## 8. Get Unread Notifications

### Endpoint

```http
GET /api/v1/notifications/unread
```

### Description

Returns only unread notifications.

### Success Response

```json
{
  "notifications": [
    {
      "id": 105,
      "title": "Hackathon Registration",
      "message": "Registration closes tomorrow.",
      "type": "EVENT",
      "priority": "MEDIUM",
      "isRead": false
    }
  ]
}
```

### Status Code

```
200 OK
```

---

# Notification JSON Schema

```json
{
  "id": 101,
  "title": "Placement Drive",
  "message": "TCS recruitment drive on Friday.",
  "type": "PLACEMENT",
  "priority": "HIGH",
  "recipientId": 55,
  "isRead": false,
  "createdAt": "2026-07-01T09:30:00Z",
  "updatedAt": "2026-07-01T09:45:00Z"
}
```

# Real-Time Notification Mechanism

The application uses **WebSocket** to deliver notifications instantly without requiring the user to refresh the page.

### Workflow

1. User logs into the application.
2. Authentication is completed using JWT.
3. The client establishes a WebSocket connection.
4. An administrator creates a notification.
5. The notification is stored in the database.
6. The notification service publishes the event.
7. The WebSocket server pushes the notification to all eligible connected users.
8. The frontend immediately displays the new notification.


# Conclusion

The proposed REST API provides a consistent interface for managing notifications in the Campus Notification System. The design follows REST principles, uses meaningful endpoint naming, standard HTTP methods, JWT-based authentication, JSON payloads, and WebSocket-based real-time delivery. This design offers a solid foundation for future implementation in Spring Boot while providing frontend developers with a clear API contract.
------------------------------------
---

# Stage 2

## Persistent Storage Selection

For the Campus Notification System, **PostgreSQL** is the recommended database.

### Why PostgreSQL?

PostgreSQL is a relational database that provides reliability, consistency, and excellent support for structured data.

The reasons for choosing PostgreSQL are:

- Supports ACID transactions to ensure data integrity.
- Handles relationships between users and notifications efficiently.
- Excellent indexing support for faster queries.
- Easily integrates with Spring Boot using Spring Data JPA.
- Suitable for handling millions of notification records.
- Open-source and production-ready.

---

# Database Schema

## Table: users

| Column | Data Type | Constraints |
|---------|-----------|-------------|
| id | BIGSERIAL | Primary Key |
| name | VARCHAR(100) | NOT NULL |
| email | VARCHAR(150) | UNIQUE |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |

---

## Table: notifications

| Column | Data Type | Constraints |
|---------|-----------|-------------|
| id | BIGSERIAL | Primary Key |
| title | VARCHAR(255) | NOT NULL |
| message | TEXT | NOT NULL |
| type | VARCHAR(50) | NOT NULL |
| priority | VARCHAR(20) | NOT NULL |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |

---

## Table: user_notifications

This table stores notifications assigned to users.

| Column | Data Type | Constraints |
|---------|-----------|-------------|
| id | BIGSERIAL | Primary Key |
| user_id | BIGINT | Foreign Key |
| notification_id | BIGINT | Foreign Key |
| is_read | BOOLEAN | DEFAULT FALSE |
| read_at | TIMESTAMP | NULL |

---

# Database Relationship

```
Users
   |
   | 1
   |
   |------< User_Notifications >------1
                     |
                     |
               Notifications
```

A user can receive many notifications, and one notification can be sent to multiple users.

---

# Challenges as Data Volume Increases

As the number of users and notifications grows, several challenges may occur.

### 1. Slow Query Performance

Searching through millions of notifications may become slower.

### Solution

Create indexes on frequently searched columns.

Example:

- user_id
- notification_id
- created_at
- is_read

---

### 2. Large Database Size

Old notifications continuously increase storage usage.

### Solution

Archive notifications older than one year into archive tables.

---

### 3. High Concurrent Requests

Thousands of students may access notifications simultaneously.

### Solution

Use Redis Cache to store recently accessed notifications and reduce database load.

---

### 4. High Notification Traffic

Sending notifications individually becomes slow.

### Solution

Use a message queue such as RabbitMQ or Apache Kafka to process notifications asynchronously.

---

### 5. Table Growth

Very large tables increase query execution time.

### Solution

Use table partitioning based on creation date.

---

# SQL Queries

## Create Notification

```sql
INSERT INTO notifications(title,message,type,priority)
VALUES
(
'Placement Drive',
'TCS Recruitment Drive on Friday',
'PLACEMENT',
'HIGH'
);
```

---

## Get All Notifications

```sql
SELECT *
FROM notifications
ORDER BY created_at DESC;
```

---

## Get Notification By ID

```sql
SELECT *
FROM notifications
WHERE id = 101;
```

---

## Update Notification

```sql
UPDATE notifications
SET
title='Updated Placement Drive',
message='Venue changed to Seminar Hall',
priority='HIGH'
WHERE id=101;
```

---

## Delete Notification

```sql
DELETE FROM notifications
WHERE id=101;
```

---

## Mark Notification as Read

```sql
UPDATE user_notifications
SET
is_read = TRUE,
read_at = CURRENT_TIMESTAMP
WHERE
user_id = 1
AND notification_id = 101;
```

---

## Get Unread Notifications

```sql
SELECT n.*
FROM notifications n
JOIN user_notifications un
ON n.id = un.notification_id
WHERE
un.user_id = 1
AND un.is_read = FALSE
ORDER BY n.created_at DESC;
```

---

# Conclusion

PostgreSQL is an ideal choice for the Campus Notification System because it provides reliability, scalability, strong transactional support, and seamless integration with Spring Boot. By implementing proper indexing, caching, database partitioning, and asynchronous processing, the system can efficiently manage large volumes of notification data while maintaining high performance.
------------------
---

# Stage 3

# Query Analysis and Optimization

The current SQL query provided by the previous developer is:

```sql
SELECT FROM notifications *
WHERE studentID = 1042
AND isRead = false
ORDER BY createdAt DESC;
```

---

# Is the Query Correct?

No. The query contains a syntax error.

The `*` should come immediately after the `SELECT` keyword.

### Correct Query

```sql
SELECT *
FROM notifications
WHERE studentID = 1042
AND isRead = FALSE
ORDER BY createdAt DESC;
```

---

# Why is the Query Slow?

When the notification table contains nearly **5 million records**, retrieving data becomes more expensive if the database cannot efficiently locate the required rows.

The main reasons for slower execution are:

### 1. Full Table Scan

Without suitable indexes, the database scans every notification record before applying the filter.

### 2. Sorting Overhead

The query sorts notifications by `createdAt`. Sorting a large result set consumes additional processing time.

### 3. Large Number of Notifications

A single student may have accumulated thousands of notifications over time, increasing the amount of data that must be processed.

### 4. Growing Database Size

As more notifications are stored, disk I/O increases and query execution naturally becomes slower.

---

# Recommended Improvements

Instead of indexing every column, create a composite index that matches the filtering and sorting conditions used by the query.

### Composite Index

```sql
CREATE INDEX idx_student_read_created
ON notifications
(studentID, isRead, createdAt DESC);
```

This index helps the database:

- Locate a student's notifications quickly.
- Filter unread notifications efficiently.
- Return results already ordered by `createdAt`.
- Reduce unnecessary sorting operations.

---

# Expected Computational Cost

### Without Index

```
O(N)
```

The database may examine every notification record.

For approximately **5,000,000** notifications, this results in a full table scan.

---

### With Composite Index

```
O(log N + K)
```

Where:

- **N** = Total number of notifications
- **K** = Number of unread notifications belonging to the requested student

The database searches the index using logarithmic time and retrieves only the matching rows.

---

# Should Every Column be Indexed?

No.

Adding indexes to every column is generally not a good database design strategy.

### Reasons

- Every INSERT operation must also update every index.
- UPDATE statements become slower.
- DELETE operations require additional index maintenance.
- More disk space is consumed.
- Many indexes remain unused and provide no benefit.

Indexes should only be created on columns that are frequently used in:

- WHERE clauses
- JOIN conditions
- ORDER BY clauses
- GROUP BY clauses

Choosing indexes based on query patterns provides better performance than indexing every column.

---

# SQL Query

## Students Who Received Placement Notifications During the Last 7 Days

```sql
SELECT DISTINCT studentID
FROM notifications
WHERE notificationType = 'Placement'
AND createdAt >= CURRENT_DATE - INTERVAL '7 days';
```

---

# Additional Optimization Suggestions

To keep query performance consistent as the database grows, the following improvements can also be adopted:

- Partition the notifications table using the creation date.
- Archive old notifications that are no longer accessed frequently.
- Use pagination when returning notification lists.
- Cache frequently requested notification data using Redis.
- Analyze slow queries periodically using PostgreSQL's `EXPLAIN ANALYZE` command.

---

# Conclusion

The original query contains a syntax error and is inefficient for a table containing millions of records. A carefully designed composite index significantly improves performance by reducing full table scans and unnecessary sorting. Rather than indexing every column, indexes should be created based on actual query patterns to balance read performance with write efficiency.