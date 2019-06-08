# Basic Web Technologies A Developer Should Know to Design Modern Web APIs

## Duration

90 Minutes
	
## Part 1 - Basic HTTP

In this part, the attendants' knowledge of HTTP will be revised. It covers necessary elements of HTTP with extensive real world examples. We start off by illustrating how HTTP is used in the state of the art JAM stack application. Then we inpect parts of a request and response in details, which include HTTP Methods, Headers and MIME types. We conclude this part by comparing the classic stateful server rendered website design and the popular stateless API design for mobile and single page applications, in which we talked about session, cookies, authentication token management and RESTFul architectural style.
	
### Content	

- Overview of HTTP elements: clients, servers, proxies, requests and responses.
- Introduction to different MIME types and use cases. HTML, JSON, XML, File Upload, Image Rendering, etc…
- Introduction to important HTTP status codes and the best practice to choose which.
- Common request and response headers such as Content-Type, Accept, Accep-Language, Authorization, Date, Cache-Control and how to define customer headers such as X-Your-Header
- Architecture of a server rendered website. Sessions and cookies.
- Architecture of a stateless API server and it communication with the clients such as mobile and single page applications. 
- Introduction to RESTFul style.
	
## Part 2 - See how people are doing

After learning the basic, we take a step furthur seeing how credible companies like Facebook and Google design their APIs. How does the authentication work. Which HTTP methods, status code and MIME types are used in which situation. And how do they handle errors. We discussed pros and cons of each approaches. Aside from Google and Facebook, we also see how APIs are designed in some corporate we have worked with. At the end, we introduce a possibility of using some other approach such as HATEOAS (using Spring-Data-Rest) and RMI (in JavaEE).
	
### Content	

- Google APIs
- Facebook APIs & GraphQL
- Custom Apis in some corporate.
- Spring Data Rest HATEOAS
- RMI
	
## Part 3 - Common API Design Patterns

In this part, we list problems common most APIs and discussed how to design the API to solve the problems. These solutions will show the proper uses of HTTP elements in different situations and can be archetypes to be applied with real world problems.
	
### Content	

- Search, filter, sort & pagination.
- Retrieve an item.
- Delete an item.
- Soft-delete an item.
- Update an item.
- Create an item.
- Bulk create items.
- Bulk update items.
- Bulk delete items.
- Retrieve an item in different formats and languages. (content negotiation)
- Access public resources. (static images, CSS, JS)
	
## Part 4 - More Advanced Topics (เผื่อเวลาเหลือ)

In this part, we focused on the more advanced topics.
	
### Content	

- HTTP Caching
- Load Balance
- API Gateway
- Websocket
- Webhook & Push Notification
- Video Streamming
	
## Part 5 - Design Your Own APIs

We conclude the session with a competition. Given KKP shopping cart UI flow, each attendants will be asked to redesign the set of APIs for customers to add products to shopping carts and create orders. The winner is the one that can design the most simple API. The result is to emphasize that one of the most important goal of API design is to make it simple, which can be achive by hiding unecessary information from the client as much as possible.
