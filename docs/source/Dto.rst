
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

The examples below illustrate how to leverage these features in your DTO classes.

Read-Only and Write-Only Fields
-------------------------------

**Description**  
- **Read-only fields**: Included in the serialized output but cannot be set during deserialization.
- **Write-only fields**: Can be set during deserialization but are not included in the serialized output.

**Implementation**::

    @ReadOnly
    private Long id;

    @WriteOnly
    private Long countryId;

**Example Use Case**  
In the `CarDTO` class:
- `id` is a read-only field, ensuring the client cannot modify it.
- `countryId` is a write-only field, allowing seamless association with a country by its ID during object creation.

Field Exposure and Custom Mapping
---------------------------------

**Description**  
The `@Expose` annotation allows customization of:

- **Source**: Specify the origin of the field value, including nested fields.
- **Custom format**: Format the output value (e.g., decimal places, number formatting).
- **Static method processing**: Apply a transformation to the field value via a static method.

**Implementation**::

    @Expose(source = "year", methodName = "getTheDecade")
    @ReadOnly
    private Integer decade;

    @Expose(source = "country__continent")
    @ReadOnly
    private String continent;

**Example**  
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

**Example Use Case**  
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

**Example**  
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

**Example**  
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

**Example**  
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

**Example**  
- `mark` is formatted to two decimal places.
- `population` uses a thousands separator.

