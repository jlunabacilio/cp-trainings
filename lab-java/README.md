# REST in JAVA

This lab showcase the implementation of a simple users management using Java programming language. Main purpose is demostrate how does GitHub Copilot helps to increase speed and reduces errors when programming.

## Tools and versions required
- Java 17.0.11
- Maven 3.6.3
- Use Visual Studio code and next plugins:
  - Extension Pack for Java
  - GitHub Copilot
  - GitHub Copilot Chat

## How to start
1. Clone next GitHub Repository:

https://github.com/jlunabacilio/java-users-demo

2. Check it out the files created:

| Name | Description | 
|----------|----------|
| src/main/java/com/demo/Users.java | La clase Users funciona como una entidad de usuario en una base de datos. Utiliza JPA para mapear la clase a una tabla de base de datos. Los campos id, fullname y status representan las columnas de la tabla. Los métodos getter y setter permiten acceder y modificar estos campos. | 
| src/main/java/com/demo/UsersRepository.java | El script define una interfaz UsersRepository que extiende JpaRepository. Esto proporciona métodos CRUD (Crear, Leer, Actualizar, Eliminar) para la entidad Users sin necesidad de implementar estos métodos manualmente. La interfaz permite interactuar con la base de datos de manera sencilla. |

3. Open _Users.java_ script in the code editor, open in-line chat and ask to the Copilot agent using the slash command "/explain": `/explain Explain how thte selected code works`.

4. Repeat same step for _UsersRepository.java_ file

5. It is required to create _UsersApplication_ and _UsersController_ to make it works.

6. You can also try adding comments to your code `Add comments to the selected code`.

## Creating users in UsersApplication.java
Idea of UsersApplication.java is to inizialize using some static data when the application starts.

1. Create _UsersApplication.java_ file in src/main/java/com/demo

2. Add next dependencies at first level of the file.

```java
package com.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import java.util.List;
import jakarta.annotation.PostConstruct;
```

3. Start adding next comments:

```
// Initializes the Spring Boot Application

Inside public class UsersApplication:
// Dependency injection of UsersRepository
// Constructor to initialize UsersRepository
// Main method to run the Spring Boot application
// Method annoted with @PostConstruct to run after the bean initialization
// Create a list of users with name, active or not
// Saving the list of users to the repository
// Featching all users from the repository and printing them
```

4. Ask to the GitHub Copilot Chat to explain the entire code.

Full code:
``` java
package com.demo;

import jakarta.annotation.PostConstruct;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import java.util.List;

@SpringBootApplication
public class UsersApplication {

    private final UsersRepository usersRepository;

    public UsersApplication(UsersRepository usersRepository) {
        this.usersRepository = usersRepository;
    }

    public static void main(String[] args) {
        SpringApplication.run(UsersApplication.class, args);
    }

    @PostConstruct
    public void init() {
        List<Users> users = List.of(
            new Users("John Doe", true),
            new Users("Jane Doe", false)
        );

        usersRepository.saveAll(users);
        usersRepository.findAll().forEach(System.out::println);
    }
    
}
```

## Creating UsersController.java

Idea of _UsersController.java_ is to be responsible for handling HTTP requests related to Users objects.

### Creating UsersController.java using GitHub Copilot
1. Create _UsersController.java_ file in `src/main/java/com/demo`

2. Copy and paste next libraries:
```java
package com.demo;

import java.net.URI;
import java.util.List;

import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
```

3. Type next prompt after libraries: `//generate a rest controller with th path of /`, this line will trigger suggestions code from GitHub Copilot, clic tab up to get:
```java
@RestController
@RequestMapping("/")
@Validated
```

4. In public class UsersController, start typing next code line: ` private final UsersRepository repository;` and start pressing tab to see suggestions of GitHub Copilot.

5. Ask to the copilot chat using next prompt: `Generate a get method with the path of /users`.
6. Ask again for get by id, post, put and delete methods (functional code below).
7. Test it building the _.jar_ file using next command: `mvn clean package` and run using: `java -jar target/java-users-demo-1.0-SNAPSHOT.jar`

```java
package com.demo;

import java.net.URI;
import java.util.List;

import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/")
@Validated
public class UsersController {
    private final UsersRepository repository;

    public UsersController(UsersRepository repository) {
        this.repository = repository;
    }

    @GetMapping("/users")
    public ResponseEntity<List<Users>> getAllUsers() {
        List<Users> users = repository.findAll();
        return ResponseEntity.ok(users);
    }

    @GetMapping("/users/{id}")
    public ResponseEntity<Users> getUserById(@PathVariable Long id) {
        Users users = repository.findById(id).orElse(null);
        return ResponseEntity.ok(users);
    }

    @PostMapping("/users")
    public ResponseEntity<Users> createUser(@RequestBody Users users) {
        Users createdUsers = repository.save(users);
        return ResponseEntity.created(URI.create("/" + createdUsers.getId())).body(createdUsers);
    }
}
```

### Creating tests using GitHub Copilot

1. Uncomment libraries in _UsersControllerTest.java_ from `src/test/java/com/demo`
Template
```java
package com.demo;

import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.when;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;

import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;

@WebMvcTest
@AutoConfigureMockMvc

public class UsersControllerTest {
    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UsersRepository usersRepository;

}
```
3. In `public class UsersControllerTest` start typing: `@Autowired` and press tab for the suggestions given by GitHub Copilot
4. You can get multiple suggestions by clicking on `View > Command Palette > GitHub Copilot: Open Completions Panel`
5. To get GitHub Copilot suggestions on test, ask to the Copilot Chat to  `/tests create the test for the POST endpoint of users controller`.
6. Run test directly from Visual Studio Code or running `mvn test` command.

```java
package com.demo;

import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.when;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;

import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;

@WebMvcTest
@AutoConfigureMockMvc

public class UsersControllerTest{
    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UsersRepository repository;

    // generate a test for create user method
    @Test
    public void testCreateUser() throws Exception{
        Users user = new Users("John Doe", true);
        when(repository.save(any(Users.class))).thenReturn(user);

        mockMvc.perform(post("/users")
            .content(new ObjectMapper().writeValueAsString(user))
            .contentType(MediaType.APPLICATION_JSON))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.fullname").value("John Doe"));
    }

}
```

## Contact
If you have any questions or need further assistance, feel free to contact me at josea.luna@softtek.com or david.rodriguezh@softtek.com.
