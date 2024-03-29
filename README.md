## E-commerce API with AI assistant

E-commerce API is a Java-based, server-side application created to establish communication with client and database.   
The AI assistant also known as ChatGPT version 3.5 Turbo was used.     
The version can be changed as required. The AI assistant function requires the correct API key provided by OpenAI.    
It allows the user to obtain general information about the shop, contact and specific information about available products.     
Multiplatform e-commerce application, encompassing a wide range of functionalities to meet the diverse needs of online retail.    
The application uses a MySQL database to store users and orders information.    
Containerisation with Docker allows the API to run on different platforms.

## AI assistant example usage

API allows to send such message for this specific endpoint:    
| HTTP method | endpoint | description | request type | response type |
| -------------- | -------------- | -------------- | -------------- | -------------- |
| :yellow_circle: POST | /ai/askGPT | Ask the AI assistant a question | GPTRequest | String |

where request looks like that:
```json
{
    "requestType": "give advice",
    "clientRequest": "Do you have any laptops here?"
}
```
As the result we receive the following message from ChatGPT:
```
Yes, we do have laptops available in our online shop. One of our featured laptops is the "Laptop XYZ," priced at $1299.99. It's a high-performance laptop with advanced features. We currently have 10 units in stock. If you're interested, you can find more details and place an order on our website. If you have any other questions or need further assistance, feel free to ask.
```
As you can see, AI assistant have access to information about actual shop inventory.     
ChatBot has been configured early on to respond according to a certain rules.    
These rules can be changed by admin via specific endpoints.    

## Deploy

All in one deploy copy-paste commands for windows and linux systems.

Windows:
```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\run_docker_commands.ps1
```
Linux:
```console
chmod +x run_docker_commands.sh
./run_docker_commands.sh
```

## Docker

Follow these commands if u want to make containers manually and adjust them to your prorities.

### Database

The Dockerfile will provide containerisation and initialisation of the MySQL database.  
There is a set of commands to get to the same point when starting the application.  
Firstly make sure you are in main project directory.   
Now u can execute building process:   
```console
docker build -t mysqlshopapi:latest -f Docker_Database/Dockerfile .
```
I chose port 3307 because the standard 3306 port for MySQL is occupied by other container. It is up to you.    
You can then run the container with the default password or you can change it (don't forget to change it in the project properties too):
```console
docker run --name shopAPIDatabaseContainer -e MYSQL_ROOT_PASSWORD=sebastian -d -p 3307:3306 mysqlshopapi:latest
```
Now the MySQL container should run properly.

### Server

This Dockerfile will provide containerisation and initialisation of the Java application.  
There is a set of commands to get to the same point when starting the application.  
Firstly make sure you are in project directory.   
Now u can execute building process:   
```console
docker build -t javashopapi:latest -f Docker_Server/Dockerfile .
```
I chose port 8081 for Java container, it is up to you.    
You can then run the container:
```console
docker run --name shopAPIServerContainer -d -p 8081:8081 javashopapi:latest
```
Now the Java container should run properly.

## Initialize development stage

Once u are in main project directory this time.   
Firstly install all dependencies:
```console
mvn clean install
```
Then run application:
```console
mvn spring-boot:run
```
Now everything should be set up.

## Endpoints

Rest API embraces users and orders management issues.    
Some of the options implemented are creating account, changing user's data, orders registration.    
Rest endpoints for client only:    

| HTTP method | endpoint | description | request type | response type |
| -------------- | -------------- | -------------- | -------------- | -------------- |
| :green_circle: GET | /products | Retrieve a list of products | - | List&lt;Product&gt; |
| :green_circle: GET | /products/{id} | Get details for a specific product | - | Product |
| :green_circle: GET | /cart?userID={userID} | View the current contents of the shopping cart | - | Cart |
| :yellow_circle: POST | /cart/add | Add a product to the shopping cart | Cart | String |
| :yellow_circle: POST | /login | Authenticate a user | Credentials | String |
| :yellow_circle: POST | /register | Register a new user | User | String |
| :green_circle: GET | /orders/past?userID={userID} | View a list of past orders | - | List&lt;Order&gt; |
| :yellow_circle: POST | /orders/add | Add all user's cart products to the shopping orders | OrderRequest | Order |
| :green_circle: GET | /orders/details?orderID={orderID} | Retrieve details of a specific order | - | List&lt;OrderedProduct&gt; |
| :green_circle: GET | /search?searchQuery={searchQuery} | Search for products based on user input | - | List&lt;Product&gt; |
| :green_circle: GET | /categories | Retrieve a list of product categories | - | List&lt;Category&gt; |
| :green_circle: GET | /profile/info?userID={userID} | View user profile information | - | User |
| :yellow_circle: POST | /profile/update | Update user profile information | User | String |
| :green_circle: GET | /products/{reviewID}/reviews | Get product reviews | - | List&lt;Review&gt; |
| :yellow_circle: POST | /products/{reviewID}/reviews/add | Add a review for a product | Review | String |
| :yellow_circle: POST | /ai/askGPT | Ask the AI assistant a question | GPTRequest | String |

Rest endpoints for admin only:    

| HTTP method | endpoint | description | request type | response type |
| -------------- | -------------- | -------------- | -------------- | -------------- |
| :yellow_circle: POST | /admin/products/add | Add a new product to the catalog | Product | String |
| :red_circle: DELETE | /admin/products/delete/{productId} | Delete a product from the catalog | - | String |
| :purple_circle: PATCH | /admin/products/update/{productId} | Update product details | Product | String |
| :yellow_circle: POST | /admin/orders/fulfill/{orderId} | Mark an order as fulfilled | - | String |
| :yellow_circle: POST | /admin/users/update/{userId} | Update user information | User | String |
| :green_circle: GET | /admin/reports/sales | Retrieve sales report | - | List&lt;Product&gt; |
| :green_circle: GET | /admin/reports/inventory | Retrieve inventory report | - | List&lt;Product&gt; |

## Database

A MySQL database was used to store users and orders information.  
The entire database is containerised using Docker.  
The JDBC interface has been used to create a connection to the database.

### Table: shopAPI.users

| Column Name   | Data Type             | Constraints     |
|---------------|-----------------------|-----------------|
| userID        | VARCHAR(36)           | PRIMARY KEY     |
| name          | VARCHAR(255) NOT NULL |                 |
| email         | VARCHAR(255) NOT NULL UNIQUE |          |
| password      | VARCHAR(255) NOT NULL |                 |
| phoneNum      | INT NOT NULL          |                 |
| role          | VARCHAR(50) NOT NULL  |                 |
| accCreated    | DATETIME NOT NULL      |                 |

### Table: shopAPI.products

| Column Name      | Data Type                | Constraints        |
|------------------|--------------------------|--------------------|
| productID        | VARCHAR(36)              | PRIMARY KEY        |
| productName      | VARCHAR(255) NOT NULL    |                    |
| price            | DECIMAL(10, 2) NOT NULL  |                    |
| description      | VARCHAR(500) NOT NULL    |                    |
| category         | VARCHAR(50) NOT NULL     |                    |
| availableQuantity| INT NOT NULL             |                    |
| imagePath        | VARCHAR(255)             |                    |


### Table: shopAPI.orders

| Column Name        | Data Type                | Constraints                  |
|--------------------|--------------------------|------------------------------|
| userID             | VARCHAR(36)              | FOREIGN KEY (userId)| REFERENCES shopAPI.users(userID) |
| orderID            | VARCHAR(36)              | PRIMARY KEY                  |
| orderType          | VARCHAR(50) NOT NULL     |                              |
| additionalInfo     | VARCHAR(255)             |                              |
| registrationTime   | DATETIME                 |                              |

### Table: shopAPI.orderedProducts

| Column Name   | Data Type                | Constraints                             |
|---------------|--------------------------|-----------------------------------------|
| orderID       | VARCHAR(36)              | FOREIGN KEY (orderID) REFERENCES shopAPI.orders(orderID) |
| productID     | VARCHAR(36)              | FOREIGN KEY (productID) REFERENCES shopAPI.products(productID) |
| productName   | VARCHAR(255) NOT NULL    |                                         |
| price         | DECIMAL(10, 2) NOT NULL  |                                         |
| quantity      | INT NOT NULL             |                                         |

### Table: shopAPI.cart

| Column Name   | Data Type                | Constraints                                  |
|---------------|--------------------------|----------------------------------------------|
| cartID        | VARCHAR(36)              | PRIMARY KEY                                  |
| userID        | VARCHAR(36)              | FOREIGN KEY (userID) REFERENCES shopAPI.users(userID) |
| productID     | VARCHAR(36)              | FOREIGN KEY (productID) REFERENCES shopAPI.products(productID) |
| quantity      | INT                      |                                              |

### Table: shopAPI.reviews

| Column Name   | Data Type                | Constraints                                  |
|---------------|--------------------------|----------------------------------------------|
| reviewID      | VARCHAR(36)              | PRIMARY KEY                                  |
| productID     | VARCHAR(36)              | FOREIGN KEY (productID) REFERENCES shopAPI.products(productID) |
| userID        | VARCHAR(36)              | FOREIGN KEY (userID) REFERENCES shopAPI.users(userID) |
| comment       | VARCHAR(500) NOT NULL    |                                              |
| rating        | INT NOT NULL             |                                              |
| reviewTime    | DATETIME NOT NULL         |                                              |

## Tests

Some simple JUnit tests have been implemented:
```java
MySQLConnectionTest()
testRegisterUser_SuccessfulRegistration()
```

## License

E-commerce API is released under the MIT license.

## Author

Sebastian Brzustowicz &lt;Se.Brzustowicz@gmail.com&gt;

