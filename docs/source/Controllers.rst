Controllers
===========

ListController
--------------

The ``ListController`` class is a generic controller designed for use in Spring Boot applications. It provides a common implementation for listing records from a repository with a variety of filtering options. This class is particularly useful when you need to build REST APIs for managing database records where listing and filtering functionalities are required.

This controller exposes ``GET /`` endpoint which returns paginated list of desired model's records from database.

It requires three type parameters:

- **Model**: The class type of the entity (e.g., `Student`).
- **ID**: The type of the model's identifier (e.g., `Long`).
- **ModelRepository**: The repository interface extending ``JpaRespository`` and ``JpaSpecificationExecutor`` (e.g., `StudentRepository`).

This controller provides a standard way to list model's records, which can be extended or customized according to specific requirements. It includes support for various filtering options to enable flexible querying.

Example Usage
^^^^^^^^^^^^^
To use the ``ListController``, extend it in a controller class for a specific model and repository. Below is an example of how you can extend `ListController` for managing `Student` model.


.. code-block:: java

    @RequestMapping("/student")
    @RestController
    @Tag(name = "Student")
    public class StudentController extends ListController<Student, Long, StudentRepository> {
        public StudentController(StudentRepository repository) {
            super(repository);
        }

        @Override
        protected Class<?> getDTO() {
            return Student.class;
        }
    }

Constructor
^^^^^^^^^^^
The constructor of ``ListController`` is used to inject the repository that will handle db operations for the model. This repository is passed to the superclass constructor where it passes the repository to the service layers. We do not work with repository directly from Controllers.


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

Here, the example sets up a filter for the ``name`` field with a ``CONTAINS`` operation and a ``STRING`` field type.

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



RetrieveController
------------------

The ``RetrieveController`` class is another generic controller that provides a standardized implementation for retrieving a single record from the database using a repository. This controller exposes a ``GET /{lookup}`` endpoint that locates and retrieves a matching record based on a customizable lookup field.

By default, the controller matches the input value provided in the path variable with the ``id`` field of the records. This behavior can be customized to use different fields for lookup, allowing for flexible record retrieval.

It requires three type parameters:

- **Model**: The class type of the entity (e.g., `Student`).
- **ID**: The type of the model's identifier (e.g., `Long`).
- **ModelRepository**: The repository interface extending ``JpaRespository`` and ``JpaSpecificationExecutor`` (e.g., `StudentRepository`).

This controller provides a standard way to list model's records, which can be extended or customized according to specific requirements. It includes support for various filtering options to enable flexible querying.

Example Usage
^^^^^^^^^^^^^
To use the ``RetrieveController``, extend it in a controller class for a specific model and repository. Below is an example of how you can extend `RetrieveController` for managing `Student` model.


.. code-block:: java

    @RequestMapping("/student")
    @RestController
    @Tag(name = "Student")
    public class StudentController extends RetrieveController<Student, Long, StudentRepository> {
        public StudentController(StudentRepository repository) {
            super(repository);
        }

        @Override
        protected Class<?> getDTO() {
            return Student.class;
        }
    }

Constructor
^^^^^^^^^^^
The constructor of ``RetrieveController`` is used to inject the repository that will handle db operations for the model. This repository is passed to the superclass constructor where it passes the repository to the service layers. We do not work with repository directly from Controllers.


.. code-block:: java

    public StudentController(StudentRepository repository) {
        super(repository);
    }

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


- **configLookupFilter()**: By default, the `RetrieveController` searches for the given lookup value in the `id` field of records. If your model does not have an `id` field or if you want to use a different field for this purpose, you can override this method to specify your desired field.


.. code-block:: java

    @Override
    protected Filter configLookupFilter() {
        return Filter.builder()
                .name("nationalNumber")
                .fieldType(FieldType.INTEGER)
                .operation(FilterOperation.EQUAL)
                .build();
    }

In this example, we specified the ``nationalNumber`` field which is an ``Integer`` field to lookup the records.