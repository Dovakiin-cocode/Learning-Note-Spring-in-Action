# Chapter 1 
Referenced from Spring in Action 
After constructing a Spring Boot project from spring.io, you can import it into your IDE (mine is IntelliJ).
Except the default main class of Spring Boot, we declare another class- HomeController
The code of this class is shown as following:
```
@Controller
public class HomeController {
    @GetMapping("/")
    public String home(){
        return "home";
    }
}
```
Description:
- The @Controller(as well as @Component;@Service;@Repository) identifies this class as a component which can be discovered by the auto-scanning feature of Spring.
- The @GetMapping("/") annotation denotes that is a GET request is received from the root path /, the annotated method should handle the request.
- The return "home" string is interpereted as the logical name of a VIEW. The template name is derived from the logical view name by PREFIXING it with /templates/ and POSTFIXING it with .html. The resulting path is thus /templates/home.html . So the template files should be placed at /src/main/resources/template/home.html. This is the content of the home.html file:
```
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:th="http://www.thymeleaf.org"
      lang="en">
<head>
    <meta charset="UTF-8">
    <title>Taco Cloud</title>
</head>
<body>
    <h1>Welcome to...</h1>
    <img th:src="@{/images/TacoCloud.jpg}"/>
<!--use Thymeleaf th:src attribute and an @{...} expression to reference the image with a context relative path-->
</body>
</html>
```
Description
- The statement <img th:src="@{/images/TacoCloud.jpg}"> is to use Thymeleaf's th:src attribute and an @{...} expression to reference the image with a context relative path.

```
package com.example.tacocloud_1;

import static org.hamcrest.Matchers.containsString;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static
        org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
import static
        org.springframework.test.web.servlet.result.MockMvcResultMatchers.view;
import com.example.tacocloud_1.Controllers.HomeController;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

/**
 * @author:Junlin Lu
 * @date:26/07/2020
 * @version:1.0.0
 * @description:
 */
@RunWith(SpringRunner.class)
@WebMvcTest(HomeController.class)
public class HomeControllerTest {
    @Autowired
    private MockMvc mockMvc;

    @Test
    public void testHomePage() throws Exception{
        mockMvc.perform(get("/")).//perform a GET request to the CONTROLLER
                andExpect(status().isOk()).//expect an OK as the status of the response
                andExpect(view().name("home")).//the responded view should have a name "home"
                andExpect(content().string(//the rendered view should contain the text "Welcome to..."
                        containsString("Welcome to...")));
    }
}

```
Description:
- @WebMvcTest(HomeController.class) it arranges for HomeController to be registered in Spring MVC so that you can throw request against it.


# Draft
In a spring web application:
- it is a controller's job to fetch and process the data.
- it is a view's job to render the data into HTML that will displayed in the browser.

We need the following three components in support of the "taco creation page"
- A domain class that defines the Properties of a taco ingredient.
- A Spring MVC controller class that fetches ingredients information and passes it along to the view.
- A VIEW template that renders a list of ingredients in the user's browser.

Establish the domain:
- The ingredient class
```
package com.example.tacocloud_1.Tacos;

import lombok.Data;
import lombok.RequiredArgsConstructor;

/**
 * @author:Junlin Lu
 * @date:27/07/2020
 * @version:1.0.0
 * @description:
 */
@Data
@RequiredArgsConstructor
public class Ingredient {

    private final String id;
    private final String name;
    private final Type type;

    public static enum Type{
        WRAP,PROTEIN,VEGGIES,CHEESE,SAUCE
    }
}
```
# Description
It is amazing that there are no getter, setter, hashcode... 
That is because we use @Data annotation (actually Lombok), to generate all of the missing methods as well as a constructor that accepts all （**not sure** the **final**） properties as arguments and keep the code slim and trim.

Controllers are the major players in Spring MVC framework, whose primary job is to handle HTTP request off to a view to render an HTML(based on browser) or write data directly to the body of a response(RESTful).

Now we need a simple controller which :
- Handle HTTP GET request where the request path is /design
- Build a list of ingredients
- Hand the request and the ingredient data off to a view template to be rendered as a HTML file and sent to the requesting web-browser.
The controller is like:
```
package com.example.tacocloud_1.Controllers;

import com.example.tacocloud_1.Tacos.Ingredient;
import com.example.tacocloud_1.Tacos.Taco;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;

import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

/**
 * @author:Junlin Lu
 * @date:27/07/2020
 * @version:1.0.0
 * @description:
 */
@Slf4j
@Controller
@RequestMapping("/design")
public class DesignTacoController {
    @GetMapping
    public String showDesignForm(Model model){
        List<Ingredient> ingredients = Arrays.asList(
                new Ingredient("FLTO", "Flour Tortilla", Ingredient.Type.WRAP),
                new Ingredient("COTO", "Corn Tortilla", Ingredient.Type.WRAP),
                new Ingredient("GRBF", "Ground Beef", Ingredient.Type.PROTEIN),
                new Ingredient("CARN", "Carnitas", Ingredient.Type.PROTEIN),
                new Ingredient("TMTO", "Diced Tomatoes", Ingredient.Type.VEGGIES),
                new Ingredient("LETC", "Lettuce", Ingredient.Type.VEGGIES),
                new Ingredient("CHED", "Cheddar", Ingredient.Type.CHEESE),
                new Ingredient("JACK", "Monterrey Jack", Ingredient.Type.CHEESE),
                new Ingredient("SLSA", "Salsa",Ingredient.Type.SAUCE),
                new Ingredient("SRCR", "Sour Cream", Ingredient.Type.SAUCE)
        );
        Ingredient.Type[] types = Ingredient.Type.values();
        for(Ingredient.Type t:types){
            model.addAttribute(t.toString().toLowerCase(),filterByType(ingredients,t));
        }
        model.addAttribute("design",new Taco());
        return "design";
    }

    /**
     * Pass in the ingredients list and get the related ingredients of the passed in Type
     * Use Stream to filter the elements which has the same type and then collect them as a new List
     * and return.
     * @param ingredients
     * @param type
     * @return
     */
    private List<Ingredient> filterByType(List<Ingredient> ingredients, Ingredient.Type type){
        return ingredients.stream().filter(x->x.getType().equals(type)).collect(Collectors.toList());
    }
}
```
# Description:
- @RequestMapping("/design") and @GetMapping are two annotations which are used to derive the GET request from the path "/design".
- The family of request-mapping annotations:@RequestMapping;@GetMapping;@PostMapping;@PutMapping;@PutMapping;@DeleteMapping;@PatchMapping
- Here I add the method filterByType() which is not given in the book. It makes use of Stream of Java. By passing in the ingredients List and the target type, the Stream generated from the List is filtered according to the target type and re-collect as a new List and is returned.
- The list of ingredients is added as an attribute of the Model object passed into the showDesignForm() method. Model is the class whose instance ferries data between the Controller and View. Ultimately, data that's placed in Model attributes is copied into the servlet response attributes, where the view can find them. This method returns a String "design" which is the logical name of the view which is used to render the Model to the browser

We use Thymeleaf to define the View. After adding the dependency of Thymeleaf, Spring Boot autoconfiguration will find that Thymeleaf is in the classpath and will automatically create the beans supporting Thymeleaf views for Spring MVC.
Thymeleaf is not aware of Spring Model abstraction thus unable to work with the data that the controller placed in Model. But Thymeleaf can work with servlet request attributes. Therefore before Spring hands the request over to the view, it copies the model data into request attributes that Thymeleaf is ready to access.
i.e Say there were a request attribute whose key is "message", and we wanted it to be rendered into an HTML <p> tag by Thymeleaf:

<p th:text="${message}">placeholder message</p>
When the template is rendered into HTML, the body of the <p> element will be replaced with the value of the servlet request attribute whose key is "message".
- th:text is a Thymeleaf-namespaced attribute to perform replacement.
- ${} operator means to use the value of the variable in the curly brace.
Also, there is another Thymeleaf-namespaced attribute "th:each" iterating a collection of elements.
i.e.
```
<h3>Designate your wrap:</h3>
<div th:each="ingredient:${wrap}">
    <input name="ingredients" type="checkbox" th:value="${ingredient.id}"/>
    <span th:text:"${ingredient.name}">INGREDIENT</span><br/>
</div>
```
Firstly, use the th:each attribute in the div tag to repeat the <div> tags once per item in the collection found in **wrap** request attribute.
Secondly, in the <input> tag, the value of the <input> element "value" with th:value finding the id in the ingredient item.
Thirdly, the ingredient.name is used to replace the INGREDIENT text by the Thymeleaf-namespaced attribute **th:text**.

The Thymeleaf template code is shown as following:
```
<!DOCTYPE html>
<!--suppress ALL-->
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:th="http://www.thymeleaf.org"
      lang="en">
<head>
    <meta charset="UTF-8">
    <title>Taco Cloud</title>
    <link rel="stylesheet" th:href="@{/styles.css}"/>
</head>
<body>
<h1>Design your taco!</h1>
<img th:src="@{/images/TacoCloud.jpg}"/>

<form method="POST" th:object="${design}">
    <div class="grid">
        <div class="ingredient-group" id="wraps">
            <h3>Designate your wrap:</h3>
            <div th:each="ingredient:${wrap}">
                <input name="ingredients" type="checkbox" th:value="ingredient.id"/>
                <span th:text="ingredient.name">INGREDIENT</span>
            </div>
        </div>

        <div class="ingredient-group" id="proteins">
            <h3>Please choose the protein</h3>
            <div th:each="ingredient:${protein}">
                <input name="ingredients" type="checkbox" th:value="ingredient.id"/>
                <span th:text="ingredient.name">INGREDIENT</span>
            </div>
        </div>

        <div class="ingredient-group" id="cheeses">
            <h3>Please choose the cheese</h3>
            <div th:each="ingredient:${cheese}">
                <input name="ingredients" type="checkbox" th:value="ingredient.id"/>
                <span th:text="ingredient.name">INGREDIENT</span>
            </div>
        </div>

        <div class="ingredient-group" id="veggies">
            <h3>Please choose the veggies</h3>
            <div th:each="ingredient:${veggies}">
                <input name="ingredients" type="checkbox" th:value="ingredient.id"/>
                <span th:text="ingredient.name">INGREDIENT</span>
            </div>
        </div>

        <div class="ingredient-group" id="sauces">
            <h3>Please choose the sauces</h3>
            <div th:each="ingredient:${sauce}">
                <input name="ingredients" type="checkbox" th:value="ingredient.id"/>
                <span th:text="ingredient.name">INGREDIENT</span>
            </div>
        </div>

        <h3>Name your taco creation:</h3>
        <input type="text" th:field="*{name}"/>
        <br/>
        <button>Submit your taco</button>
    </div>
</form>

</body>
</html>
```
# Description
- Line 2 <!--suppress All --> is used to suppress the red line in the curly brace like ${wrap} etc.
- The method attribute in the <form> tag is POST as this is a form which should be passed to the backend after the checkboxes being selected.
- On the very last part of this code block, there is a statement: <input type="text" th:field="*{name}"/> We thus talk about the differences between the $ and *:
${} is used to get the KEY in the context.
*{} is used to get the value of some exact instance, which should be indicated in advance. As in this case the *{name} is the name of ${design} which is returned from the method showDesignForm(Model model).
Additionally, @{} operator is used to produce a CONTEXT-RELATIVE path to the static artifacts that they are referencing (Actually the /static directory)

28/07/2020
Also we can find there is a POST in the tag <form>.
It does not declare an action directly, but when the "submitt" button is pressed, the BROWSER will gather up all the data in the FORM and send it to the SERVER as an HTTP POST request to the SAME PATH from which the GET request displayed the form ("/design" path). Therefore, we need a controller handler method on receiving end of the POST request. That is, we need a method to handle POST request in DesignTacoController class. We use @PostMapping this time. The postDesign method is like the following:
```
@PostMapping
    public String processDesign(Taco design){
        System.out.println("I am in processing Design");
        log.info("Processing design"+design);
        return "redirect:/orders/current";
    }
```
When the GET method is called, a new Taco object is passed to the Thymeleaf template. After the form is submitted via the POST method, this object is passed to the backend, that is the processDesign () method. As the return statement is "redirect:/orders/current" , we actually pass a GET request. This request is handled by the following class (OrderController):
```
package com.example.tacocloud_1.Controllers;

import com.example.tacocloud_1.Tacos.Order;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;

/**
 * @author:Junlin Lu
 * @date:28/07/2020
 * @version:1.0.0
 * @description:
 */
@Slf4j
@Controller
@RequestMapping("/orders")
public class OrderController {

    @GetMapping("/current")
    public String orderForm(Model model){
        model.addAttribute("order",new Order());
        return "orderForm";
    }

    @PostMapping
    public String processOrder(Order order){
        log.info("Order submitted: " + order);
        return "redirect:/";
    }
}
```
Being redirected to the path "orders/current", via the GET method, a new Order object is constructed and put in the new key-value pair in Model with the key "order". Then direct to the Thymeleaf orderForm.html which is shown as the following:
```
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:th="http://www.thymeleaf.org"
      lang="en">
<head>
    <meta charset="UTF-8">
    <title>Taco Cloud</title>
    <link rel="stylesheet" th:href="@{/styles.css}" />
</head>
<body>
    <form method="POST" th:action="@{/orders}" th:object="${order}">
        <img th:src="@{/images/TacoCloud.jpg}"/>
        <a th:href="@{/design}" id="another">Design another Taco</a><br/>

        <div th:if="${#fields.errors()}">
            <span class="validationError">
                Please correct the information below and resubmit.
            </span>
        </div>

        <h3>Deliver my taco masterpiece to</h3>
        <label for="name">Name:</label>
        <input type="text" th:field="*{name}" id="name"/>
        <br/>

        <h3>Street</h3>
        <label for="street">Street:</label>
        <input type="text" th:field="*{street}" id="street"/>
        <br/>

        <h3>City</h3>
        <label for="city">City:</label>
        <input type="text" th:field="*{city}" id="city"/>
        <br/>

        <h3>State</h3>
        <label for="state">State:</label>
        <input type="text" th:field="*{state}" id="state"/>
        <br/>

        <h3>Zip</h3>
        <label for="zip">Zip:</label>
        <input type="text" th:field="*{zip}" id="zip"/>
        <br/>

        <h3>CCNumber</h3>
        <label for="ccNumber">CCNumber:</label>
        <input type="text" th:field="*{ccNumber}" id="ccNumber"/>
        <br/>

        <h3>CCExpiration</h3>
        <label for="ccExpiration">CCExpiration:</label>
        <input type="text" th:field="*{ccExpiration}" id="ccExpiration"/>
        <br/>

        <h3>CCCVV</h3>
        <label for="ccCvv">CCCVV:</label>
        <input type="text" th:field="*{ccCvv}" id="ccCvv"/>
        <br/>

        <input type="submit" value="Submit order"/>
    </form>
</body>
</html>
```
After being submitted, the POST method in order controller is invoked.
### 2.3 Validating form input
- Declare validation rules on the class that is to be validated: specifically, the Taco class.
- Specify that validation should be performed in the controller methods that require validation: specifically, the DesignTacoController's processDesign() method and OrderController's processOrder() method.
- Modify the form views to display validation errors.
# 2.3.1 Declaring validation rules
The Order class is modified as the following:
```
package com.example.tacocloud_1.Tacos;

import lombok.Data;
import org.hibernate.validator.constraints.CreditCardNumber;

import javax.validation.constraints.Digits;
import javax.validation.constraints.NotBlank;
import javax.validation.constraints.Pattern;

/**
 * @author:Junlin Lu
 * @date:28/07/2020
 * @version:1.0.0
 * @description:
 */
@Data
public class Order {
    @NotBlank(message="Name is required")
    private String name;
    @NotBlank(message="Street is required")
    private String street;
    @NotBlank(message = "City is required")
    private String city;
    @NotBlank(message = "State is required")
    private String state;
    @NotBlank(message = "Zip code is required")
    private String zip;
    @CreditCardNumber(message= "Not a valid credit card number")
    private String ccNumber;
    @Pattern(regexp = "^(0[1-9]|1[0-2])([\\/])([1-9][0-9])$")
    private String ccExpiration;
    @Digits(integer =3,fraction=0,message="Invalid CVV")
    private String ccCvv;
}
```
We have declared how a Taco and Order should be validated. Now, we need to revisit the controllers to specify the validation should be performed.
# 3 Working with Data
# 3.1 Reading and writing data with JDBC
Spring supports both JDBC and JPA. Spring JDBC support roots from JdbcTemplate class.
With JdbcTemplate, we can solely focus on the query of database and how to map the gotten data to some object.
# 3.1.1 Adapting the domain for persistence
Generally, we need a unique field to be the identifier of the object when do persisting objects to the database. Similarly, created time is always necessary as well. The two class Taco and Order is hence modified as the following:
Taco
```
package com.example.tacocloud_1.Tacos;

import lombok.Data;

import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;
import java.util.Date;
import java.util.List;

/**
 * @author:Junlin Lu
 * @date:17/06/2020
 * @version:1.0.0
 * @description:
 */
@Data
public class Taco {

    private Long id;
    private Date createdAt;
    @NotNull
    @Size(min=5,message="Name must be at least 5 characters long")
    private String name;

    @Size(min=1,message="You must choose at least one ingredient")
    private List<String> ingredients;
}

```
Order:
```
package com.example.tacocloud_1.Tacos;

import lombok.Data;
import org.hibernate.validator.constraints.CreditCardNumber;

import javax.validation.constraints.Digits;
import javax.validation.constraints.NotBlank;
import javax.validation.constraints.Pattern;
import java.util.Date;

/**
 * @author:Junlin Lu
 * @date:28/07/2020
 * @version:1.0.0
 * @description:
 */
@Data
public class Order {
    private long id;
    private Date placedAt;
    @NotBlank(message="Name is required")
    private String name;
    @NotBlank(message="Street is required")
    private String street;
    @NotBlank(message = "City is required")
    private String city;
    @NotBlank(message = "State is required")
    private String state;
    @NotBlank(message = "Zip code is required")
    private String zip;
    @CreditCardNumber(message= "Not a valid credit card number")
    private String ccNumber;
    @Pattern(regexp = "^(0[1-9]|1[0-2])([\\/])([1-9][0-9])$")
    private String ccExpiration;
    @Digits(integer =3,fraction=0,message="Invalid CVV")
    private String ccCvv;
}
```
Now the domain class is ready for being persisted.
# 3.1.2 Working with JdbcTemplate
Add the JdbcTemplate dependency with Maven:
```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```
And we are going to use an H2 embedded database, as the dependency:
```
<dependency>
	<groupId>com.h2database</groupId>
	<artifactId>h2</artifactId>
	<scope>runtime</scope>
</dependency>
```
The repository of Ingredient should perform these:
- fina all the ingredients and store them into a List
- find a single Ingredient by its id.
- Save an Ingredient object.
The interface of the Ingredient Repository
```
package com.example.tacocloud_1.data;
import com.example.tacocloud_1.Tacos.Ingredient;
/**
 * @author:Junlin Lu
 * @date:29/07/2020
 * @version:1.0.0
 * @description:
 */
public interface IngredientRepository {
    Iterable<Ingredient> findAll();
    Ingredient findOne(String id);
    Ingredient save(Ingredient ingredient);
}
```
The implement class JdbcIngredientRepository:
```
package com.example.tacocloud_1.data;
import com.example.tacocloud_1.Tacos.Ingredient;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;
import java.sql.ResultSet;
import java.sql.SQLException;

/**
 * @author:Junlin Lu
 * @date:29/07/2020
 * @version:1.0.0
 * @description:
 */
@Repository
public class JdbcIngredientRepository implements IngredientRepository{
    private JdbcTemplate jdbc;

    @Autowired
    public JdbcIngredientRepository(JdbcTemplate jdbc){
        this.jdbc=jdbc;
    }

    @Override
    public Iterable<Ingredient> findAll() {
        return jdbc.query("select id, name, type from Ingredient",
                this::mapRowToIngredient);
    }

    @Override
    public Ingredient findOne(String id) {
        return jdbc.queryForObject("select id, name, type from Ingredient where id=?",
                this::mapRowToIngredient,id);
    }

    @Override
    public Ingredient save(Ingredient ingredient) {
       jdbc.update("insert into Ingredient(id,name,type) value(?,?,?)",
                ingredient.getId(),
                ingredient.getName(),
                ingredient.getType().toString());
       return ingredient;
    }

    private Ingredient mapRowToIngredient(ResultSet rs, int rowNum) throws SQLException{
        return new Ingredient(
                rs.getString("id"),
                rs.getString("name"),
                Ingredient.Type.valueOf(rs.getString("type")));
    }
}
```
and the DesignTacoController class is modified as:
```
package com.example.tacocloud_1.Controllers;
import com.example.tacocloud_1.Tacos.Ingredient;
import com.example.tacocloud_1.Tacos.Taco;
import com.example.tacocloud_1.data.IngredientRepository;
import com.example.tacocloud_1.data.JdbcIngredientRepository;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.Errors;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import javax.validation.Valid;
import java.util.ArrayList;
import java.util.List;
import java.util.stream.Collectors;

/**
 * @author:Junlin Lu
 * @date:27/07/2020
 * @version:1.0.0
 * @description:
 */
@Slf4j
@Controller
@RequestMapping("/design")
public class DesignTacoController {
    private final IngredientRepository ingredientRepository;//why here we use an instance of an interface??

    @Autowired
    public DesignTacoController(IngredientRepository ingredientRepository){
        this.ingredientRepository=ingredientRepository;
    }

    @GetMapping
    public String showDesignForm(Model model){
        List<Ingredient> ingredients = new ArrayList<>();
        ingredientRepository.findAll().forEach(i->ingredients.add(i));

        Ingredient.Type[] types = Ingredient.Type.values();
        for(Ingredient.Type type:types){
            model.addAttribute(type.toString().toLowerCase(),filterByType(ingredients,type));
        }
        model.addAttribute("design",new Taco());

        return "design";
    }
    @PostMapping
    public String processDesign(@Valid Taco design, Errors errors){
        if(errors.hasErrors()){
            return "design";
        }
        log.info("Processing design"+design);
        return "redirect:/orders/current";
    }

    /**
     * Pass in the ingredients list and get the related ingredients of the passed in Type
     * Use Stream to filter the elements which has the same type and then collect them as a new List
     * and return.
     * @param ingredients
     * @param type
     * @return
     */
    private List<Ingredient> filterByType(List<Ingredient> ingredients, Ingredient.Type type){
        return ingredients.
                stream().
                filter(x->x.getType().equals(type)).
                collect(Collectors.toList());
    }
}
```
You may wonder why we @AutoWired an Interface rather than the implementing class to be the bean.
That is Spring will search the bean via the annotation @Repository and by type. 
So, what about if we had two implementing classes of one interface, and all of them can be recognized by Spring Auto-Scanning mechanism?? If you think there is some problem, then how to fix it??
- The Spring auto-scanning will automatically find the component, and according to "Interface-Oriented programming", we should @Autowired a Interface object, then Spring will find it for us(by @Component etc. and the keyword "implement"). @Autowired an interface reference is declaring we need a Java been which implemented this interfaace, by auto-scanning mechanism, Spring will automatically find the exact bean class (implementing the interface) and inject it to the context.
- But while there are two implementing classes of one interface, Spring will throw out the Exception, because @Autowired is to autowire by type. And the two implementing classes has the same interface type. Thus we need to to component by name, in this case. We have two annotation: @Resource and @Qualifier to inject beans by name. @Resource is used in the case where the Component is named in @Service(name="somename"), however @Qualifier shall take the class name.
You may find it strange that in the code block, the query() method (in findAll() )and queryObjcet() method (in findOne())of JdbcTemplate is written like this:
```
@Override
    public Iterable<Ingredient> findAll() {
        return jdbc.query("select id, name, type from Ingredient",
                this::mapRowToIngredient);
    }

    @Override
    public Ingredient findOne(String id) {
        return jdbc.queryForObject("select id, name, type from Ingredient where id=?",
                this::mapRowToIngredient,id);
    }
```
However, in the source code of query(), it is like this:
```
@Override
	public <T> List<T> query(String sql, RowMapper<T> rowMapper) throws DataAccessException {
		return result(query(sql, new RowMapperResultSetExtractor<>(rowMapper)));
	}
```
where, it seems that the second param of query() should be some object of RowMapper<T>. It seems not reasonable to put the method:
```
private Ingredient mapRowToIngredient(ResultSet rs, int rowNum) throws SQLException{
        return new Ingredient(
                rs.getString("id"),
                rs.getString("name"),
                Ingredient.Type.valueOf(rs.getString("type")));
    }
```
into the params. Isn't it?
Well, if we take a deeper look into the method, we can find that the "RowMapper" is an Functional Interface.
```
@FunctionalInterface
public interface RowMapper<T> {

	/**
	 * Implementations must implement this method to map each row of data
	 * in the ResultSet. This method should not call {@code next()} on
	 * the ResultSet; it is only supposed to map values of the current row.
	 * @param rs the ResultSet to map (pre-initialized for the current row)
	 * @param rowNum the number of the current row
	 * @return the result object for the current row (may be {@code null})
	 * @throws SQLException if an SQLException is encountered getting
	 * column values (that is, there's no need to catch SQLException)
	 */
	@Nullable
	T mapRow(ResultSet rs, int rowNum) throws SQLException;
}
```
So the program is error-free, and the generic type T here, is Ingredient which is the return type of "mapRowToIngredient" method.
07/29/2020










