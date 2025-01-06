Usage
=====

.. _installation:

Installation
------------

To install the Spring REST Framework, include the following dependencies in your project:

.. code-block:: xml

   <dependencies>
      <!-- Other dependencies -->
      <dependency>
         <groupId>io.github.nikanique</groupId>
         <artifactId>spring-rest-framework</artifactId>
         <version>2.1.0</version>
      </dependency>
   </dependencies>

.. _getting_started:

Getting Started
----------------

To start using the library, follow these steps:

1. Add the necessary dependencies to your project:
   
   Add the required dependencies into your project following the
   installation section.

2. Declare your models and repositories:

   For example, declare a Student model.

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

   
   Create Repository for you model:

   .. code-block:: java

      import com.example.demo.model.Student;
      import org.springframework.data.jpa.repository.JpaRepository;
      import org.springframework.data.jpa.repository.JpaSpecificationExecutor;
      import org.springframework.stereotype.Repository;
      
      @Repository
      public interface StudentRepository extends JpaRepository<Student, Long>, JpaSpecificationExecutor<Kid> {
      }
    

3. Configure your API endpoints and serializers DTO:
   
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
      private String grade;
      
      @ReadOnly
      private Long id;
      }
   
   Create your Controller by extending **QueryController** which will generate List and Retrieve endpoint for you.

   .. code-block:: java

      @RequestMapping("/student")
      @RestController
      @Tag(name = "Student")
      public class StudentController extends QueryController<Kid, Long, StudentRepository> {
         public StudentController(StudentRepository repository) {
               super(repository);
         }
      
         @Override
         protected Class<?> getDTO() {
               return StudentDto.class;
         }
      }  
      

4. Run your application, and enjoy your APIs.
   
   You can see your API at http://app-server:port/swagger-ui.html

