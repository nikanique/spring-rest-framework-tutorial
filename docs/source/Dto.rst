
DTO
=====

DTO (Data Transfer Object) classes are crucial components in modern applications for efficiently transferring data between layers or across services. The provided `Dto` class serves as a powerful base class, offering a wide range of features to enhance data validation, serialization and deserialization.

Introduction
------------

The `Dto` class supports for:

- **Field exposure**: Controls how model's fields are serialized and deserialized.
- **Field validation**: Enforces constraints on input data.
- **Custom serialization logic**: Allows fine-grained control over JSON output.
- **Field mapping**: Maps DTO fields to model fields, including nested relationships.
- **Read-only and write-only fields**: Restricts field accessibility as needed, only for output, or input.


Read-Only and Write-Only Fields
-------------------------------

- **Read-only fields**: Included in the serialized output but cannot be set during deserialization.
- **Write-only fields**: Can be set during deserialization but are not included in the serialized output.

**Implementation**::

    @ReadOnly
    private Long id;

    @WriteOnly
    private Long countryId;


In the `CarDTO` class:
- `id` is a read-only field, ensuring the client cannot modify it.
- `countryId` is a write-only field, allowing seamless association with a country by its ID during object creation.

Field Exposure and Custom Mapping
---------------------------------

**Description**  
The `@Expose` annotation has following parameters:

- **Source**: Specify the origin of the field value, including nested fields.
- **format**: Format the output value (e.g., decimal places, number formatting).
- **methodName**: Apply a transformation to the field value via a static method.
- **isRequired**: Validate the field to be present.
- **defaultValue**: Set a default value for a field if it is not present.

**Implementation**::

    @Expose(source = "year", methodName = "getTheDecade")
    @ReadOnly
    private Integer decade;

    @Expose(source = "country__continent")
    @ReadOnly
    private String continent;


- `decade` is derived from the `year` field using the static method `getTheDecade`.
- `continent` maps to the `country__continent` field, a nested relationship in the model.

Nested Model Field Mapping
--------------------------

**Description**  
DTOs support filtering and serialization of nested model fields using a double underscore (``__``) notation in the `@Expose` annotation's `source` parameter. This allows seamless representation of related model data.

**Implementation**::

    @Expose(source = "country__continental__population", format = "#,###")
    @ReadOnly
    private Integer population;


In `CarDTO`, `population` maps to a nested model field (`country.continental.population`) and formats the value with thousands separators.

Field Validation
----------------

**Description**  
The `@FieldValidation` annotation and the `validate` method enforce constraints on input fields to ensure data integrity.

**Features**  
- Minimum/maximum length constraints.
- Nullability checks.
- Custom validation logic in the overridden `validate` method.

**Implementation**::

    @FieldValidation(minLength = 3, nullable = false)
    private String company;


Custom validation using `validate` method:

.. code-block:: java

     @Data
    public class OrderDTO extends Dto {

        @ReadOnly
        private Long id;   
        private String company;

        @Override
        public Map<String, String> validate(Set<String> fieldNames, Boolean raiseValidationError) throws Throwable {
            Map<String, String> validationErrors = super.validate(fieldNames, false);

            if (company != null && company.equals("BMW")) {
                validationErrors.put("company", "company must not be BMW");
            }

            if (!validationErrors.isEmpty() && raiseValidationError) {
                throw new ValidationException(validationErrors);
            }

            return validationErrors;
        }
    }


In `ApplianceDTO`:
- `company` must have at least three characters and cannot be null.
- Custom validation in the `validate` method further restricts the value of `company` (e.g., cannot be "BMW").

Custom Serialization Logic
--------------------------

**Description**  
The `toRepresent` method allows developers to override default serialization behavior and define how the DTO should be represented in JSON.

**Implementation**::

    public static ObjectNode toRepresent(ObjectNode node) {
        ObjectMapper mapper = new ObjectMapper();
        ObjectNode wrapperNode = mapper.createObjectNode();
        wrapperNode.set("data", node);
        wrapperNode.put("description", "hello");
        return wrapperNode;
    }


In `ApplianceDTO`, `toRepresent` wraps the serialized object in a custom JSON structure with a `description` field.

Static Method Transformations
-----------------------------

**Description**  
The `@Expose` annotation's `methodName` parameter enables field value transformations using static methods.

**Implementation**::

    @Expose(source = "year", methodName = "getTheDecade")
    @ReadOnly
    private Integer decade;

    public static Integer getTheDecade(Object value) {
        return (Integer) value / 10;
    }

The `getTheDecade` method processes the `year` field, converting it into a decade value.

Formatting Exposed Fields
-------------------------

**Description**  
Use the `format` parameter in the `@Expose` annotation to specify custom output formats for fields.

**Implementation**::

    @Expose(format = "{.2f}")
    private Float mark;

    @Expose(format = "#,###")
    private Integer population;


- `mark` is formatted to two decimal places.
- `population` uses a thousands separator.



postDeserialization() Method
----------------------------

The `postDeserialization()` method allows you to customize and refine DTOs after they are created. This is particularly useful for performing additional processing, such as populating read-only fields, or enriching your DTOs with computed values. It's an essential tool for handling advanced scenarios while maintaining the simplicity and modularity of your code.

For example, consider the following scenario where a `SchoolDto` needs to populate its `address` field based on latitude and longitude values after the DTO is deserialized:

.. code-block:: java

    @Data
    public class SchoolDto extends Dto {

        private String name;
        @ReadOnly
        private String address;  // Read-only field to be set after deserialization
        @ReadOnly
        private Long id;

        private Float latitude;
        private Float longitude;

        @Override
        public void postDeserialization(EntityManager entityManager) {
            // Custom logic to compute the address using latitude and longitude
            // In a real scenario, this could involve a call to an external API
            this.address = retrieveAddressFromGeoCoordinate(latitude, longitude);
        }

    }

In this example:
- The `address` field is marked as `@ReadOnly` and is not set directly during deserialization.
- The `postDeserialization()` method is overridden to compute and assign the value of `address` using a utility method, `retrieveAddressFromGeoCoordinate()`.



Below is an example DTO class that demonstrates all the features of the `Dto` :

.. code-block:: java

    @Data
    public class OrderDTO extends Dto {

        @ReadOnly
        private Long id;

        @Expose(source = "customer__full_name")  // retrieve from order.customer.full_name
        @ReadOnly
        private String customerName;

        @Expose(source = "customer__city__name") // retrieve from order.customer.city.name
        @ReadOnly
        private String customerCity;

        @FieldValidation(nullable = false, minLength = 3)
        @Expose(source = "product__name") // retrieve from order.product.name
        private String productName;

        @Expose(format = "#,###")
        private Integer quantity;

        @Expose(format = "{.2f}")
        private Double pricePerUnit;

        @Expose(source = "total_price", format = "{.2f}")
        @ReadOnly
        private Double totalPrice;

        // Enables associating the order with a customer using the customer's ID
        @ReferencedModel(
                model = "com.example.app.model.Customer",
                referencingField = "customer"
        )
        @WriteOnly
        private Long customerId;

        @ReadOnly
        private Timestamp orderTimestamp;

        @FieldValidation(nullable = false)
        private String orderStatus;

        @Expose(source = "order_details", methodName = "formatOrderDetails") // Calculated field
        @ReadOnly
        private String orderSummary;

        public static String formatOrderDetails(Object value) {
            return "Summary: " + value.toString();
        }

        @Override
        public Map<String, String> validate(Set<String> fieldNames, Boolean raiseValidationError) throws Throwable {
            Map<String, String> validationErrors = super.validate(fieldNames, false);

            if (quantity != null && quantity <= 0) {
                validationErrors.put("quantity", "Quantity must be greater than zero");
            }

            if (!validationErrors.isEmpty() && raiseValidationError) {
                throw new ValidationException(validationErrors);
            }

            return validationErrors;
        }

        public static ObjectNode toRepresent(ObjectNode node) {
            ObjectMapper mapper = new ObjectMapper();
            ObjectNode wrapperNode = mapper.createObjectNode();
            wrapperNode.set("data", node);
            wrapperNode.put("description", "Order information with enhanced representation");
            return wrapperNode;
        }
    }
