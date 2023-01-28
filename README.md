# Java Sample App using Spring Cloud Vault

This to demostrate spring cloud vault integration with Hashicorp vault (community edition).
(Please refer for more details : https://learn.hashicorp.com/collections/vault/encryption-as-a-service)
There are 2 services in this application.
1. Vault-server
   This is used to encrypt and decrypt the sensitive data. This will not store any data in his system.

2. spring-vault 
   This is our main application which is going to add and retrieve user information. This will interact with vault-server to encrypt and decrypt the data on demand.
   This will store encrypted data in PostgreSQL database. We will never store the plain or decrypted data in any storage.


## Setup

You can run the sample as a standalone Java application. You will need a Vault instance and a Postgres instance to get started.

1. Build the spring-vault image using below commands by navigating to spring-cloud-vault folder :
   mvn clean install
   docker build -t spring-vault --build-arg JAR_FILE=/target/spring-vault-demo-1.0.jar .

2. Start the services using below command in main folder (springVaultDemo)
   docker-compose up
  
3. Setup Vault service
   After starting the services, we will proceed with setting up the vault by executing below commands on CLI terminal of vault. This is one time activity.

   vault operator init --key-shares=5 -key-threshold=2

   Above commands will generate unseal and root keys. Please make sure that you copy it somewhere for future use.
   Now vault is initialized but it is sealed state. To unseal, execute below command with any 2 unseal keys. (We have to unseal the vault to be after every restart, However there is way to auto unseal using azure/aws/gcp cloud providers.)

   vault operator unseal <<key1>>
   vault operator unseal <<key2>>

   Once it is unsealed,you can open the UI and enable transit(encryption service).
   http://localhost:8200

   Login using the generated root key. Once you are logged, enable the transit service and create the encryption key with name "creditCards".
   Please note that you can create key with any name but you will have to update the same in spring-vault service.

   You can test the encryption and decryption service using provided postman collection (encrypt and decrypt request).
   Please the update "X-Vault-Token" header with root key.

4. Update the vault token in environment
   In the docker-compose.yaml file, update the "VAULT.TOKEN" environment variable of spring-valut service with the value the current root key.
   Please note that for demo purpose, we are using root key. Ideal way is to create different token key based on policies as per documentation.

5. DB setup
   Connect to PostgreSQL database and create the required table using below sql :

   CREATE TABLE orders (
    id bigserial primary key,
    customer_name varchar(60) NOT NULL,
    product_name varchar(20) NOT NULL,
    order_date timestamp NOT NULL
  );

6. Reinitialize the services using below command in main folder (springVaultDemo)
   docker-compose up

7. Test the application
   You can test the application using provided postman collection (addUser and getUser). You will see that you will always receive the decrypted value of customer name but in the database, it is stored as encrypted value.
