# Unibuddy Engineering Exercise

This exercise is based on the deployed Unibuddy Chat service. The chat service is a core component in our product suite. 
We've based our interview exercise on this code so you can get a feel of the code and products you'd been working on, and we can understand how you would adapt to working with our code base! 


## Chat Service

Chat service provides GraphQL and Rest interfaces to our chat API. We mostly use GraphQL for interacting with it. 

The service is built on [Nest](https://github.com/nestjs/nest), using TypeScript.
We use [Jest](https://jestjs.io/docs/getting-started) for testing and encourage Test Driven Development. 


## Prep
You'll want the following tools installed to help in the exercise:

| Dependency  | Install link |
|------|-----|
|Docker | https://docs.docker.com/get-docker/ |
|Nodejs | https://nodejs.org/en/download/package-manager|
|Nvm | https://github.com/nvm-sh/nvm/blob/master/README.md |

### Usage

cd to the repo (you'll fork and clone this in the exercise itself.)

First, get the supporting services running:

```bash
$ docker compose up -d
```

next, switch to the right versions of node, etc

```bash
$ nvm use
```

Finally, install the require dependencies

```bash
$ npm install
```

Once this is complete, you should be good the run the code - check package.json for examples of what functions are alreasy defined. 

e.g.

```bash
$ npm run test
```

# Interview Exercise

The exercise is broken into 3 parts. To attempt the exercise, we'd like you to fork this repo, making the changes to your own version of the code. 
We encourage you to submit solutions to each part of the exercise as seperate commits, back to your fork, so that it's easier to follow and discuss. 

When you've completed the exercise, please add the following github handles to your fork, so we can review your submission:

https://github.com/davidbebb
https://github.com/RichUnibuddy
https://github.com/anlauren

We encourage pair programming, but do want the exercise to be fair, so please make sure you can complete the exercise solo and explain your work. 

## Part 1

The service fails to start - ```npm run start:dev``` -  Use the messages to fix the code, so that the service runs successfully

## Part 2

A test is failing - ```npm run test``` - impolement the code necessary to pass the test

## Part 3 - Strech

Currently, we allow tags to be added to a conversation, so we can help users to find things they're interested in.

We would like to extend the functionality, to allow the sender of a message to add or update tags on a single message, and allow other users to find messages based on these tags.

While we don't expect everyone to complete this part of the exercise, it will form the basis of disucssion in an interview. Please make as much progress with this, as you feel comfortable doing. Don't allow it to be all-consuming. A couple of hours at most for all parts of the exercise. 

We'd love to hear about

### How you would go about implementing the solution

The functional requirements include:
- sender of message can add/update          a tag
- any user          can search messages using tags

Following is the system design:

**API**

| Endpoint | Method | Payload | Response | Description |
|----------|--------|--------------|----------|-------------|
| `/messages/{messageId}/tags` | GET | N/A | `{ "tags": ["tag1", "tag2", ...] }` | Get all tags for a specific message |
| `/messages/{messageId}/tags` | POST | `{ "tag": "new_tag" }` | `{ "message": "Tag added successfully", "tags": ["tag1", "new_tag", ...] }` | Add a new tag to a message |
| `/messages/{messageId}/tags/{tagName}` | DELETE | N/A | `{ "message": "Tag removed successfully", "tags": ["tag1", ...] }` | Remove a specific tag from a message |
| `/messages/search` | POST | `{ "tags": ["tag1", "tag2"], "conversationId": "conv_id" }` | `{ "messages": [{ "id": "msg_id", "content": "...", "tags": [...] }, ...] }` | Search for messages by tags within a conversation |
| `/conversations/{conversationId}/tags` | GET | N/A | `{ "tags": ["tag1", "tag2", ...] }` | Get all unique tags used in a conversation |
| `/tags` | GET | N/A | `{ "tags": ["tag1", "tag2", ...] }` | Get all unique tags across all messages |


**Database**

1. Update message schema (probably inside `/src/message/models`) to store tags
2. Need to run a migration to add the tags field to all existing documents.
3. Create an index for tags field to improve query performance when searching for message by tags (improves read perfomance but may affect write performance).
4. (maybe; see next question) Validate tags to ensure they meet certain criteria


**Server-side**

1. Create new message DTOs for tag-related operations (adding, updating tags; searching by tags).
2. Update `src/message/.data.ts` to handle these tag-related operations on database.
3. Update `src/message/.resolver.ts` to add new mutations and queries for GraphQL API requests.
4. Update GraphQL schema to include these mutations and queries.
5. (maybe) Update coversation websocket to emit real-time events when tags are added or removed from messages
6. (maybe; see next question) Implement tag suggestions for autocomplete by fetching popular tags from db.


### What problems you might encounter

- normalising tag format to avoid duplication and difficulty in searching: remove whitespace, special charcaters, uppercase 
- autosuggest/autocomplete tag creation
- cache hot tags
- access control on tag addition, modification, and deletion
- use a relational database


### How you would go about testing
1. Unit Tests
    - Normal functionality tests include adding, removing and searching for tags
    - Abnormal functionality tests such as adding duplicate tags, removing non-existant, searching for tags that don't exist
    - Checking proper permissions tests for addition and removal of tags
    - Check mutations and queries create requests and responses which match expected structures
2. Integration Tests
    - Test full folow of each tag operation
3. Performance Tests
    - Find response time and memroy usage when performing operations over large datasets (conversations)
4. Real-time update Tests
    - Test that tag additions and removals are broadcast to all relevant clients



# Additional
The following docs are from the live service repo. You may find them helpful. 


# development
$ npm run start:dev
```

You are now able to make requests to the api.
There are two interfaces; a [rest interface](http://localhost:3000/api), and a [graphql interface](http://localhost:3000/graphql).


The rest interface allows a client to set up a new conversation, and to manage who has access to it.

The graphql interface allows users to send and recieve messages in the conversations that they are in.


You can create a conversation through the [Swagger UI](http://localhost:3000/api) to attain a conversationId (to use with the graphql end points). Some of the requests require authorization. Select the `Authorize` button (top right of the screen) and enter the key `ssssh`. To create a conversation select `Try it out` in POST /conversation.

## Structure

The code in each module is separated into 3 layers
1) controllers and/or resolvers: These provide the external interfaces for the REST and Graphql interfaces respectively, and passes the request to the logic layer
2) logic: This implements common business rules, and can make requests to other modules and the repository layer to fulfil the request.
3) repository: This manages hwo data is stored in the module. It should only be used directly by the logic layer.


my-app/
├─ src/                             
│  ├─ example-graphql/              # Example module
│    ├─ example-graphql.module.ts
│    ├─ example-graphql.repository.spec.ts
│    ├─ example-graphql.repository.ts --------- Controls how the data is stored
│    ├─ example-graphql.resolver.spec.ts
│    ├─ example-graphql.resolver.ts ----------- Provides the external interface for the service
│    ├─ example-graphql.logic.spec.ts
│    ├─ example-graphql.logic.ts -------------- Implements common business logic
│  ├─ app.module.ts
│  ├─ main.ts

## Testing

- Unit test file of each file is created in the same path under name of <fileName>.spec.ts
- E2e tests are in test folder
- Jest is used for creating these tests.

To run the unit tests you wil need to have the databases running - run `docker compose up -d`

```bash
# unit tests
$ npm run test
$ npm run test:watch

# e2e tests - the service needs to be running - see Running the app
$ npm run test:e2e
$ npm run test:e2e:watch

```


## Mocking the unibuddy_api end point (e.g /api/v1/users/)

You may want to mock data from the ub_internal_api end points. We can do this using [Mock Server](https://www.mock-server.com/)


For `/api/v1/users/{$userId}` an example would be 

url: /api/v1/users/599ebd736a1d100004aeb744

```
{
"account_role": "university",
"email": "edinburgh+admin@unibuddy.co",
"first_name": "Uni",
"id": "599ebd736a1d100004aeb744",
"last_name": "Admin",
"profile_photo": null
}
```
