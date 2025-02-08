.. _architecture:

Architecture
------------

.. image:: images/spring_rest_framework.jpg
   :alt: Spring Rest Framework Architecture

The framework is built on top of **Spring Boot, Spring MVC, Spring Data JPA, and Spring Security**, providing a robust suite of tools for developing **REST APIs**. The core components are as follows:

Controller Layer
------------------------
This layer consists of **generic controllers** that provide common implementation for different types of requests:

- **GenericListController**
- **GenericCreateController**
- **GenericDeleteController**
- **GenericRetrieveController**
- **GenericUpdateController**
- **GenericCommandController**
- **GenericQueryController**

These controllers help in structuring API endpoints for **CRUD operations** in a reusable and modular way.

Service Layer
--------------------------
This layer contains two service classes operating as **business logic**:

- **CommandService** – Handles commands related to modifications (Create, Update, Delete).
- **QueryService** – Handles data retrieval operations.

Utility Layer
-------------------------
This layer provides tools used in the service layer:

- **DtoManager** – Manages Data Transfer Objects.
- **SwaggerSchemaGenerator** – Generates API documentation using Swagger.
- **EntityBuilder** – Helps construct entity objects.
- **SpecificationsBuilder** – Builds query specifications dynamically.

FilterSet and Serializer:

- **FilterSet** – Defines filtering rules for querying data.
- **Serializer** – Handles object serialization and deserialization.


The entire structure is built upon **Spring Framework**, leveraging its powerful features for dependency injection, security, and configuration.

