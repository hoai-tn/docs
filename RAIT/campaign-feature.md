# Dynamic Discount System Implementation (Phase 1)

## 1. Overview of Existing Database Structure

The current database schema includes the following key tables relevant to implementing the discount system:

- **clients**: Contains client information.
- **services**: Lists the salon's services.
- **records**: Represents appointments or bookings.
- **companies** and **branches**: Represent the salon's organizational hierarchy.
- **record_services**: Links records to services, including pricing details.

---

## 2. Proposed New Tables and Modifications

### A. New Tables

#### 1. `promotions`

This table stores all promotion campaigns.

```sql
CREATE TABLE promotions (
  id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
  company_id BIGINT UNSIGNED NOT NULL,
  name VARCHAR(255) NOT NULL,
  description TEXT,
  discount_type ENUM('percentage', 'fixed', 'buy_one_get_one', 'loyalty') NOT NULL,
  discount_value DECIMAL(10,2) DEFAULT NULL,
  start_date DATETIME NOT NULL,
  end_date DATETIME NOT NULL,
  active TINYINT(1) NOT NULL DEFAULT '1',
  created_at TIMESTAMP NULL DEFAULT NULL,
  updated_at TIMESTAMP NULL DEFAULT NULL,
  PRIMARY KEY (id),
  FOREIGN KEY (company_id) REFERENCES companies(id) ON DELETE CASCADE
);
```

#### 2. `promotion_services`

Associates promotions with specific services.

```sql
CREATE TABLE promotion_services (
  id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
  promotion_id BIGINT UNSIGNED NOT NULL,
  service_id BIGINT UNSIGNED NOT NULL,
  PRIMARY KEY (id),
  FOREIGN KEY (promotion_id) REFERENCES promotions(id) ON DELETE CASCADE,
  FOREIGN KEY (service_id) REFERENCES services(id) ON DELETE CASCADE
);
```

#### 3. `promotion_times`

Defines when a promotion is active.

```sql
CREATE TABLE promotion_times (
  id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
  promotion_id BIGINT UNSIGNED NOT NULL,
  day_of_week TINYINT UNSIGNED, -- 1 (Monday) to 7 (Sunday), NULL if any day
  start_time TIME,              -- NULL if no specific start time
  end_time TIME,                -- NULL if no specific end time
  PRIMARY KEY (id),
  FOREIGN KEY (promotion_id) REFERENCES promotions(id) ON DELETE CASCADE
);
```

#### 4. `client_promotions`

Tracks client-specific promotions, such as new client discounts or loyalty programs.

```sql
CREATE TABLE client_promotions (
  id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
  promotion_id BIGINT UNSIGNED NOT NULL,
  client_id BIGINT UNSIGNED NOT NULL,
  usage_count INT UNSIGNED DEFAULT 0,
  max_usage INT UNSIGNED DEFAULT 1,
  created_at TIMESTAMP NULL DEFAULT NULL,
  updated_at TIMESTAMP NULL DEFAULT NULL,
  PRIMARY KEY (id),
  FOREIGN KEY (promotion_id) REFERENCES promotions(id) ON DELETE CASCADE,
  FOREIGN KEY (client_id) REFERENCES clients(id) ON DELETE CASCADE
);
```

### B. Modifications to Existing Tables

#### 1. `records`

Add a field to associate a booking with a promotion.

```sql
ALTER TABLE records
ADD COLUMN promotion_id BIGINT UNSIGNED DEFAULT NULL,
ADD FOREIGN KEY (promotion_id) REFERENCES promotions(id);
```

---

## 3. Integration with Existing System

### Booking Process

- **Promotion Application**: When a booking is made, the system checks for applicable promotions based on the client, service, date, and time.
- **Recording Promotion**: If a promotion applies, the `promotion_id` is stored in the `records` table.
- **Price Calculation**: Discounts are applied to services in `record_services` based on the promotion details.

### Client Management

- **New Clients**: Automatically associate new clients with relevant promotions in `client_promotions`.
- **Loyalty Programs**: Update `usage_count` in `client_promotions` after each booking to track loyalty rewards.

### Service Management

- **Promotion Services**: Use `promotion_services` to specify which services are eligible for each promotion.

### Time-Specific Promotions

- **Promotion Times**: Define active days and times for promotions in `promotion_times`.

---

## 4. Detailed Explanations

### A. `promotions` Table

- **Purpose**: Central repository for all promotion campaigns.
- **Key Fields**:
  - `discount_type`: Determines the type of discount:
    - `'percentage'`: A percentage off the service price.
    - `'fixed'`: A fixed amount off the service price.
    - `'buy_one_get_one'`: Special offer where purchasing one service entitles the client to another for free.
    - `'loyalty'`: Used for loyalty program discounts.
  - `discount_value`: The value associated with the discount.
  - `start_date` and `end_date`: Define the promotion's validity period.
  - `active`: Indicates if the promotion is currently active.

### B. `promotion_services` Table

- **Purpose**: Links promotions to the services they apply to.
- **Usage**: Specifies eligible services for each promotion.

### C. `promotion_times` Table

- **Purpose**: Specifies when (days and times) a promotion is applicable.
- **Fields**:
  - `day_of_week`: Numeric representation (1 for Monday through 7 for Sunday). `NULL` if the promotion is valid any day.
  - `start_time` and `end_time`: Time window during which the promotion is active. `NULL` if the promotion is valid all day.

### D. `client_promotions` Table

- **Purpose**: Manages promotions specific to individual clients.
- **Usage Scenarios**:
  - **New Client Discount**: Assign a promotion when a new client registers.
  - **Loyalty Program**: Track the number of times a client has used a promotion.
- **Fields**:
  - `usage_count`: How many times the client has used the promotion.
  - `max_usage`: Maximum number of times the promotion can be used by the client.

### E. Modification to `records` Table

- **Purpose**: Associates a booking with a promotion for tracking and reporting.
- **Implementation**: The `promotion_id` field links the booking to the applied promotion.

---

## Examples of Implementing Promotional Campaigns

### 1. Discount for New Clients

- **Setup**:
  - Add a promotion with `discount_type = 'percentage'` and the desired `discount_value`.
  - Set `start_date` and `end_date`.
- **Client Association**:
  - Upon client registration, create an entry in `client_promotions` linking the client to the promotion.

### 2. Time-Specific Discounts

- **Setup**:
  - Create a promotion.
  - Define applicable times in `promotion_times` (e.g., `day_of_week = 1` for Monday, `start_time = '14:00:00'`, `end_time = '16:00:00'`).
  - Link relevant services in `promotion_services`.

### 3. Buy One, Get One Free

- **Setup**:
  - Create a promotion with `discount_type = 'buy_one_get_one'`.
  - Use `promotion_services` to specify the qualifying services.

### 4. Seasonal or Holiday Discounts

- **Setup**:
  - Add a promotion with the holiday period in `start_date` and `end_date`.
  - Set `discount_type` and `discount_value` accordingly.
  - Mark the promotion as active only during the holiday season.

### 5. Loyalty Program Discounts

- **Setup**:
  - Create a promotion with `discount_type = 'loyalty'`.
  - Set `max_usage` in `client_promotions` to the number of visits after which the discount applies.
- **Tracking**:
  - Increment `usage_count` in `client_promotions` after each client visit.
  - Apply the discount when `usage_count` reaches the threshold.

---

## Additional Considerations

### Promotion Prioritization

- Define business logic to handle cases where multiple promotions might apply.
- Decide whether promotions can be combined or if only the best available discount should be applied.

### Exclusions and Limitations

- Consider adding additional fields or tables if you need to specify exclusions (e.g., services or clients that are not eligible).

### User Interface

- Ensure that your administration panel allows staff to:
  - Create, edit, and deactivate promotions.
  - Associate promotions with services and times.
  - Monitor promotion usage and effectiveness.

### Performance Optimization

- Index key fields such as `promotion_id`, `client_id`, and `day_of_week` to optimize query performance.
- Regularly monitor database performance, especially if promotions are frequently accessed.

### Data Integrity

- Use transactions where appropriate to maintain data consistency during complex operations.
- Implement proper error handling in your application code to manage exceptions.

---

## Conclusion

By adding these tables and modifications, the existing database will support the dynamic discount system. This design allows for flexible creation and management of various promotional campaigns, aligning with business objectives to attract new clients, fill slow hours, promote specific services, and reward loyal customers.
