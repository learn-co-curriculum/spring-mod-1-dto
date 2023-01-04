# Data Transfer Objects

## Learning Goals

- Explain the significance of the DTO design pattern.
- Mention how to hide a field in a JSON response.

## What is a Data Transfer Object?

A **data transfer object (DTO)** is an object that encapsulates data in order to
carry it between processes. This is a design pattern that reduces the amount of
calls and processes back and forth to the server by batching up multiple parameters
in a single call.

As we may recall from the "What is Spring MVC?" lesson, Spring uses a
`DispatcherServlet` to process user requests in HTTP. The `DispatcherServlet`
helps orchestrate and communicate data from one server to a client (or another
server). There are multiple processes that occur to transfer data to and from.
Using DTOs to carry the data makes it easy to pass multiple attributes in one
structure when these processes, or instances of an executing program, are being
run.

Consider the following class:

```java
package com.example.springwebdemo.dto;

public class FootballTeamDTO {
    private String teamName;
    private int wins;
    private int losses;
    private boolean currentSuperBowlChampion;

    public String getTeamName() {
        return teamName;
    }

    public void setTeamName(String teamName) {
        this.teamName = teamName;
    }

    public int getWins() {
        return wins;
    }

    public void setWins(int wins) {
        this.wins = wins;
    }

    public int getLosses() {
        return losses;
    }

    public void setLosses(int losses) {
        this.losses = losses;
    }

    public boolean getCurrentSuperBowlChampion() {
        return currentSuperBowlChampion;
    }

    public void setCurrentSuperBowlChampion(boolean currentSuperBowlChampion) {
        this.currentSuperBowlChampion = currentSuperBowlChampion;
    }
}
```

A DTO is a simple POJO or "Plain Old Java Object". It contains only accessor and
mutator methods with no additional business logic. DTOs exist to store and deliver
only the necessary data that may need to be provided to another server or user.
It should also be noted that most DTOs have "DTO" attached to its class name to
let other developers know about the purpose of this class.

## Using Data Transfer Objects

The DTO acts as a model in the MVC model. This will provide us an object to
transfer back and forth from an API client. So how can we use a DTO in a Spring
MVC? Let's use the `FootballTeamDTO` from above. We'll create a controller and
service class to interact with this type of data and turn this into a REST API.

```java
// Controller

package com.example.springwebdemo.controller;

import com.example.springwebdemo.dto.FootballTeamDTO;
import com.example.springwebdemo.service.FootballService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@RestController
public class FootballController {

    private final FootballService footballService;

    @Autowired
    public FootballController(FootballService footballService) {
        this.footballService = footballService;
    }

    @PostMapping("/football-team")
    public String addFootballTeam(@RequestBody FootballTeamDTO footballTeam) {
        return footballService.addFootballTeam(footballTeam);
    }

    @GetMapping("/football-team/{teamName}")
    public FootballTeamDTO getFootballTeam(@PathVariable String teamName) {
        return footballService.getFootballTeam(teamName);
    }
}
```

```java
// Service

package com.example.springwebdemo.service;

import com.example.springwebdemo.dto.FootballTeamDTO;
import org.springframework.stereotype.Service;

import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

@Service
public class FootballService {
    List<FootballTeamDTO> footballTeams;

    public FootballService() {
        footballTeams = new ArrayList<>();
    }

    public String addFootballTeam(FootballTeamDTO footballTeam) {
        footballTeams.add(footballTeam);
        return String.format("%s has been added!", footballTeam.getTeamName());
    }

    public FootballTeamDTO getFootballTeam(String teamName) {
        Optional<FootballTeamDTO> footballTeamDTO = footballTeams.stream()
                .filter(footballTeam -> footballTeam.getTeamName().equals(teamName))
                .findAny();
        
        // If the team does not exist, throw a NoSuchElementException
        return footballTeamDTO.orElseThrow();
    }
}
```

Let's break down this code a little bit.

We want to be able to accomplish two main goals with this REST API:

- Add a football team.
- Get a football team by its team name.

We'll use a `List` data structure within the service class to keep track of the
football teams. This will also make it easy to add teams and get the teams from
the list.

But let's look at the `FootballController` class, as we have introduced some new
annotations.

### @PostMapping

If we want to add a new football team to our application, then we need to use the
`@PostMapping` annotation. The `@PostMapping` in shorthand for the
`@RequestMapping(method=RequestMethod.POST)` annotation, which will handle mapping
HTTP POST requests.

### @RequestBody

This annotation allows the method to accept JSON formatted client data and
convert it to a Java object. For example, the JSON object mentioned above would
be mapped to a `FootballTeamDTO` object with the attributes set to the values in
the JSON.

### Get Method

We'll use the `@GetMapping` and `@PathVariable` annotation that we used before to
have our application process the GET request. The only difference here is we'll
return a DTO instead of a `String`. When we do this, it will serialize the data
into a JSON object to return.

## Passing in a JSON

Let's run the application and see what happens. We will first need to add a
football team to the application.

Open up Postman and click the "Headers" tab. Make sure that the headers are not
hidden. If they are, click on the "hidden" link to un-hide the headers.

![headers-hidden](https://curriculum-content.s3.amazonaws.com/spring-mod-1/dto/headers-hidden.png)

Then add "Content-Type":"application/json" as shown in the screenshot below:

![Content-Type-Headers](https://curriculum-content.s3.amazonaws.com/spring-mod-1/dto/api-headers-content-type.png)

Then we can add a football team in JSON format. The `FootballTeamDTO` has the
fields `teamName`, `wins`, `losses`, and `currentSuperBowlChampion`. So we will
have to make sure to include those in the JSON we want to send to our application.
We will add the following JSON to our application:

```json
{
    "teamName":"Dallas-Cowboys", 
    "wins":7, 
    "losses":3, 
    "currentSuperBowlChampion":0
}
```

Note: The 0 value for `currentSuperBowlChampion` represents `false` where 1
would represent `true`. This is a shorthand for true/false and is how Java
stores `boolean` values.

In Postman, navigate to the "Body" tab, choose the "raw" radio button, and then
copy-paste the JSON in like so:

![JSON-response-body](https://curriculum-content.s3.amazonaws.com/spring-mod-1/dto/postman-post-football-team.png)

This will be the response body that we send to our REST API! For the request URL,
make sure "POST" is selected and that the URL is
"http://localhost:8080/football-team". Then click "Send" to post.

![Post-Football-Team](https://curriculum-content.s3.amazonaws.com/spring-mod-1/dto/postman-add-football-team.png)

So what happened? When we passed the request body as part of the POST request, the
controller took JSON and converted it into a data transfer object, a Java object,
that it could then parse. The DTO was then sent off to the service class in order
to perform some kind of business logic. It returned a success message to let us
know that it successfully added the football team to the list in the service
class.

## Getting a JSON

Now that we have successfully sent a POST request to the application, let's see
if we can get that same data back that we posted. We'll send a GET request now
using the path "http://localhost:8080/football-team/Dallas-Cowboys", as specified
in the `@GetMapping` annotation in the `FootballController` class.

![Get-Football-Team](https://curriculum-content.s3.amazonaws.com/spring-mod-1/dto/postman-get-footbal-team.png)

When we send this request to the application, notice the response body we receive
back in response - it's a JSON containing the same information we had passed to
the API.

So what happened here? When we passed in the request with the "Dallas-Cowboys" as
the dynamic path variable, it will search the `List` in the `FootballService`
class to see if a football team with that team name exists. Since it does, it will
return the `FootballTeamDTO` in a serialized JSON format.

## Resources

- [Baeldung Java DTO Pattern](https://www.baeldung.com/java-dto-pattern)
- [PostMapping Annotation](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/PostMapping.html)
- [RequestBody Annotation](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/RequestBody.html)
