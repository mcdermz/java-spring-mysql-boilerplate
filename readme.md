# Setting up a CRUD app with Java/Gradle, Spring/JPA and MySQL
---
## Boilerplate:

* `cd` into your project directory
* `$ git clone https://github.com/spring-guides/gs-rest-service.git`
* `$ cd gs-rest-service/initial`

Rename your app:
```
$ mv src/main/java/hello src/main/java/<app name>
```
Add the `Application` file needed to compile:
* `$ touch src/main/java/<app name>/Application.java`

```
package <app name>; // rename to your application's name

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

## Connect your project to MySQL:

**Create MySQL database**

* `$ mysql -uroot -e "create database if not exists example_db;"`
* `$ mysql -uroot -e "create database if not exists example_db_test;"`
* Open a new terminal tab and connect to the database:
  - `$ mysql -uroot example_db`

**Open your project in IntelliJ**
* Add these two lines to the top-level `dependencies` section in `build.gradle`:
```
compile('org.springframework.boot:spring-boot-starter-data-jpa')
runtime('mysql:mysql-connector-java')
```
* Refresh Gradle from within IntelliJ

**Setup the connection**
* `$ mkdir src/main/resources`
* `$ touch src/main/resources/application.properties`

  ```
  // from application.properties

  spring.datasource.url=jdbc:mysql://localhost/<DATABASE NAME>?serverTimezone=UTC
  spring.datasource.username=root
  spring.datasource.password=
  spring.datasource.driver-class-name=com.mysql.jdbc.Driver
  spring.jpa.hibernate.ddl-auto=update
  spring.jpa.database-platform=org.hibernate.dialect.MySQLDialect
  ```

## Create an Entity and Respository:

* Create a POJO (plain old Java object) and name it `Resource-name`
* Annotate the class with `@Entity`
* Optionally annotate the class with `@Id` and `@GeneratedValue`
* Add getters and setters for each of the fields
  - Do this easily in IntelliJ by typing `command-N` from within your `Entity` class

Example (make sure you are using the right `package` and use your IDE to `import` dependencies):
```
@Entity
@Table(name = "lessons")
public class Lesson {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    private String title;


    @Column(columnDefinition = "date")
    @JsonFormat(pattern = "yyyy-MM-dd")
    private Date deliveredOn;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public Date getDeliveredOn() {
        return deliveredOn;
    }

    public void setDeliveredOn(Date deliveredOn) {
        this.deliveredOn = deliveredOn;
    }
}
```
Create the Repository:
* Create a POJO named `<Resource-name>Repository`

```
public interface LessonRepository extends CrudRepository<Lesson, Long> {


}
```

## Create a Controller with CRUD mapping
* Create a POJO named `<Resource-name>sController`

```
@RestController
@RequestMapping("/guitars")
public class GuitarController {
    private final GuitarRepository repository;

    public GuitarController(GuitarRepository repository) {
        this.repository = repository;
    }

    @GetMapping("/")
    public Iterable<Guitar> all() {
        return this.repository.findAll();
    }

    @PostMapping("")
    public Guitar create(@RequestBody Guitar guitar) {
        return this.repository.save(guitar);
    }

    @GetMapping("/{id}")
    public Guitar show(@PathVariable("id") long id) {
        return this.repository.findOne(id);
    }

    @PutMapping("/{id}")
    public Guitar put(@PathVariable("id") long id, @RequestBody Guitar guitar) {
        Guitar dbGuitar = repository.findOne(id);
        if (guitar.getBrand() != null) {
            dbGuitar.setBrand(guitar.getBrand());
        }
        if (guitar.getModel() != null) {
            dbGuitar.setModel(guitar.getModel());
        }
        return this.repository.save(dbGuitar);
    }

    @DeleteMapping("/{id}")
    public void delete(@PathVariable("id") long id) {
        this.repository.delete(id);
    }
}
```
Run with `./gradlew bootRun`
