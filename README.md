Aiven Demo
-----------

<p align="center">
    <img src="images/pipeline.png" width="1200" height="500">
</p>


### Topics
- **ecommerce.users:** Compacted Topic storing the latest user state
- **ecommerce.products:** Compacted Topic storing the latest product state
- **ecommerce.events:** Stores the raw events
- **ecommerce.events.filtered:** Stores only purchase events
- **ecommerce.events.users:** Stores events enriched with user information
- **ecommerce.events.enriched:** Stores the final enriched events with user and product information