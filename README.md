# AWS_Refactor_Monolith_to_Microservices

Prerequisites
Auth0 account
GitHub account
NodeJS version up to 12.xx
Serverless
Create a Serverless account user
Install the Serverless Framework’s CLI (up to VERSION=2.21.1). Refer to the official documentation for more help.
npm install -g serverless@2.21.1
serverless --version


Login and configure serverless to use the AWS credentials
# Login to your dashboard from the CLI. It will ask to open your browser and finish the process.
serverless login
# Configure serverless to use the AWS credentials to deploy the application
# You need to have a pair of Access key (YOUR_ACCESS_KEY_ID and YOUR_SECRET_KEY) of an IAM user with Admin access permissions
sls config credentials --provider aws --key YOUR_ACCESS_KEY_ID --secret YOUR_SECRET_KEY --profile serverless


Project overview
In this project you will develop and deploy a simple "TODO" application using AWS Lambda and Serverless framework. This application will allow users to create/remove/update/get TODO items. Each TODO item contains the following fields:
todoId (string) - a unique id for an item
createdAt (string) - date and time when an item was created
name (string) - name of a TODO item (e.g. "Change a light bulb")
dueDate (string) - date and time by which an item should be completed
done (boolean) - true if an item was completed, false otherwise
attachmentUrl (string) (optional) - a URL pointing to an image attached to a TODO item
You might also store an id of a user who created a TODO item. Each TODO item can optionally have an attachment image. Each user only has access to TODO items that he/she has created.

Welcome page before login

Auth0 login and authorization prompt

App running successfully.
Images courtesy: Pixabay.com (Free for commercial use. No attribution required)
Tasks overview
Fork and clone the project starter code. It comes with a backend and frontend client application.
The starter code comes with TODO flags in various files. Search for all the TODO: comments in the code to find the placeholders that you need to implement.
Tip: Use grep -r "TODO" . command in the project directory to list out the filenames.
These are the files that needs to be updated.

Backend directory structure with highlighted files to edit
Before you dive deeper, it's a good practice to have a quick look at the project rubric. Your submission will be evaluated against this rubric.
Backend
The serverless.yml file, your starting point, has the pre-built endpoints referencing to the the following handler functions:
# Your TODO: tasks to implement
└── backend
    ├── serverless.yml                    #TODO:
    └── src
        ├── auth
        └── lambda
            ├── auth
            │   └── auth0Authorizer.ts    #TODO:
            └── http
                ├── createTodo.ts         #TODO:
                ├── deleteTodo.ts         #TODO:
                ├── generateUploadUrl.ts  #TODO:
                ├── getTodos.ts           #TODO:
                └── updateTodo.ts         #TODO:

The handler functions above, in turn, will require additional helpers function. The helper files in this following directory handle different layers like businessLogic, dataLayer, and fileStorage, thereby following a "separation of concerns" strategy.
# Your TODO: tasks to implement
└── backend
    └── src
        ├── auth
        └── helpers                    # Handle different layers
            ├── todos.ts            # TODO: Implement businessLogic
            ├── todosAcess.ts       # TODO: Implement dataLayer
            └── attachmentUtils.ts  # TODO: Implement: fileStorage 

Tip: In the handler/helper functions, use async/await constructs instead of passing callbacks, to get results of asynchronous operations.
You need to implement all the handler and helper functions listed above. Here is the detailed list:
Auth
This function is already defined in the serverless file.
See the /backend/src/lambda/auth/auth0Authorizer.ts file for your specific tasks. This function should implement a custom authorizer for API Gateway that should be added to all other functions.
GetTodos
In the serverless file, provide iamRoleStatements property for performing necessary Actions on DynamoDB
See the /backend/src/lambda/http/getTodos.ts file. It should return all TODOs for a current user. A user id can be extracted from a JWT token that is sent by the frontend. It should return data that looks like this:
{
"items": [
{
"todoId": "123",
"createdAt": "2019-07-27T20:01:45.424Z",
"name": "Buy milk",
"dueDate": "2019-07-29T20:01:45.424Z",
"done": false,
"attachmentUrl": "http://example.com/image.png"
},
{
"todoId": "456",
"createdAt": "2019-07-27T20:01:45.424Z",
"name": "Send a letter",
"dueDate": "2019-07-29T20:01:45.424Z",
"done": true,
"attachmentUrl": "http://example.com/image.png"
},
]
}


CreateTodo
In the serverless file, provide iamRoleStatements property. Decide the Actions and AWS Resource.
See the /backend/src/lambda/http/createTodos.ts file. It should create a new TODO for a current user. A shape of data send by a client application to this function can be found in the /backend/src/requests/CreateTodoRequest.ts file. It receives a new TODO item to be created in JSON format that looks like this:
{
 "createdAt": "2019-07-27T20:01:45.424Z",
 "name": "Buy milk",
 "dueDate": "2019-07-29T20:01:45.424Z",
 "done": false,
 "attachmentUrl": "http://example.com/image.png"
}

It should return a new TODO item that should looks like this:
{
 "item": {
  "todoId": "123",
  "createdAt": "2019-07-27T20:01:45.424Z",
  "name": "Buy milk",
  "dueDate": "2019-07-29T20:01:45.424Z",
  "done": false,
  "attachmentUrl": "http://example.com/image.png"
}
}


UpdateTodo
See your task for this function the serveerless file.
See the /backend/src/lambda/http/updateTodo.ts file. It should update a TODO item created by a current user. A shape of data send by a client application to this function can be found in the /backend/src/requests/UpdateTodoRequest.ts file. It receives an object that contains three fields that can be updated in a TODO item:
{
 "name": "Buy bread",
 "dueDate": "2019-07-29T20:01:45.424Z",
 "done": true
}


The ID of an item that should be updated is passed as a URL parameter. It should return an empty body.
DeleteTodo
See your task for this function the serveerless file.
See the /backend/src/lambda/http/deleteTodo.ts file. It should delete a TODO item created by a current user. Expects an id of a TODO item to remove. It should return an empty body.
GenerateUploadUrl
Again, see your task for this function the serveerless file.
See the /backend/src/lambda/http/generateUploadUrl.ts file. It returns a pre-signed URL that can be used to upload an attachment file for a TODO item. It should return a JSON object that looks like this:
{
"uploadUrl": "https://s3-bucket-name.s3.eu-west-2.amazonaws.com/image.png"
}


All functions above are already connected to appropriate events from API Gateway.
CORS configuration in the handler functions
You can return correct headers in API handler responses, in either of the following two ways:
You can either include the CORS headers in the handler function, as:
export const handler: APIGatewayProxyHandler = async (event: APIGatewayProxyEvent): Promise<APIGatewayProxyResult> => {
const newTodo: CreateTodoRequest = JSON.parse(event.body);
// Write your logic here
.
.
.
return {
 statusCode: 201,
 headers: {
   'Access-Control-Allow-Origin': '*',
   'Access-Control-Allow-Credentials': true
 },
 body: JSON.stringify({
   item: todoItem
 })
};
}


Otherwise, you can use middy middleware package, for example:
import * as middy from 'middy'
import { cors, httpErrorHandler } from 'middy/middlewares'
export const handler = middy(
 async (event: APIGatewayProxyEvent): Promise<APIGatewayProxyResult> => {
     // Write your logic here
     .
     .
     .
     return undefined
 }
)
handler
 .use(httpErrorHandler())
 .use(
     cors({
       credentials: true
  })
)

Refer here for more examples.
In addition, you must declare the function in the serverless.yml file with the CORS property as cors: true, see an example:
 <FUNCTION-NAME>:
    handler: <YOUR-HANDLER>
    events:
      - http:
          method: get
          path: todos
          cors: true
          authorizer: Auth
    iamRoleStatementsInherit: true
    iamRoleStatements:
      - Effect: Allow
        Action:
          - dynamodb:GetItem
          - dynamodb:UpdateItem
          - dynamodb:PutItem
        Resource: arn:aws:dynamodb:<TABLE-NAME>


AWS Resources
Next, you need to add necessary AWS resources to the resources section of the serverless.yml file, such as, DynamoDB table and S3 bucket. We have identifies the minimal set of AWS resources in the serverless file. Check it out.
For any help in the Resources section of the Serverless YAML file, refer to the examples below:
AWS::ApiGateway::GatewayResponse
AWS::DynamoDB::Table
AWS::S3::Bucket - See the Enable cross-origin resource sharing example.
AWS::S3::BucketPolicy
DynamoDB in the Serverless YAML file
Create DynamoDB table - To store TODO items, use a DynamoDB table with local secondary index(es). You can refer to an example of how to use LocalSecondaryIndex in your Serverless YAML file's resource section. Your DynamoDB resource code would like this:
# Here "TodosTable" is the name for cross-referencing  in the serverless.yml file
# It's not the name of the actual DynamoDB table. 
TodosTable:
Type: AWS::DynamoDB::Table
Properties:
  AttributeDefinitions:
    - AttributeName: userId
      AttributeType: S
    - AttributeName: todoId
      AttributeType: S
    - AttributeName: createdAt
      AttributeType: S
  KeySchema:
    - AttributeName: userId
      KeyType: HASH
    - AttributeName: todoId
      KeyType: RANGE
  BillingMode: PAY_PER_REQUEST
  TableName: ${self:provider.environment.TODOS_TABLE}
  LocalSecondaryIndexes:
    - IndexName: ${self:provider.environment.TODOS_CREATED_AT_INDEX}
      KeySchema:
        - AttributeName: userId
          KeyType: HASH
        - AttributeName: createdAt
          KeyType: RANGE
      Projection:
        ProjectionType: ALL # What attributes will be copied to an index

In the code above, the userId is acting like a partitionKey, todoId as the sortKey, and createdAt as the indexKey. Also, it uses the variables defined in the provider.environment section at the top of the serverless.yaml file. For example:
provider:
  stage: ${opt:stage, 'dev'}
  region: ${opt:region, 'us-east-1'}
  environment:
      TODOS_TABLE: Todos-${self:provider.stage}
      TODOS_CREATED_AT_INDEX: CreatedAtIndex

Do not copy the YAML code snippets from above, it may cause indentation issues during copying over.
Fetch data from DynamoDB table - To query an index you need to use the query() method like:
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
Best practices
Validate HTTP requests
Validate incoming HTTP requests either in Lambda handlers or using request validation in API Gateway. The latter can be done either using the serverless-reqvalidator-plugin or by providing request schemas in function definitions.
Application monitoring
The project application has at least some of the following:
Metrics: Generate application-level metrics
Enable the distributed tracing. We recommend to use AWS X-ray for this purpose. To do this, you will have to:
Enable X-ray tracing in the serverless YAML file
Instrument code in the handler functions to generate sub-segments. Meaning, wrap all AWS calls with X-ray SDK.
Once you deploy and test your API endpoints, you will need to check the Service map in the AWS X-Ray web console. View the traces.
Logging: The application must have a sufficient amount of log statements. The starter code comes with a configured Winston logger that creates JSON formatted log statements. You can use it to write log messages like this:
import { createLogger } from '../../utils/logger'
const logger = createLogger('auth')
// You can provide additional information with every log statement
// This information can then be used to search for log statements in a log storage system
logger.info('User was authorized', {
// Additional information stored with a log statement
key: 'value'
})
Deploy the Backend
To deploy the backend application, run the following commands:
cd backend
npm update --save
npm audit fix
# For the first time, create an application in your org in Serverless portal
serverless
# Next time, deploy the app and note the endpoint url in the end
serverless deploy --verbose
# If you face a permissions error, you may need to specify the user profile
sls deploy -v --aws-profile serverless
# sls is shorthand for serverless
# -v is shorthand for --verbose
 
If deployment is successful, then you can
Check the Serverless dashboard update
Check the AWS resources - API Gateway, S3, Lambda, CloudWatch logs
Verify the endpoints.
Otherwise, see the Troubleshooting tips on the page next.

A successful deployment will create a resource stack in the CloudFormation console
Configure the Frontend
The /client/ folder contains the frontend web application which consumes the backend API developed in this project. You don't need to make any changes to the frontend code in the /client/ folder, except for the Authentication related changes, as explained below.
Authentication - Login to the Auth0 portal, and navigate to your Dashboard.
Create a "Single Page Web Applications" type Auth0 application
Go to the App settings, and setup the Allowed Callback URLs
Setup the Allowed Web Origins for CORS options.
Setup the application properties. We recommend using asymmetrically encrypted (RS256) JWT tokens.
Copy "domain" and "client id" to save in the /client/src/config.ts file.
In your backend auth handler function, fetch the Auth0 certificate programmatically.

Auth0 App settings
Edit the /client/src/config.ts file to configure your Auth0 client application and API endpoint:
// TODO: Once your application is deployed, copy an API id here so that the frontend could interact with it
// apiId is the ID generated by the serverless deploy command
const apiId = '...' 
export const apiEndpoint = `https://${apiId}.execute-api.us-east-1.amazonaws.com/dev`
export const authConfig = {
// TODO: Create an Auth0 application and copy values from it into this map
domain: '...',    // Domain from Auth0
clientId: '...',  // Client id from an Auth0 application
callbackUrl: 'http://localhost:3000/callback'  // Localhost URL that the front
}
 
Run the Frontend
Once you've set the paramteres in the client/src/config.ts file, run the following commands:
cd client
npm update --save
npm audit fix --legacy-peer-deps
npm install --save-dev
npm run start
 
This should start a React development server at http://localhost:3000/ that will interact with the backend APIs.
Need help with the errors? See the troubleshooting tips on the page next.
Postman collection
An alternative way to test your API is to use the Postman. You can find a Postman collection, Final Project.postman_collection.json, that contains sample requests in the project repo. To import this collection do the following.
Start the Postman application and import the Final Project.postman_collection.json file as a collection.
Set the environment variables before making any requests.
Troubleshoot
Make sure the serverless deploy --verbose did not emit any errors.
Failed to compile after running the frontend. This is mostly due to NodeJS dependencies used in your frontend package.json file.

(Frontend) Error due to typescript dependency compatibility
Status code 403 error while fetching a resource occurs due to unauthorized access. Check your:
backend/src/lambda/auth/auth0Authorizer.ts file.
domain and clientId in the client/src/config.ts file.

(Backend API) Unauthorized access error due to not setting up the URL that can be used to download the Auth0 certificate
CORS error or error while uploading your image file may occur due to many reasons:
CORS is not set correctly in the Lambda functions
CORS is not set in the API Gateway
CORS is not set at the S3 bucket level permissions
apiId is not set correctly in the client/src/config.ts file.
Sometimes other errors, that can only be traced in the CloudWatch logs. See an example below.

CORS error while trying to upload the image.
It says No 'Access-Control-Allow-Origin' header is present on the requested resource.
Check your CloudWatch log groups. In the example below, it shows that a CORS error occurred due to a data type mismatch in the backend/src/helpers/attachmentUtils.ts file.

Check your AWS CloudWatch log groups

Log events in a particular Log group

Details of an error
If you are using a federated AWS user account (called Vocareum) provided by Udacity, and facing an unexpected error in the AWS web console, then you need to log out and re-login to your AWS console.

Error at AWS API Gateway, faced by Vocareum users
 

Set the environment variables before making any GET/POST requests

Verify the backend APIs from the Postman client
NEXT
 
 
Project Submission
 DUE DATE
Apr 18
Project deadlines help you stay on track but are not enforced. Deadlines can be adjusted from Program Home.
 STATUS
Unsubmitted
Project past due
Once you have finished developing your application, please set apiId and Auth0 parameters in the config.ts file in the client folder. A reviewer would start the React development server to run the frontend that should be configured to interact with your serverless application.
IMPORTANT
Please leave your application running until a submission is reviewed. If implemented correctly it will cost almost nothing when your application is idle.
Make sure you review the Project Rubric before submitting your project.

1) Functionality
CRITERIA
MEETS SPECIFICATIONS
The application allows users to create, update, delete TODO items
A user of the web application can use the interface to create, delete and complete a TODO item.
The application allows users to upload a file.
A user of the web interface can click on a "pencil" button, then select and upload a file. A file should appear in the list of TODO items on the home page.
The application only displays TODO items for a logged in user.
If you log out from a current user and log in as a different user, the application should not show TODO items created by the first account.
Authentication is implemented and does not allow unauthenticated access.
A user needs to authenticate in order to use an application.

2) Code Base
CRITERIA
MEETS SPECIFICATIONS
The code is split into multiple layers separating business logic from I/O related code.
Code of Lambda functions is split into multiple files/classes. The business logic of an application is separated from code for database access, file storage, and code related to AWS Lambda.
Code is implemented using async/await and Promises without using callbacks.
To get results of asynchronous operations, a student is using async/await constructs instead of passing callbacks.

3) Best Practices
CRITERIA
MEETS SPECIFICATIONS
All resources in the application are defined in the "serverless.yml" file
All resources needed by an application are defined in the "serverless.yml". A developer does not need to create them manually using AWS console.
Each function has its own set of permissions.
Instead of defining all permissions under provider/iamRoleStatements, permissions are defined per function in the functions section of the "serverless.yml".
Application has sufficient monitoring.
Application has at least some of the following:
Distributed tracing is enabled
It has a sufficient amount of log statements
It generates application level metrics
HTTP requests are validated
Incoming HTTP requests are validated either in Lambda handlers or using request validation in API Gateway. The latter can be done either using the serverless-reqvalidator-plugin or by providing request schemas in function definitions.

4) Architecture
CRITERIA
MEETS SPECIFICATIONS
Data is stored in a table with a composite key.
1:M (1 to many) relationship between users and TODO items is modeled using a DynamoDB table that has a composite key with both partition and sort keys. Should be defined similar to this:
  KeySchema:
      - AttributeName: partitionKey
        KeyType: HASH
      - AttributeName: sortKey
        KeyType: RANGE


Scan operation is not used to read data from a database.
TODO items are fetched using the "query()" method and not "scan()" method (which is less efficient on large datasets)

Suggestions to Make Your Project Stand Out!
Fetch a certificate from Auth0 instead of hard coding it in an authorizer.
Implement pagination support to work around a DynamoDB limitation that allows up to 1MB of data using a query method.
Add your own domain name to the service.
Add an ability to sort TODOs by due date or priority (this will require adding new indexes).
Implement a new endpoint that allows sending full-text search requests to Elasticsearch (this would require copying data from DynamoDB to Elasticsearch as we did in lesson 4).


