Generic Controllers
===================

Generic Controllers standardize and simplify the implementation of RESTful APIs for managing model records, including operations like creating, retrieving, updating, and deleting records. Each generic controller is designed to be extended and configured for specific models, repositories, and use cases.


These controllers are highly configurable, enabling developers to customize the DTO, filters, and lookup fields for their specific use cases. To use these controllers, extend them in your project, inject the appropriate repository, and configure methods like `getDTO()` and `configLookupFilter()`.

===========

GenericListController
--------------

The ``GenericListController`` class is a generic controller designed for use in Spring Boot applications. It provides a common implementation for listing records from a repository with a variety of filtering options. This class is particularly useful when you need to build REST APIs for managing database records where listing and filtering functionalities are required.

This controller exposes ``GET /`` endpoint which returns paginated list of desired model's records from database.

It requires three type parameters:

- **Model**: The class type of the entity (e.g., `Student`).
- **ID**: The type of the model's identifier (e.g., `Long`).
- **ModelRepository**: The repository interface extending ``JpaRespository`` and ``JpaSpecificationExecutor`` (e.g., `StudentRepository`).

This controller provides a standard way to list model's records, which can be extended or customized according to specific requirements. It includes support for various filtering options to enable flexible querying.

Example Usage
^^^^^^^^^^^^^
To use the ``GenericListController``, extend it in a controller class for a specific model and repository. Below is an example of how you can extend `GenericListController` for managing `Student` model.

Assuming that we have Student model in our project:

.. code-block:: java

    import jakarta.persistence.Entity;
    import jakarta.persistence.GenerationType;
    import jakarta.persistence.Id;
    import lombok.Data;
    
    @Entity
    @Data
    public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private Integer age;
    private String major;
    
    }

And this is the StudentRepository. You should extend both ``JpaRepository`` and ``JpaSpecificationExecutor`` to enable the repository to be used in .

.. code-block:: java

    import com.example.demo.model.Student;
    import org.springframework.data.jpa.repository.JpaRepository;
    import org.springframework.data.jpa.repository.JpaSpecificationExecutor;
    import org.springframework.stereotype.Repository;

    @Repository
    public interface StudentRepository extends JpaRepository<Student, Long>, JpaSpecificationExecutor<Student> {
    }


declare a DTO representing your model's field in web API.

   .. code-block:: java

      import io.github.nikanique.springrestframework.annotation.Expose;
      import io.github.nikanique.springrestframework.annotation.ReadOnly;
      import io.github.nikanique.springrestframework.dto.Dto;
      import lombok.Data;

      @Data
      public class StudentDto extends Dto{

      @Expose(source = "name")
      private String fullName;
      private Integer age;
      private String major;
      
      @ReadOnly
      private Long id;

      }




Finally, the controller :

.. code-block:: java

    @RequestMapping("/student")
    @RestController
    @Tag(name = "Student")
    public class StudentController extends GenericListController<Student, Long, StudentRepository> {
        public StudentController(StudentRepository repository) {
            super(repository);
        }

        @Override
        protected Class<?> getDTO() {
            return StudentDto.class;
        }
    }

API Method
^^^^^^^^^^^

``GET /``
Retrieves a paginated list of records with optional filters and sorting.

**Parameters:**

- `page` (default: 0): The page number.
- `size` (default: 10): The number of records per page.
- `sortBy` (default: empty): Field to sort by.
- `direction` (default: `ASC`): Sort direction (`ASC` or `DESC`).

**Example Usage:**

.. code-block:: bash

    curl -X GET "http://localhost:8080/student?page=1&size=5&sortBy=name&direction=DESC"

Constructor
^^^^^^^^^^^
The constructor of ``GenericListController`` is used to inject the repository that will handle db operations for the model. This repository is passed to the superclass constructor where it passes the repository to the service layers. We do not work with repository directly from Controllers.


.. code-block:: java

    public StudentController(StudentRepository repository) {
        super(repository);
    }

Methods
^^^^^^^
- **getDTO()**: This method must be overridden to return the class type of the DTO (Data Transfer Object) that the controller will use to serialize/deserialize the model's records.

.. code-block:: java

    @Override
    protected Class<?> getDTO() {
        return StudentDto.class;
    }

In the example, it returns ``StudentDto.class``.

To learn more about the ``Dto`` please read the :ref:`DTO`.

- **configFilterSet()**: This method configures the filtering options available for listing records. It should be overridden to specify the filters that can be applied to the records. In the example, it uses the ``FilterSet.builder()`` method to create a ``FilterSetBuilder`` instance, which helps in constructing a ``FilterSet`` object with the desired filters. The ``FilterSet.builder()`` method initializes a new ``FilterSetBuilder`` that provides a fluent API for adding filters. The ``addFilter`` method is used to specify a filter on a particular field, its operation, and the field type.

.. code-block:: java

    @Override
    protected FilterSet configFilterSet() {
        return FilterSet.builder()
                .addFilter("name", FilterOperation.CONTAINS, FieldType.STRING)
                .build();
    }

Here, the example sets up a filter for the ``name`` field with a ``CONTAINS`` operation and a ``STRING`` field type:

.. code-block:: bash

    curl -X GET "http://localhost:8080/student?name=Ale"

To read more about the ``FilterSet`` please read the :ref:`FilterSet`.

- **filterByRequest()**: This method is used to customize the filtering criteria based on the HTTP request object's properties. It is called during the processing of listing records to apply additional filters that are derived from the request parameters or headers.


.. code-block:: java

    @Override
    protected List<SearchCriteria> filterByRequest(HttpServletRequest request, List<SearchCriteria> searchCriteria) {
        searchCriteria.add(new SearchCriteria(
                "schoolIid",
                FilterOperation.EQUAL,
                request.getHeader("schoolIid")
        ));
        return searchCriteria;
    }

In the example, the method adds a new ``SearchCriteria`` to the existing list of criteria. This new criteria filters the records based on the value of the ``schoolIid`` header in the HTTP request. The filter operation is set to ``EQUAL``, meaning that only records with a matching ``schoolIid`` will be included in the results.
The method allows for dynamic and request-specific filtering of records, enhancing the flexibility and relevance of the data returned by the API.

To learn more about the ``SearchCriteria`` please read the :ref:`SearchCriteria`.

- **configAllowedOrderByFields()**: This method enables developers to define restrictions on the fields that can be used for sorting in the ``GET /`` endpoint of the generic list controller. By default, the method returns an empty set:

.. code-block:: java

    default Set<String> configAllowedOrderByFields() {
        return Collections.emptySet();
    }

The default implementation imposes **no limitations** on the sorting parameters, allowing all fields to be used for sorting. If you want to impose specific restrictions, they should override this method and return a set of allowed field names. For example:

.. code-block:: java

    @Override
    public Set<String> configAllowedOrderByFields() {
        return Set.of("name", "dateCreated", "status");
    }

**Result**: Only the fields ``name``, ``dateCreated``, and ``status`` will be allowed for sorting.

To prevent sorting entirely, return a set containing a single empty string:

.. code-block:: java

    @Override
    public Set<String> configAllowedOrderByFields() {
        return Set.of("");
    }

**Result**: Sorting will be disabled for the ``GET /`` endpoint.

GenericRetrieveController
------------------

The ``GenericRetrieveController`` class is another generic controller that provides a standardized implementation for retrieving a single record from the database using a repository. This controller exposes a ``GET /{lookup}`` endpoint that locates and retrieves a matching record based on a customizable lookup field.

By default, the controller matches the input value provided in the path variable with the ``id`` field of the records. This behavior can be customized to use different fields for lookup, allowing for flexible record retrieval.

It requires three type parameters:

- **Model**: The class type of the entity (e.g., `Student`).
- **ID**: The type of the model's identifier (e.g., `Long`).
- **ModelRepository**: The repository interface extending ``JpaRespository`` and ``JpaSpecificationExecutor`` (e.g., `StudentRepository`).

This controller provides a standard way to list model's records, which can be extended or customized according to specific requirements. It includes support for various filtering options to enable flexible querying.

.. _retrivecontroller_example_usage:

Example Usage
^^^^^^^^^^^^^
To use the ``GenericRetrieveController``, extend it in a controller class for a specific model and repository. Below is an example of how you can extend `GenericRetrieveController` for managing `Student` model.


.. code-block:: java

    @RequestMapping("/student")
    @RestController
    @Tag(name = "Student")
    public class StudentController extends GenericRetrieveController<Student, Long, StudentRepository> {
        public StudentController(StudentRepository repository) {
            super(repository);
        }

        @Override
        protected Class<?> getDTO() {
            return StudentDto.class;
        }
    }

.. _retrivecontroller_constructor:

Constructor
^^^^^^^^^^^
The constructor of ``GenericRetrieveController`` is used to inject the repository that will handle db operations for the model. This repository is passed to the superclass constructor where it passes the repository to the service layers. We do not work with repository directly from Controllers.


.. code-block:: java

    public StudentController(StudentRepository repository) {
        super(repository);
    }

.. _retrivecontroller_methods:

Methods
^^^^^^^
- **getDTO()**: This method must be overridden to return the class type of the DTO (Data Transfer Object) that the controller will use to serialize/deserialize the model's record.

.. code-block:: java

    @Override
    protected Class<?> getDTO() {
        return StudentDto.class;
    }

In the example, it returns ``StudentDto.class``.

To learn more about the ``Dto`` please read the :ref:`DTO`.


- **configLookupFilter()**: By default, the `GenericRetrieveController` searches for the given lookup value in the `id` field of records. If your model does not have an `id` field or if you want to use a different field for this purpose, you can override this method to specify your desired field.


.. code-block:: java

    @Override
    protected Filter configLookupFilter() {
        return Filter.builder()
                .name("nationalNumber")
                .fieldType(FieldType.INTEGER)
                .operation(FilterOperation.EQUAL)
                .build();
    }

In this example, we specified the ``nationalNumber`` as lookup field which is an ``Integer`` field to retrieve the record.


- **filterByRequest()**: Like ``GenericListController`` this controller use this method to customize the filtering criteria based on the HTTP request object's properties. It is called during the processing of record lookup to apply additional filters that are derived from the request parameters or headers.


.. code-block:: java

    @Override
    protected List<SearchCriteria> filterByRequest(HttpServletRequest request, List<SearchCriteria> searchCriteria) {
        searchCriteria.add(new SearchCriteria(
                "schoolIid",
                FilterOperation.EQUAL,
                request.getHeader("schoolIid")
        ));
        return searchCriteria;
    }


GenericCreateController
------------------------

The ``GenericCreateController`` class is a generic controller for creating model records. It exposes an endpoint with the ``POST`` method for adding new records.

Type Parameters
^^^^^^^^^^^^^^^^

- **Model**: The class type of the entity (e.g., `Student`).
- **ID**: The type of the model's identifier (e.g., `Long`).
- **ModelRepository**: The repository interface extending `JpaRepository`.

Example Usage
^^^^^^^^^^^^^^

Below is an example of how to extend the ``GenericCreateController`` to manage a `Student` model.

.. code-block:: java

    @RequestMapping("/student")
    @RestController
    @Tag(name = "Student")
    public class StudentCreateController extends GenericCreateController<Student, Long, StudentRepository> {
        public StudentCreateController(StudentRepository repository) {
            super(repository);
        }

        @Override
        protected Class<?> getDTO() {
            return StudentDto.class;
        }
    }

Methods
^^^^^^^

- **getDTO()**: This method returns the class type of the DTO used for both deserializing the request body and serializing the response data.

    .. code-block:: java

        @Override
        protected Class<?> getDTO() {
            return StudentDto.class;
        }

    The `StudentDto` class specified in this example will be used as the default DTO for both the request and response in the `GenericCreateController`. This simplifies development when a single DTO is sufficient for both purposes.

    If you need to use different DTOs for request and response, you can override the following methods to provide distinct DTO classes:

    .. code-block:: java

        @Override
        public Class<?> getCreateRequestBodyDTO() {
            return CreateStudentDto.class; // DTO for request body
        }

        @Override
        public Class<?> getCreateResponseBodyDTO() {
            return StudentResponseDto.class; // DTO for response
        }

    By default, both of these methods return the result of `getDTO()`. Overriding them allows for customization of the serialization and deserialization processes for requests and responses independently. This is particularly useful in scenarios where the data requirements for creating a record differ from those for returning a record.


GenericUpdateController
------------------------

The ``GenericUpdateController`` provides a generic implementation for updating model records. It supports both ``PUT`` (complete update) and ``PATCH`` (partial update) methods.

Type Parameters
^^^^^^^^^^^^^^^

- **Model**: The class type of the entity (e.g., `Student`).
- **ID**: The type of the model's identifier (e.g., `Long`).
- **ModelRepository**: The repository interface extending `JpaRepository` and `JpaSpecificationExecutor`.

Example Usage
^^^^^^^^^^^^^

Below is an example of how to extend the ``GenericUpdateController`` for managing `Student` records.

.. code-block:: java

    @RequestMapping("/student")
    @RestController
    @Tag(name = "Student")
    public class StudentUpdateController extends GenericUpdateController<Student, Long, StudentRepository> {
        public StudentUpdateController(StudentRepository repository) {
            super(repository);
        }

        @Override
        protected Class<?> getDTO() {
            return StudentDto.class;
        }
    }

Methods
^^^^^^^

- **configLookupFilter()**: Specifies the field used for locating records. Defaults to the `id` field but can be customized.

    .. code-block:: java

        @Override
        protected Filter configLookupFilter() {
            return Filter.builder()
                    .name("nationalNumber")
                    .fieldType(FieldType.INTEGER)
                    .operation(FilterOperation.EQUAL)
                    .build();
        }

- **getDTO()**: This method returns the class type of the DTO used for both deserializing the request body and serializing the response data.

    .. code-block:: java

        @Override
        protected Class<?> getDTO() {
            return StudentDto.class;
        }

    The `StudentDto` class specified in this example will be used as the default DTO for both the request and response in the `GenericUpdateController`. This simplifies development when a single DTO is sufficient for both purposes.

    If you need to use different DTOs for request and response in ``GenericUpdateController``, you can override the following methods to provide distinct DTO classes:

    .. code-block:: java

        @Override
        public Class<?> getUpdateRequestBodyDTO() {
            return UpdateStudentDto.class; // DTO for request body
        }

        @Override
        public Class<?> getUpdateResponseBodyDTO() {
            return StudentResponseDto.class; // DTO for response
        }

    By default, both of these methods return the result of `getDTO()`. Overriding them allows for customization of the serialization and deserialization processes for requests and responses independently. This is particularly useful in scenarios where the data requirements for updating a record differ from those for returning a record.



- **filterByRequest()**: This method customizes the filtering criteria based on the HTTP request object's properties. It is called during the processing of record lookup before updating to apply additional filters that are derived from the request parameters or headers.


.. code-block:: java

    @Override
    protected List<SearchCriteria> filterByRequest(HttpServletRequest request, List<SearchCriteria> searchCriteria) {
        searchCriteria.add(new SearchCriteria(
                "schoolIid",
                FilterOperation.EQUAL,
                request.getHeader("schoolIid")
        ));
        return searchCriteria;
    }


GenericDeleteController
------------------------

The ``GenericDeleteController`` provides a generic implementation for deleting model records. It exposes an endpoint with the ``DELETE`` method.

Type Parameters
^^^^^^^^^^^^^^^^

- **Model**: The class type of the entity (e.g., `Student`).
- **ID**: The type of the model's identifier (e.g., `Long`).
- **ModelRepository**: The repository interface extending `JpaRepository` and `JpaSpecificationExecutor`.

Example Usage
^^^^^^^^^^^^^

Below is an example of how to extend the ``GenericDeleteController`` for managing `Student` records.

.. code-block:: java

    @RequestMapping("/student")
    @RestController
    @Tag(name = "Student")
    public class StudentDeleteController extends GenericDeleteController<Student, Long, StudentRepository> {
        public StudentDeleteController(StudentRepository repository) {
            super(repository);
        }

        @Override
        protected Class<?> getDTO() {
            return StudentDto.class;
        }
    }

Methods
^^^^^^^

- **configLookupFilter()**: Specifies the field used for locating records. Defaults to the `id` field but can be customized.

    .. code-block:: java

        @Override
        protected Filter configLookupFilter() {
            return Filter.builder()
                    .name("id")
                    .fieldType(FieldType.INTEGER)
                    .operation(FilterOperation.EQUAL)
                    .build();
        }

- **getDTO()**: Returns the DTO class type.


- **filterByRequest()**: This method customizes the filtering criteria based on the HTTP request object's properties. It is called during the processing of record lookup before deleting to apply additional filters that are derived from the request parameters or headers.


.. code-block:: java

    @Override
    protected List<SearchCriteria> filterByRequest(HttpServletRequest request, List<SearchCriteria> searchCriteria) {
        searchCriteria.add(new SearchCriteria(
                "schoolIid",
                FilterOperation.EQUAL,
                request.getHeader("schoolIid")
        ));
        return searchCriteria;
    }


GenericQueryController
------------------------

The `GenericQueryController` is an abstract controller designed for use in Spring Boot applications to facilitate querying and retrieving model records. It provides a consistent and reusable implementation for listing and retrieving entities from a repository. The controller supports advanced features such as filtering, sorting, and response serialization.

It provides these two endpoints for the given model:

  - `GET /`: Retrieves a paginated list of records with optional filters and sorting.
  - `GET /{lookup}`: Retrieves a single record based on a lookup value.


Usage Example
^^^^^^^^^^^^^^^^

Here's an example of how to extend the `GenericQueryController` for a specific entity:

.. code-block:: java

    @RequestMapping("/student")
    @RestController
    @Tag(name = "Student")
    public class StudentController extends GenericQueryController<Student, Long, StudentRepository> {
        public StudentController(StudentRepository repository) {
            super(repository);
        }

        @Override
        protected Class<?> getDTO() {
            return StudentDto.class;
        }
    }

### Parameters

- **Model**: The class type of the entity (e.g., `Student`).
- **ID**: The type of the entity's identifier (e.g., `Long`).
- **ModelRepository**: The repository interface extending `JpaRepository` and `JpaSpecificationExecutor` (e.g., `StudentRepository`).

API Methods
^^^^^^^^^^^^^^^^

``GET /``

Retrieves a paginated list of records with optional filters and sorting.

**Parameters:**

- `page` (default: 0): The page number.
- `size` (default: 10): The number of records per page.
- `sortBy` (default: empty): Field to sort by.
- `direction` (default: `ASC`): Sort direction (`ASC` or `DESC`).

**Example Endoint Usage:**

.. code-block:: bash

    curl -X GET "http://localhost:8080/student?page=1&size=5&sortBy=name&direction=DESC"

**Response:** Returns a paginated list of objects in the specified format.

``GET /{lookup}``

Retrieves a single record based on a lookup value (e.g., ID).

**Parameters:**

- `lookup`: The lookup value used to fetch the record.

**Example Usage:**

.. code-block:: bash

    curl -X GET "http://localhost:8080/student/1"

**Response:** Returns the object corresponding to the lookup value.

Methods
^^^^^^^^^^^^^^^^^^^

- ``getListResponseDTO()`` and ``getRetrieveResponseDTO()``
Specifies the DTO classes used for serializing list and retrieve responses. Defaults to the result of ``getDTO()``. If both the list and retrieve endpoints use the same DTO class, you can simply override only the ``getDTO()`` method to specify the common DTO. 

However, if different DTO classes are needed for list and retrieve operations, you can override these methods individually to provide the appropriate DTO for each endpoint:

.. code-block:: java

    @Override
    protected Class<?> getListResponseDTO() {
        return ListStudentDto.class;
    }

    @Override
    protected Class<?> getRetrieveResponseDTO() {
        return DetailedStudentDto.class;
    }

This separation allows for flexible customization, enabling you to tailor the response structure of each endpoint to the specific needs of your application.

- ``configFilterSet()``: Configures the filters available for querying in ``GET /`` endpoint listing the records. By default, returns an empty filter set.

- ``configLookupFilter()``: Specifies the filter used for retrieving a single record by lookup value in ``GET /{lookup}`` endpoint. Default: ID field with Equal filter.

- ``configAllowedOrderByFields()`` : This method enables developers to define restrictions on the fields that can be used for sorting in the ``GET /`` endpoint of the generic list controller. By default, the method returns an empty set:

.. code-block:: java

    default Set<String> configAllowedOrderByFields() {
        return Collections.emptySet();
    }

The default implementation imposes **no limitations** on the sorting parameters, allowing all fields to be used for sorting. If you want to impose specific restrictions, they should override this method and return a set of allowed field names. For example:

.. code-block:: java

    @Override
    public Set<String> configAllowedOrderByFields() {
        return Set.of("name", "dateCreated", "status");
    }

**Result**: Only the fields ``name``, ``dateCreated``, and ``status`` will be allowed for sorting.

To prevent sorting entirely, return a set containing a single empty string:

.. code-block:: java

    @Override
    public Set<String> configAllowedOrderByFields() {
        return Set.of("");
    }

**Result**: Sorting will be disabled for the ``GET /`` endpoint.
  
GenericCommandController
-------------------------

The ``GenericCommandController`` is an abstract class designed to handle generic CUD (Create, Update, Delete) operations in a Spring Boot application. The controller exposes endpoints for creating, updating, partially updating, and deleting resources.

Example Usage
^^^^^^^^^^^^^^^^^

.. code-block:: java

    @RequestMapping("/student")
    @RestController
    @Tag(name = "Student")
    public class StudentCommandController extends GenericCommandController<Student, Long, StudentRepository> {
        public StudentCommandController(StudentRepository repository) {
            super(repository);
        }

        @Override
        protected Class<?> getDTO() {
            return StudentDto.class;
        }
    }

**Type Parameters**

- **Model**: The class type of the entity (e.g., ``Student``).
- **ID**: The type of the entity's identifier (e.g., ``Long``).
- **ModelRepository**: The repository interface extending ``JpaRepository`` and ``JpaSpecificationExecutor`` (e.g., ``StudentRepository``).


Endpoints
^^^^^^^^^^

- **POST** ``/``  
  Creates a new resource in the database. The request body is deserialized using the ``getCreateRequestBodyDTO()`` DTO.

  .. code-block:: java

      @PostMapping("/")
      public ResponseEntity<ObjectNode> post(HttpServletRequest request) throws IOException {
          return this.create(this, request);
      }

- **PUT** ``/{lookup}``  
  Fully updates an existing resource. The request body is deserialized using the ``getUpdateRequestBodyDTO()`` DTO.

  .. code-block:: java

      @PutMapping("/{lookup}")
      public ResponseEntity<ObjectNode> put(@PathVariable(name = "lookup") Object lookupValue, HttpServletRequest request) throws Throwable {
          return this.update(this, lookupValue, request);
      }

- **PATCH** ``/{lookup}``  
  Partially updates an existing resource. Similar to PUT but allows partial updates.

  .. code-block:: java

      @PatchMapping("/{lookup}")
      public ResponseEntity<ObjectNode> partialUpdate(@PathVariable(name = "lookup") Object lookupValue, HttpServletRequest request) throws Throwable {
          return this.partialUpdate(this, lookupValue, request);
      }

- **DELETE** ``/{lookup}``  
  Deletes a resource identified by the lookup value.

  .. code-block:: java

      @DeleteMapping("/{lookup}")
      public ResponseEntity<Void> delete(HttpServletRequest request, @PathVariable(name = "lookup") Object lookupValue) {
          return deleteObject(this, request, lookupValue);
      }

Customization Points
^^^^^^^^^^^^^^^^^^^^^^

- ``getCreateRequestBodyDTO()`` and ``getCreateResponseBodyDTO()``  
  Specifies the DTOs used for serializing/deserializing create request and response bodies. Defaults to the value of ``getDTO()``.

  .. code-block:: java

      @Override
      public Class<?> getCreateRequestBodyDTO() {
          return CreateStudentDto.class;
      }

      @Override
      public Class<?> getCreateResponseBodyDTO() {
          return StudentResponseDto.class;
      }

- ``getUpdateRequestBodyDTO()`` and ``getUpdateResponseBodyDTO()``  
  Specifies the DTOs used for serializing/deserializing update request and response bodies.

- ``configLookupFilter()``  
  Configures the filter used to identify a resource during update or delete operations. Defaults to filtering by an ``id`` field.

  .. code-block:: java

      @Override
      protected Filter configLookupFilter() {
          return new Filter("id", FilterOperation.EQUAL, FieldType.LONG);
      }

If all endpoints use the same DTO class, you can simply override only the ``getDTO()`` method to specify the common DTO.
