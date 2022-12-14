# Getting Started

## Serverless 

Your ability to delegate more of your operational duties to AWS and so increase your agility and innovation is made possible by serverless, which is the cloud's inherent architecture. You may create and execute apps and services using serverless technology without giving servers any thought. It does away with infrastructure management duties like capacity provisioning, operating system upkeep, server or cluster provisioning, and patching. Almost any backend service or application can be built with them, and everything needed to run and scale your application with high availability is taken care of for you.

## AWS Lambda

You can run code with AWS Lambda without setting up or managing servers. There is no fee when your code is not executing; you only pay for the compute time you use.

You may use Lambda to run code for almost any kind of application or backend service with no administration required. Simply upload your code, and Lambda will handle all the necessary steps to run and grow it with high availability.


## Amazon API Gateway

In order to construct, publish, maintain, monitor, and protect APIs at any size, developers can use the fully managed service known as Amazon API Gateway. It provides a complete framework for managing APIs. API Gateway manages traffic, permission and access control, monitoring, and API version management while enabling you to perform hundreds of thousands of API calls concurrently.

## Amazon DynamoDB

A consistent, single-digit millisecond latency at any scale is required by all applications, and Amazon DynamoDB provides a quick and adaptable NoSQL database service that can meet this need.

For internet-scale applications, it is a fully managed, multiregion, multimaster, persistent database with built-in backup and restore, security, and in-memory caching.

## Amazon S3

Amazon Simple Storage Service is a cloud-based object storage service with industry-leading scalability, data availability, security, and performance. Amazon S3 is a simple web service interface that allows you to store and retrieve any amount of data from anywhere on the internet.

Customers of all sizes and industries can use it to store and protect any amount of data for a variety of use cases, including websites, mobile apps, backup and restore, archiving, enterprise applications, IoT devices, and big data analytics.

# Demo
 

# Serverless TODO

In this project I develop and deploy a simple "TODO" application using AWS Lambda and Serverless framework. This application will allow users to create/remove/update/get TODO items. Each TODO item contains the following fields: 
   - todoId (string) - a unique id for an item
   - createdAt (string) - date and time when an item was created
   -  name (string) - name of a TODO item (e.g. "Change a light bulb")
   - dueDate (string) - date and time by which an item should be completed
   - done (boolean) - true if an item was completed, false otherwise
   - attachmentUrl (string) (optional) - a URL pointing to an image attached to a TODO item

# Functions implemented

To implement this project you need to implement the following functions and configure them in the `serverless.yml` file:

* `Auth` - this function should implement a custom authorizer for API Gateway that should be added to all other functions.
* `GetTodos` - should return all TODOs for a current user. 
* `CreateTodo` - should create a new TODO for a current user. A shape of data send by a client application to this function can be found in the `CreateTodoRequest.ts` file
* `UpdateTodo` - should update a TODO item created by a current user. A shape of data send by a client application to this function can be found in the `UpdateTodoRequest.ts` file
* `DeleteTodo` - should delete a TODO item created by a current user. Expects an id of a TODO item to remove.
* `GenerateUploadUrl` - returns a presigned url that can be used to upload an attachment file for a TODO item. 

All functions are already connected to appriate events from API gateway

An id of a user can be extracted from a JWT token passed by a client

You also need to add any necessary resources to the `resources` section of the `serverless.yml` file such as DynamoDB table and and S3 bucket.


## Deploy Backend

                  cd backend

                  npm install
  
                  serverless

                  npm install --save-dev serverless-iam-roles-per-function@next 

                  serverless deploy --verbose


# Endpoints
   
  - GET - https://mc3aj77le6.execute-api.us-east-1.amazonaws.com/dev/todos
  - POST - https://mc3aj77le6.execute-api.us-east-1.amazonaws.com/dev/todos
  - PATCH - https://mc3aj77le6.execute-api.us-east-1.amazonaws.com/dev/todos/{todoId}
  - DELETE - https://mc3aj77le6.execute-api.us-east-1.amazonaws.com/dev/todos/{todoId}
  - POST - https://mc3aj77le6.execute-api.us-east-1.amazonaws.com/dev/todos/{todoId}/attachmentattachment

functions:
  
# Lambda Functions

  - Auth: serverless-dev-Auth
  - GetTodos: serverless-dev-GetTodos
  - CreateTodo: serverless-dev-CreateTodo
  - UpdateTodo: serverless-dev-UpdateTodo
  - DeleteTodo: serverless-dev-DeleteTodo
  -  GenerateUploadUrl: serverless-dev-GenerateUploadUrl 
  

# Frontend

The `client` folder contains a web application that can use the API that should be developed in the project.

To use it please edit the `config.ts` file in the `client` folder:

```ts
const apiId = '...' API Gateway id
export const apiEndpoint = `https://${apiId}.execute-api.us-east-1.amazonaws.com/dev`

export const authConfig = {
  domain: '...',    // Domain from Auth0
  clientId: '...',  // Client id from an Auth0 application
  callbackUrl: 'http://localhost:3000/callback'
}
```


# Suggestions

To store TODO items you might want to use a DynamoDB table with local secondary index(es). A create a local secondary index you need to a create a DynamoDB resource like this:

```yml

TodosTable:
  Type: AWS::DynamoDB::Table
  Properties:
    AttributeDefinitions:
      - AttributeName: partitionKey
        AttributeType: S
      - AttributeName: sortKey
        AttributeType: S
      - AttributeName: indexKey
        AttributeType: S
    KeySchema:
      - AttributeName: partitionKey
        KeyType: HASH
      - AttributeName: sortKey
        KeyType: RANGE
    BillingMode: PAY_PER_REQUEST
    TableName: ${self:provider.environment.TODOS_TABLE}
    LocalSecondaryIndexes:
      - IndexName: ${self:provider.environment.INDEX_NAME}
        KeySchema:
          - AttributeName: partitionKey
            KeyType: HASH
          - AttributeName: indexKey
            KeyType: RANGE
        Projection:
          ProjectionType: ALL # What attributes will be copied to an index

```

To query an index you need to use the `query()` method like:

```ts
await this.dynamoDBClient
  .query({
    TableName: 'table-name',
    IndexName: 'index-name',
    KeyConditionExpression: 'paritionKey = :paritionKey',
    ExpressionAttributeValues: {
      ':paritionKey': partitionKeyValue
    }
  })
  .promise()
```


## Deploy Frontend

To run a client application first edit the `client/src/config.ts` file to set correct parameters. And then run the following commands

```
cd client
npm install
npm run start
```


This should start a development server with the React application that will interact with the serverless TODO application.

