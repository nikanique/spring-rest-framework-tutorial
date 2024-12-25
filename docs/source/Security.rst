Security Configuration in Controllers
======================================

The framework provides a built-in mechanism for managing endpoint-level security. Developers can easily define required authorities for different HTTP methods in their controllers, ensuring fine-grained access control. Here's how security can be configured:

**Defining Required Authorities for Endpoints**
-----------------------------------------------------

For generic controllers, you can define the required authorities by overriding the ``getRequiredAuthorities(String HttpMethod)`` method. This allows you to enforce specific permissions for accessing endpoints.

**Example: Securing a Generic List Controller**

The following example demonstrates how to secure a list controller for the ``/schools`` endpoint, requiring the ``school:read`` authority:

.. code-block:: java

    @RequestMapping("/schools")
    @RestController
    @Tag(name = "School")
    public class SchoolListController extends GenericListController<School, Long, SchoolRepository> {
        public SchoolListController(SchoolRepository repository) throws NoSuchMethodException {
            super(repository);
        }

        @Override
        protected List<String> getRequiredAuthorities(String HttpMethod) {
            return List.of("school:read");
        }

        @Override
        public FilterSet configFilterSet() {
            return FilterSet.builder().addFilter("name", "name", FilterOperation.EQUAL, FieldType.STRING).build();
        }

        @Override
        protected Class<?> getDTO() {
            return SchoolDto.class;
        }
    }

  
- The ``getRequiredAuthorities`` method returns a list of required authorities for the specified controller. These authorities are evaluated using the logical **OR** operator, meaning access is granted if the user possesses **any one** of the listed authorities.
- In this example, any request to the ``/schools`` endpoint requires the ``school:read`` authority.

**Defining Required Authorities for generic controllers with more than one HTTP method**
-------------------------------------------------------------------------------------

For these kind of controllers (e.g., GenericQueryController and GenericCommandController), you can define authorities for each HTTP methods by overriding the ``configRequiredAuthorities(Map<String, List<String>> authorities)`` method.

**Example: Securing a Generic Command Controller**

The following example demonstrates how to secure a command controller for the ``/schools`` endpoint, defining authorities for ``POST`` and ``DELETE`` methods:

.. code-block:: java

    @RequestMapping("/schools")
    @RestController
    @Tag(name = "School")
    public class SchoolCommandController extends GenericCommandController<School, Long, SchoolRepository> {
        public SchoolCreateController(SchoolRepository repository) throws NoSuchMethodException {
            super(repository);
        }

        @Override
        protected void configRequiredAuthorities(Map<String, List<String>> authorities) {
            authorities.put("POST", List.of("school:write"));
            authorities.put("DELETE", List.of("school:delete"));

            // The PUT and PATCH will be accessable by all users with any authorities
        }

        @Override
        protected Class<?> getDTO() {
            return SchoolDto.class;
        }
    }

  - The ``configRequiredAuthorities`` method maps HTTP methods (e.g., ``POST``, ``DELETE``) to the required authority lists.
  - In this example:
    - The ``POST`` method requires the ``school:write`` authority.
    - The ``DELETE`` method requires the ``school:delete`` authority.
    - The ``PUT`` and ``PATCH`` will be accessable by all users with any authorities.


To enable authority checking, Spring Security must be properly configured and enabled in your application. 
