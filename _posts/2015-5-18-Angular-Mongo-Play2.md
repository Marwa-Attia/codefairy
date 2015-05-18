---
layout: post
title: Simple web app with AngularJS + Play + MongoDB part2
---

In [part1](http://marwa-attia.github.io/codefairy/Angular-Mongo-Play1/) of this post we explained how to configure MongoDB, AngularJS and Play framework to work together.

In part2 we will continue in developing our simple application.

First, we need to create the model.The model is so simple, a Customer class, Address class and an Account class.
We will start be the **BaseEntity.java** which is the parent class for all these entities:
```java
package models.entity;

import java.util.ArrayList;
import java.util.List;

import org.bson.types.ObjectId;
import org.mongodb.morphia.Key;
import org.mongodb.morphia.annotations.Id;
import org.mongodb.morphia.annotations.Property;
import org.mongodb.morphia.annotations.Version;

import controllers.MorphiaObject;

public abstract class BaseEntity {

	@Id
	@Property("id")
	protected ObjectId id;

	@Version
	@Property("version")
	private Long version;

	public BaseEntity() {
		super();
	}

	public ObjectId getId() {
		return id;
	}

	public void setId(ObjectId id) {
		this.id = id;
	}

	public Long getVersion() {
		return version;
	}

	public void setVersion(Long version) {
		this.version = version;
	}

	public List<? extends BaseEntity> all() {
		if (MorphiaObject.datastore != null) {
			return MorphiaObject.datastore.find(this.getClass()).asList();
		} else {
			return new ArrayList<BaseEntity>();
		}
	}

	public Key<BaseEntity> create(BaseEntity group) {
		return MorphiaObject.datastore.save(group);
	}

}

```

 **Customer.Java** class
 
 ```java
 
 package models.entity;

import org.mongodb.morphia.annotations.Embedded;
import org.mongodb.morphia.annotations.Entity;

@Entity
public class Customer extends BaseEntity {

	public Customer() {
	}

	private String name;
	@Embedded
	private Account account;
	@Embedded
	private Address address;

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public Account getAccount() {
		return account;
	}

	public void setAccount(Account accounts) {
		this.account = accounts;
	}

	public Address getAddress() {
		return address;
	}

	public void setAddress(Address address) {
		this.address = address;
	}

}
```
The _Customer_ entity has two embedded one-to-one relationship with Account and Address documents.

**Account.java**

```java

package models.entity;

import org.mongodb.morphia.annotations.Embedded;

@Embedded
public class Account  {

	public Account() {
		
	}

	private String name;
 
    public String getName() {
        return name;
    }
 
    public void setName(String name) {
        this.name = name;
    }
 
}
```

**Address.java**

```java 
package models.entity;

import org.mongodb.morphia.annotations.Embedded;


@Embedded
public class Address {
 
    private String number;
    private String street;
    private String town;
    private String postcode;
 
    public String getNumber() {
        return number;
    }
 
    public void setNumber(String number) {
        this.number = number;
    }
 
    public String getStreet() {
        return street;
    }
 
    public void setStreet(String street) {
        this.street = street;
    }
 
    public String getTown() {
        return town;
    }
 
    public void setTown(String town) {
        this.town = town;
    }
 
    public String getPostcode() {
        return postcode;
    }
 
    public void setPostcode(String postcode) {
        this.postcode = postcode;
    }
 
}
```

Now that our model is ready, we move on to building the controller for our simple application.

Let's edit our controller **Application.java** and add an action method to list all the customers as follows:

```java

  public static Result allCustomers() {
		JsonNode json = null;
		Customer customer = new Customer();
		json = Json.toJson(customer.all());
		return ok(json).as("application/json");
	}
```

This action method will query all the customers in mongoDb and serve them to the front end as a Json object which we will use AngularJS to display later on.

Now we add another action to add a new customer:

```java

@BodyParser.Of(BodyParser.Json.class)
	public static Result addCustomer() {
		  JsonNode json = request().body().asJson();
		  String name = json.findPath("name").textValue();
		  String accountName = json.findPath("account").textValue();
		  String addressNumber = json.findPath("number").textValue();
		  String addressStreet = json.findPath("street").textValue();
		  String addressTown= json.findPath("town").textValue();
		  String addressPostcode= json.findPath("postcode").textValue();
		
			  Address address = new Address();
				address.setNumber(addressNumber);
				address.setStreet(addressStreet);
				address.setTown(addressTown);
				address.setPostcode(addressPostcode);

				Account account = new Account();

				account.setName(accountName);


				Customer customer = new Customer();
				customer.setAddress(address);
				customer.setName(name);
				customer.setAccount(account);
				Key<BaseEntity> savedCustomer = customer.create(customer);

				JsonNode result = Json.toJson(customer.all());
				return ok(result).as("application/json");
		
	}
```

This action method will receive the customer details entered by the user from the front end in json format and use it to create a new customer object and store it in the database, and then query the customer list again to refresh the view with the newly added customer.

Action methods are entry points to a play-powered application, but in order for play to be able to route requests to the right action method, we have to define that in play routers first, so open your **_routers_** file, which you will find in the _conf_ folder in your application's root folder, and add these two lines:
		
		GET     /allCustomers               controllers.Application.allCustomers()
		POST     /addCustomer               controllers.Application.addCustomer()
		
Now we can start working on the view using AngularJS and jquery and bootstrap. First download [Jquery](https://jquery.com/download/), now we need to include the all our javascript file and css file in our __main.scala.html__ page:

```html

<link rel="stylesheet" href="//netdna.bootstrapcdn.com/bootstrap/3.0.0/css/bootstrap.min.css" />
<script src="@routes.Assets.at("javascripts/jquery.js")" type="text/javascript"></script>
<script src="//netdna.bootstrapcdn.com/bootstrap/3.1.1/js/bootstrap.min.js"></script>
<script src="@routes.Assets.at("javascripts/angular.js")" type="text/javascript"></script>

```
In Angular you can create a module for your web app; a module is sort of a container for the different parts of your app – controllers, services, filters, directives, etc.
so we will create our module in a separate js file called __app.js__ :

```javascript
var webApp = angular.module('webApp', [
  'ngRoute',
  'customerControllers'
]);

webApp.config(['$routeProvider',
  function($routeProvider) {
    $routeProvider.
      when('/addCustomer', {
        templateUrl: 'assets/partials/addCustomer.html',
        controller: 'AddCustomerCtrl'
      }).
      otherwise({
        redirectTo: '/'
      });
  }]);
  ```
 Here we defined a new module with 2 dependencies on  __ntRoute__ module which is an Angular module used for providing routing for Angular we apps, and __customerControllers__ which is the module that holds the controller used in our simple app. This module is called  __webApp__, and this is the name that we will use to bind the _ng-app_ directive to in our template page, this is what bootstraps the app using our __webApp__ module.
 
We also has defined some routing for our app; and this is why we added the dependency on the __ntRoute__ module.
Now we move on to implementing our own module; we will do that by creating a new controller __controller.js__.

As explained in [Angular Documentation](https://docs.angularjs.org/guide); a Controller is a JavaScript **constructor function** that is used to augment the Angular Scope.
When a Controller is attached to the DOM via the __ng-controller__ directive, Angular will instantiate a new Controller object, using the specified Controller's constructor function. 
A new child scope will be available as an _injectable_ parameter to the Controller's constructor function as $scope.


```javascript
var phonecatControllers = angular.module('customerControllers', []);

phonecatControllers.controller('CustomerListCtrl', function ($scope, $http) {
  $http.get('/allCustomers').success(function(data) {
	  $scope.users = data; 
  });
  $scope.submitMyForm=function(){
      /* while compiling form , angular created this object*/
      var formData=$scope.cust; 
      /* post to server*/
      $http.post('/addCustomer', formData).success(function(data, status, headers, config) {
    	  $scope.users.push( formData);
    	  $scope.name='';
    	  $scope.account='';
    	  $scope.number='';
    	  $scope.street='';
    	  $scope.town='';
    	  $scope.postcode='';
    	  }).
    	  error(function(data, status, headers, config) {
    	    // called asynchronously if an error occurs
    	    // or server returns response with an error status.
    	  });
     
  }
});

```

This controller makes an http ajax request to get all customers from our app. The data retrieved from the ajax call is assigned to a model called users which we will use to display the customers in a datatable as shown:

```html

<body ng-app="webApp">
	<div ng-controller="CustomerListCtrl">

		<form class="form-horizontal" ng-submit="submitMyForm();">
			<table>
				<tr>
					<td><label class="control-label">Name</label></td>
					<td><input name="name" ng-model="cust.name" class="form-control"></td>
				</tr>
				<tr>
					<td><label class="control-label">Account</label></td>
					<td><input name="account" ng-model="cust.account.name" class="form-control"></td>

				</tr>
				<tr>
					<td><label class="control-label">Building Number</label></td>
					<td><input name="number" ng-model="cust.address.number"	class="form-control"></td>
				</tr>
				<tr>
					<td><label class="control-label">Street</label></td>
					<td><input name="street" ng-model="cust.address.street" class="form-control"></td>
				</tr>
				<tr>
					<td><label class="control-label">Town</label></td>
					<td><input name="town" ng-model="cust.address.town" class="form-control"></td>
				</tr>
				<tr>
					<td><label class="control-label">Post-code</label></td>
					<td><input name="postcode" ng-model="cust.address.postcode" class="form-control"></td>
				</tr>
				<tr>
					<td><input type="submit" value="Save" class="btn btn-primary"></td>
				</tr>
			</table>
		</form>
		<br>
	
		<br>
		<table class="table">
			<tr>
				<th>Name</th>
				<th>Account</th>
				<th>Building Number</th>
				<th>Street</th>
				<th>Town</th>
				<th>Post-Code</th>
			</tr>
			<tr ng-repeat="user in users">
				<td data-title="'Name'">{{user.name}}</td>
				<td data-title="'Account'">{{user.account.name}}</td>
				<td data-title="'Building Number'">{{user.address.number}}</td>
				<td data-title="'Street'">{{user.address.street}}</td>
				<td data-title="'Town'">{{user.address.town}}</td>
				<td data-title="'Post-Code'">{{user.address.postcode}}</td>
			</tr>
		</table>
	</div>
</body>
```

Now let's run the application using the following command to start mongoDB first:
	
	[mongo-home]/bin/mongod --config [mongo-home]\mongo.config

Then we start play using this:
	
	$ cd [path-to-app]
	[VanillaJava] $ activator -jvm-debug 9999 ~run 

Open you browser and write [localhost:9000](localhost:9000) in the address.

You should now see something like this in your browser.


	



 
