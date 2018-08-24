# Lesson 23_6 - Hiding Irrelevant content

## Learning Objectives
* Hiding irrelevant content and showing only the content that is related the
logged in user.

## The Walkthrough

1. Get a copy of Lesson23

2. Edit User class
    * Open User.java
    * Add a field for userImage

```java
@Column(name = "user_image")
private String userImage;
```

3. Edit loaded constructor to User
    * Open the User.java
    * Delete the existing loaded constructor
    * Right-click the word Java and Select Generate -> Constructor
    * Click on email, password, firstName, lastName, enabled, username, userImage
    * Click OK

4. Generate getters and setters for userImage
   * Right click inside the user class and generate getters and setters

5. Edit the DataLoader class
   * Open the DataLoader.java
   * Edit the contents to look like this:

```java  
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Component;

import java.util.Arrays;

@Component
public class DataLoader implements CommandLineRunner {

  @Autowired
  UserRepository userRepository;

  @Autowired
  RoleRepository roleRepository;

  @Autowired
  private PasswordEncoder passwordEncoder;

  @Override
  public void run(String... strings) throws Exception {
    System.out.println("Loading data . . .");

    roleRepository.save(new Role("USER"));
    roleRepository.save(new Role("ADMIN"));

    Role adminRole = roleRepository.findByRole("ADMIN");
    Role userRole = roleRepository.findByRole("USER");

    String userImageUrl = "https://secure.gravatar.com/avatar/?size=35";

    User user = new User("bob@bob.com","bob","Bob",
            "Bobberson", true, "bob", userImageUrl);
    user.setRoles(Arrays.asList(userRole));
    user.setPassword(passwordEncoder.encode(user.getPassword()));
    userRepository.save(user);

    user = new User("jim@jim.com","jim","Jim",
            "Jimmerson", true, "jim", userImageUrl);
    user.setRoles(Arrays.asList(userRole));
    user.setPassword(passwordEncoder.encode(user.getPassword()));
    userRepository.save(user);

    user = new User("sam@every.com","password","Sam",
            "Everyman", true, "everyman", userImageUrl);
    user.setRoles(Arrays.asList(userRole, adminRole));
    user.setPassword(passwordEncoder.encode(user.getPassword()));
    userRepository.save(user);

    String adminImageUrl = "http://gravatar" +
            ".com/avatar/d83c9bca2b678b35596a8d288d74a466?s=35";
    user = new User("admin@secure.com","password",
            "Admin","User", true, "admin", adminImageUrl);
    user.setRoles(Arrays.asList(adminRole));
    user.setPassword(passwordEncoder.encode(user.getPassword()));
    userRepository.save(user);
  }
}
```

6. Edit the HomeController
    * Open the HomeController.java file
    * Replace the existing index method with the following code:

```java
@RequestMapping("/")
public String index(Authentication currentUserDetails, Model model) {
  if(currentUserDetails != null){
  model.addAttribute("user", userService.findByUsername(currentUserDetails
          .getName()));
  }
  return "index";
}
```

7. Edit the index template
    * Open index.html
    * Edit it to look like this:
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="www.thymeleaf.org" xmlns:sec="www.thymeleaf.org/extras/spring-security">
<head>
    <meta charset="UTF-8"/>
    <title>Title</title>
</head>
<body>
<h1>Home Page</h1>
<div sec:authorize="isAuthenticated()">
    <h3>
    Logged user: <span sec:authentication="name">
        <img style="border-radius: 50%; width: 35px; height: 35px;"
                th:src="${user.userImage}" />
    </h3>
    This content is only shown to authenticated users.<br />

    <div sec:authorize="hasAuthority('ADMIN')" >
        This content is only shown to administrators.
    </div>
    <div sec:authorize="hasAuthority('USER')" >
        This content is only shown to users.
    </div>
    The value of the "name" property of the authentication object should appear here.</span><br />
    Roles: <span sec:authentication="principal.authorities">[USER, ADMIN]</span><br /><br />
</div>

<div sec:authorize="isAnonymous()">
    <h2>Insecure Page</h2>
    This content is only shown to anonymous users.
</div>

<div sec:authorize="!isAuthenticated()">
    <a href="/login">Login</a>
    <a href="/register">Register</a>
</div>
<div sec:authorize="isAuthenticated()">
    <a href="/secure">Secure Page</a>
    <a href="/logout">Logout</a>
</div>
</body>
</html>
```
8. Edit the secure template
    * Open secure.html
    * Edit it to look like this:
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="www.thymeleaf.org"
      xmlns:sec="http://www.w3.org/1999/xhtml">
<head>
    <meta charset="UTF-8"/>
    <title>Title</title>
</head>
<body>
<h1>Secure Page</h1>
<div sec:authorize="isAuthenticated()">
    <h3>
        Logged user: <span sec:authentication="name" >
        <img style="border-radius: 50%; width: 35px; height: 35px;"
             th:src="${user.userImage}" />
    </h3>

</div>
<a href="/">Home</a>
<a href="/logout">logout</a>
</body>
</html>
```  

9. Run your application and open a browser, got to http://localhost:8080/

## What is going on

### User class
The variable userImage will be used to store the user image as a string.
Here we are creating a new loaded constructor that contains the userImage.

### Data loader
Data loader will be updated with a string for user image.

### HomeController
findByUsername finds the user by their username and the in-memory authentication
provider gets the username.

### index page
Authentication is used to check if the user is logged in or not.
sec:authorize="isAuthenticated()" - If use user is logged in then the content
inside the div is shown to the logged in user.

sec:authorize="!isAuthenticated()" - If the user is not logged in then the content
inside this div is displayed.

sec:authentication="name" - shows the name of the logged user.

th:src="${user.userImage} - is where we get the user image. 
