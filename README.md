Aiven Demo
-----------

<p align="center">
    <img src="images/pipeline.png" width="1000" height="500">
</p>


### Topics
- **ecommerce.users:** Compacted Topic storing the latest user state
- **ecommerce.products:** Compacted Topic storing the latest product state
- **ecommerce.events:** Stores the raw events
- **ecommerce.events.filtered:** Stores only purchase events
- **ecommerce.events.users:** Stores events enriched with user information
- **ecommerce.events.enriched:** Stores the final enriched events with user and product information

Table Schemas
-----------------
**Events Table (events)** and **Events Filtered Table (events_filtered)**
```sql
eventTime BIGINT,
eventTime_ltz AS TO_TIMESTAMP_LTZ(eventTime, 3),
eventType STRING,
productId STRING,
categoryId STRING,
categoryCode STRING,
brand STRING,
price DOUBLE,
userid STRING,
userSession STRING,
    WATERMARK FOR eventTime_ltz AS eventTime_ltz - INTERVAL '5' SECONDS
```

**Users Table (users)** 
```sql
userId      STRING,
firstname   STRING,
lastname    STRING,
username    STRING,
email       STRING,
title       STRING,
address     STRING
```

**Product Table(products)**
```sql
productCode STRING,
productColor STRING,
promoCode STRING,
productName STRING
```

**Events with User Information Table (eventusers)**
```sql
eventTime   BIGINT,
productId   STRING,
price       DOUBLE,
userSession STRING,
firstname   STRING,
lastname    STRING,
email       STRING,
address     STRING
```

**Enriched Events Table (enriched_events)**
```sql
userSession STRING,
firstname   STRING,
lastname    STRING,
email       STRING,
address     STRING,
price       DOUBLE,
productName MULTISET<STRING>,
PRIMARY KEY (userSession) NOT ENFORCED
```

Job Queries
-----------
**EventFilter**
```sql
INSERT INTO events_filtered
SELECT eventTime, eventType, productId, categoryId, categoryCode, brand, price, userid, userSession
FROM events
WHERE eventType='purchase'
```

**EventUserInfo**
```sql
INSERT INTO eventusers
SELECT
    events_filtered.eventTime,
    events_filtered.productId,
    events_filtered.price,
    events_filtered.userSession,
    users.firstname,
    users.lastname,
    users.email,
    users.address
FROM events_filtered
         INNER JOIN users ON events_filtered.userid = users.userId
```


**EventEnrichment**
```sql
INSERT INTO events_enriched
SELECT eventusers.userSession,
       eventusers.firstname,
       eventusers.lastname,
       eventusers.email,
       eventusers.address,
       SUM(eventusers.price),
       COLLECT(products.productName)
FROM eventusers
         INNER JOIN products ON products.productCode = eventusers.productId
GROUP BY eventusers.userSession, eventusers.firstname, eventusers.lastname, eventusers.email, eventusers.address, products.productName
```