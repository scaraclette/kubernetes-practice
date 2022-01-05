# Ambassador Pattern
Pattern where an ambassador container brokers interactions between the application container and the rest of the world.

Benefits for ambassador pattern:
1. Building modular, reusable containers.
2. Separation of concerns makes the containers easier to build and maintain.
3. Ability to re-use container speeds up application development.

## Examples
### Using an Ambassador to Shard a Service
Sharding your storage layer.