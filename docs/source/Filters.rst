Filters and FilterSet
===========

The ``FilterSet`` and ``Filter`` classes provide a structured and flexible way to define filters for your data retrieval operations. Inspired by Django Rest Framework (DRF) filters, these components allow you to define filtering criteria for database queries. The Filter supports operations like filtering by equality, ranges, inclusion in a set, substring matching, and more, making it highly adaptable to various requirements.

``FilterSet`` Class
--------------

The ``FilterSet`` class represents a collection of filters. It provides a fluent API for creating and managing filters that can be applied to your data models.

Constructor
^^^^^^^^^^^^^
- ``FilterSet()``: Creates an empty ``FilterSet``.
- ``FilterSet(Set<Filter> filters)``: Creates a ``FilterSet`` with the given set of filters.

Methods
^^^^^^^^^^^^^
- ``Set<Filter> getFilters()``: Retrieves all filters in the ``FilterSet``.
- ``void add(Filter filter)``: Adds a single filter to the ``FilterSet``.

Builder
^^^^^^^^^^^^^
The ``FilterSetBuilder`` provides methods to construct a ``FilterSet`` instance fluently:

.. code-block::java

    protected FilterSet configFilterSet() {
        return FilterSet.builder()
                .addFilter("nationalNumber", FilterOperation.EQUAL, FieldType.INTEGER)
                .addFilter("date_time", "class_date_time", FilterOperation.BETWEEN, FieldType.DATE_TIME)
                .build();
    }

The example applies set of 2 filters on the endpoint result:

  - ``nationalNumber``: Checks if the ``nationalNumber`` equals to given ``nationalNumber`` parameter. Here the endpoint filter parameter will be same as the model field name.
  - ``date_time``: This filter parameter has a name (``date_time``) different from the corresponding model field (``class_date_time``). It is used to check whether the value of the ``class_date_time`` field of the records falls within a specified datetime range.
  
Finally we compile the FilterSet by calling ``.build()`` method.


1. **Basic Filter**:
   
.. code-block:: java

        FilterSetBuilder addFilter(String name, FilterOperation operation, FieldType fieldType)
        
    Adds a filter with a name, operation, and type.

2. **Filter with Help Text**:
   
.. code-block:: java

        FilterSetBuilder addFilter(String name, FilterOperation operation, FieldType fieldType, String helpText)

    Adds a filter and a descriptive help text.

3. **Model-Field Mapping**:

.. code-block:: java

        FilterSetBuilder addFilter(String name, String modelFieldName, FilterOperation operation, FieldType fieldType)

    Links a filter name to a specific model field.


Filter
^^^^^^^^^^^^^

The ``Filter`` class represents an individual filtering criterion. It encapsulates the details required for filtering, such as the field name, filtering operation, field type, and optional help text.

Filter class properties:

- ``String name``: The filter's name (e.g., ``id``, ``name``).
- ``String modelFieldName``: The mapped model field name (e.g., ``country__name``).
- ``FilterOperation operation``: The filtering operation.
- ``FieldType fieldType``: The type of the field being filtered (e.g., ``STRING``, ``INTEGER``).
- ``String helpText``: A description of the filter's purpose (optional).



FilterOperation
^^^^^^^^^^^^^
The `FilterOperation` enum defines the types of filtering operations supported:

- ``EQUAL``: Checks for equality.
- ``GREATER``: Filters values greater than the given input.
- ``GREATER_OR_EQUAL``: Filters values greater than or equal to the input.
- ``LESS``: Filters values less than the given input.
- ``LESS_OR_EQUAL``: Filters values less than or equal to the input.
- ``BETWEEN``: Filters values between two inputs.
- ``CONTAINS``: Checks if the field contains the given substring.
- ``IN``: Filters values that are part of a given set.


The filtering system supports querying nested model fields by mapping filter names to specific paths in related models. For example, the ``continent`` filter (mapped to ``country__continent__name``) allows you to check if a substring is present in the ``name`` field of the ``continent`` model, which is linked through the ``country`` model. This enables seamless filtering across relationships in a structured and intuitive manner.

.. code-block:: java

        protected FilterSet configFilterSet() {
        return FilterSet.builder()
                .addFilter("city_name", FilterOperation.CONTAINS, FieldType.STRING, "Check containing a name")
                .addFilter("continent", "country__continent__name", FilterOperation.CONTAINS, FieldType.STRING, "Check containing a country name")
                .addFilter("number_of_people", "population", FilterOperation.BETWEEN, FieldType.INTEGER, "Retrieves people with ages greater than provided old number")
                .build();
        }

In the example the ``continent`` (mapped to ``country__continent__name``): Checks for a substring in related model ``country``'s ``continent``'s '``name``. 
Other filters:

  - ``name``: Checks if the name contains a substring.
  - ``number_of_people`` (mapped to ``population``): Filters cities within a specific population range in specific continent.
