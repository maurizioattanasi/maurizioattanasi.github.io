---
layout: post
title:  "Jekyll Contact Form with a .NET Core Microservice"
tags:
-  Blog
-  Markdown
-  jekyll
-  .NET Core
-  microservices
-  Domain Driven Design
-  CQRS
author: Maurizio Attanasi
---

If you have a static web site hosted on services like [GitHub Pages](https://pages.github.com/) or [GitLab Pages](https://docs.gitlab.com/ee/user/project/pages/), there are many chances that you would like to use a contact form to have some feedback from your site's guests or just to get in touch with them.
There are many options to accomplish the same result, many of them paid, some with a free but feature-limited plan, and others that need a little bit of effort to integrate with your contact form.

Even if the project is very simple, we'll follow of the rules of Domain-Driven Design (DDD) Microservices following Microsoft's guidelines on the subject [Design a DDD-oriented microservice](https://docs.microsoft.com/it-it/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/ddd-oriented-microservice).

## Solution structure

The solution is made of three layers,

![architecture](/assets/images/architecture.jpg)

each one implemented as a .NET Core project.

![DDD - layers of the microservices](/assets/images/ddd-layers.png)

### Application layer

The **application layer** is implemented as a .NET Core 3.1 Web API. The main responsibilities of this module are:

- ASP.NET Web API;
- Network access to microservice;
- API contract/implementation;
- Commands and command handlers;

### Domain layer

The **domain layer** is a .NET Core dll responsible for the implementation of the DDD patterns:

- holding the domain model (a set of *POCO* classes);
- declaring the **Repository Pattern** contracts/interfaces for the aforementioned POCO classes;
- introducing some data validation for the models;
 
#### Domain Entities/Aggregates

As said in the introduction, this is a very simple project requiring only two entities his folder/namespace to implement the business logic of the application:

1. `Account` is the owner entity of the messages;
2. `Message` is the model of a single message;

#### Repository interfaces

This folder/namespace contains all the interfaces/contracts needed to implement a [Repository Pattern](https://docs.microsoft.com/it-it/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design) that will be the intermediary between the domain model layers and data mapping.

### Infrastructure layer

This project contains:

- the implementation of the business logic of the backend;
- the implementation of the interfaces of the [Repository Pattern](https://docs.microsoft.com/it-it/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design) exposed by the domain project;
- the data persistence layer;

#### Entity Framework Core

The OR/M of choice for this project is [Entity Framework Core](https://docs.microsoft.com/it-it/ef/core/), and the approach in the early stages of development of the project is [Code First](https://docs.microsoft.com/it-it/aspnet/core/data/ef-mvc/intro?view=aspnetcore-3.1) but it may change if needed.

Using EF Core CLI, the creation of the database can be done using the following commands

```cmd
maurizioattanasi$ dotnet ef migrations add initial --startup-project ../ATech.ContactFormServer.Api
```

```cmd
maurizioattanasi$ dotnet ef database update --startup-project ../ATech.ContactFormServer.Api
```

## Toolbox

The project itself, as said before, is very simple, and its implementation is only one of the possible flavors of the Domain-Driven Design microservices. A few words more on the tools used in the project to obtain the basic functionalities needed in a production-grade application.

- [Serilog](https://serilog.net): Diagnostic logging of the application is handled by this package that has many powerful options to accomplish this feature;
- [MediatR](https://github.com/jbogard/MediatR): Is used to implement the **Mediator Pattern** in the application controller and a very enlighting article on the matter is [Fat Controller CQRS Diet: Simple Command](https://codeopinion.com/fat-controller-cqrs-diet-simple-command/) written by Derek Comartin;
- [SQLite](https://www.sqlite.org/index.html): Data persistence (for demo purpose only!) is delegated to the famous SQL file database engine;

## CI/CD

A CI/CD workflow is implemented with [GitHub Actions](https://github.com/features/actions) that will deploy the application on [Microsoft Azure](https://azure.microsoft.com/it-it/).

```yaml
# Docs for the Azure Web Apps Deploy action: https://github.com/Azure/webapps-deploy
# More GitHub Actions for Azure: https://github.com/Azure/actions

name: Build and deploy ASP.Net Core app to Azure Web App - atechcontactformserver

on:
 push:
 branches:
 - master

jobs:
 build-and-deploy:
 runs-on: ubuntu-latest

 steps:
 - uses: actions/checkout@master

 - name: Set up .NET Core
 uses: actions/setup-dotnet@v1
 with:
 dotnet-version: '3.1.102'

 - name: Build with dotnet
 run: dotnet build --configuration Release

 - name: dotnet publish
 run: dotnet publish -c Release -o ${{env.DOTNET_ROOT}}/myapp

 - name: Deploy to Azure Web App
 uses: azure/webapps-deploy@v2
 with:
 app-name: 'atechcontactformserver'
 slot-name: 'production'
 publish-profile: ${{ secrets.AzureAppService_PublishProfile_3a651e8c1e9a42a5bd1996554646a4bc }}
 package: ${{env.DOTNET_ROOT}}/myapp
```

## Usage example

The API can be used in a SPA application as shown in the following code snippet.

```javascript
$(function () {

    $("#contactForm input,#contactForm textarea").jqBootstrapValidation({
      preventSubmit: true,
      submitError: function ($form, event, errors) {
        // additional error messages or events
      },
      submitSuccess: function ($form, event) {
        event.preventDefault(); // prevent default submit behaviour
        // get values from FORM
        var name = $("input#6E616D65").val();
        var email = $("input#656D61696C").val();
        var phone = $("input#70686F6E65").val();
        var message = $("textarea#6D657373616765").val();

        var _name = $("input#name").val();
        var _email = $("input#email").val();
        var firstName = name; // For Success/Failure Message
        // Check for white space in name for Success/Fail message
        var data = {
          name: name,
          phone: phone,
          email: email,
          message: message,
          honeypot: $.trim(_name) + $.trim(_email)
        };
        if (firstName.indexOf(' ') >= 0) {
          firstName = name.split(' ').slice(0, -1).join(' ');
        }
        $this = $("#sendMessageButton");
        $this.prop("disabled", true); // Disable submit button until AJAX call is complete to prevent duplicate messages
        $.ajax({
          headers: {
            'Accept': 'application/json',
            'Content-Type': 'application/json'
          },
          url: "https://atechcontactformserver.azurewebsites.net/Message/Submit/{{ site.form_id }}",
          type: "POST",
          data: JSON.stringify(data),
          'dataType': 'json',
          cache: false,
          success: function () {
            console.log(JSON.stringify(data));
            // Success message
            $('#success').html("<div class='alert alert-success'>");
            $('#success > .alert-success').html("<button type='button' class='close' data-dismiss='alert' aria-hidden='true'>&times;").append("</button>");
            $('#success > .alert-success').append("<strong>Your message has been sent. </strong>");
            $('#success > .alert-success').append('</div>');
            //clear all fields
            $('#contactForm').trigger("reset");
          },
          error: function () {
            // Fail message
            $('#success').html("<div class='alert alert-danger'>");
            $('#success > .alert-danger').html("<button type='button' class='close' data-dismiss='alert' aria-hidden='true'>&times;").append("</button>");
            $('#success > .alert-danger').append($("<strong>").text("Sorry " + firstName + ", it seems that my mail server is not responding. Please try again later!"));
            $('#success > .alert-danger').append('</div>');
            //clear all fields
            $('#contactForm').trigger("reset");
          },
          complete: function () {
            setTimeout(function () {
              $this.prop("disabled", false); // Re-enable submit button when AJAX call is complete
            }, 1000);
          }
        });
      },
      filter: function () {
        return $(this).is(":visible");
      }
    });

    $("a[data-toggle=\"tab\"]").click(function (e) {
      e.preventDefault();
      $(this).tab("show");
    });
  });

  /*When clicking on Full hide fail/success boxes */
  $('#name').focus(function () {
    $('#success').html('');
  });
```

The entire project is available on [GitHub](https://github.com/maurizioattanasi/ATech.ContactFormServer).
 
Enjoy

---
<br/>


![Stay Safe Wear a Mask](/assets/images/keep-calm-wear-mask-red-small.jpg)
