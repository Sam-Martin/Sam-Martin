---
title: "Querying Appsync With Python"
date: 2020-05-17T16:08:25+01:00
image: /images/2020/05/2020-05-17-AppSync-Getting-Started.png
summary: "AWS AppSync is a really easy way to quickly assemble a GraphQL API to interact with data from DynamoDB, Lambda, ElasticSearch, RDS, or other HTTP endpoints.

You don't have to create an API Gateway, then fiddle around with hooking it up with DynamoDB mappings, or lambdas in the middle. You define a data model, queries, and mutations, and AppSync provides an endpoint for you to query it. Hell it'll even go off and create the DynamoDB for you if you use the wizard."
draft: false
---

# AWS AppSync
AWS AppSync is a really easy way to quickly assemble a GraphQL API to interact with data from DynamoDB, Lambda, ElasticSearch, RDS, or other HTTP endpoints.

You don't have to create an API Gateway, then fiddle around with hooking it up with DynamoDB mappings, or lambdas in the middle. You define a data model, queries, and mutations, and AppSync provides an endpoint for you to query it. Hell it'll even go off and create the DynamoDB for you if you use the wizard.

This spike was my first interaction with GraphQL full stop as I've always historically worked with JSON APIs, so in this I'm not going to assume you know anything about GraphQL (as I sure as shit don't!).

# GraphQL vs REST
There are a billion different articles on the pros and cons, but the short answer is that GraphQL provides a lot of flexibility in what data 

# Setting up AppSync

I chose to start using the Wizard without a sample project so I could build it up myself to understand it better.

![AppSync Getting Started](/images/2020/05/2020-05-17-AppSync-Getting-Started.png)

Let's create a very simple data structure to list our favourite movies.

![AppSync Create a Model](/images/2020/05/2020-05-17-AppSync-Create-a-Model.png)

Name it
![AppSync Create Resources](/images/2020/05/2020-05-17-AppSync-Create-Resources.png)

Done! Now we can query it! In the background AWS went off and created an API Endpoint, DynamoDB Table, and a bunch of GraphQL definitions to define how we're allowed to query the API.  
It's now dumped us at the Query page, which is a nice way to get started.

![AppSync Queries](/images/2020/05/2020-05-17-AppSync-Queries.png)

Go ahead and click the orange Play button to `createFavouriteMovies`. Some dummy data will then appear on the right hand side as the API response to our create request.   

You can now hit play again, and hit `listFavouriteMovies` (it will look _identical_ however because we only have the one record for now!).

Now we need to figure out how to query the API outside AWS though!

# Querying AppSync with Python

You've probably noticed the message in green at the top which invites you to "learn how to integrate [your new API] with your app.", which takes you to this page.

![AppSync Queries](/images/2020/05/2020-05-17-AppSync-Integrate-With-Your-App.png)

Which.. uh.. doesn't have a lot of Python on it. Appsync is pretty Web and Mobile oriented and offers some _great_ features to this effect such as websockets and offline data management in React. We don't care about any of that though! We want Python!

Go to the Settings section and you'll find what you need to get started.

![AppSync Settings](/images/2020/05/2020-05-17-AppSync-Settings.png)

Make a note of:
* API URL
* API Key

and open up your favourite Python editor and throw in:


```
from urllib import request
from datetime import datetime
import simplejson


class GraphqlClient:

    def __init__(self, endpoint, headers):
        self.endpoint = endpoint
        self.headers = headers

    @staticmethod
    def serialization_helper(o):
        if isinstance(o, datetime):
            return o.strftime('%Y-%m-%dT%H:%M:%S.000Z')

    def execute(self, query, operation_name, variables={}):
        data = simplejson.dumps({
            "query": query,
            "variables": variables,
            "operationName": operation_name
        },
            default=self.serialization_helper,
            ignore_nan=True
        )
        r = request.Request(
            headers=self.headers,
            url=self.endpoint,
            method='POST',
            data=data.encode('utf8')
        )
        response = request.urlopen(r).read()
        return response.decode('utf8')

gq_client = GraphqlClient(
    endpoint='https://xxxxxxxx.appsync-api.eu-west-1.amazonaws.com/graphql',
    headers={'x-api-key': 'xxx-xxxxxxxxxxxxxxx'}
)

result = gq_client.execute(
    query="""
query listFavouriteMovies {
  listFavouriteMovies {
    items {
      id
      title
      rating
    }
  }
}

""", 
    operation_name='listFavouriteMovies'
)
print(result)
```

Replacing the `endpoint` and `x-api-key` values with your endpoint URL and API key.

Prior to running the script you'll need to install simplejson with `pip install simplejson`, but after that you should be able to do:

```
$ python AppSyncExample.py
{"data":{"listFavouriteMovies":{"items":[{"id":"35ef6587-9c8b-4e2e-95f9-453108eb4b04","title":"Hello, world!","rating":76}]}}}
```

Beautiful! We've successfully queried our GraphQL API from Python!
The boilerplate class `GraphqlClient` does some of the stuff (and is named identically) to the [python-graphql-client](https://github.com/prisma-labs/python-graphql-client) library, but it's easier to see what's going on without installing and calling a 3rd party plugin. We're using `simplejson` because the AppSync GraphQL API doesn't get on with some of the more complex JSON values like `NaN` so we strip them out.


# Running a Insert Operation.

We've done a query, now let's write something.
Delete everything between the triple quotes in the `result = gq_client.execute(query=` call and replace it with a new GraphQL call, a mutation:

```
mutation createFavouriteMovies($createfavouritemoviesinput: CreateFavouriteMoviesInput!) {
  createFavouriteMovies(input: $createfavouritemoviesinput) {
    id
    title
    rating
  }
}
```

This provides the structure for inserting our record. Now we need to add the variables as an additional argument in the same `execute` method call.

```
variables={
    "createfavouritemoviesinput": {
        "title": "Thor: Ragnarok", 
        "rating": 10
    }
}
```
Note the top level key `createfavouritemoviesinput` which corresponds to the arg we're passing to the `createFavouriteMovies` mutation.

We also need to update the `operation_name` param to be `createFavouriteMovies`.

Once you're done it should look like:

```
result = gq_client.execute(query= """
mutation createFavouriteMovies($createfavouritemoviesinput: CreateFavouriteMoviesInput!) {
  createFavouriteMovies(input: $createfavouritemoviesinput) {
    id
    title
    rating
  }
}
""", 
    operation_name='createFavouriteMovies',
    variables={
        "createfavouritemoviesinput": {
            "title": "Thor: Ragnarok", 
            "rating": 10
        }
    }
)
```
And running it should give you:

```
$ python AppSyncExample.py
{"data":{"createFavouriteMovies":{"id":"a4c007e9-b0f4-45d3-816d-8c9acaef259d","title":"Thor: Ragnarok","rating":10}}}
```

Beautiful!
Now If you swap the call back to the original `listFavouriteMovies` query we started off with, you should get both results back!

```
$ python AppSyncExample.py
{"data":{"listFavouriteMovies":{"items":[{"id":"6456c73f-6e72-43e4-b432-01ea7fe85448","title":"Thor: Ragnarok","rating":10},{"id":"35ef6587-9c8b-4e2e-95f9-453108eb4b04","title":"Hello, world!","rating":76}]}}}
```

There's so much more you can do with AppSync, and honestly the some of coolest features are the AWS owned client libraries for React and React Native that do client side caching for offline data and optimistic submissions.

# Query Creation

I highly recommend the [Insomnia](https://insomnia.rest/download/) tool to make your life easier when writing GraphQL queries. It allows you to browse the schema, gives you autocomplete. Flags missing/incomplete fields, etc.

Once you've installed it, create a new query

![Insomnia Create a GraphQL Query](/images/2020/05/2020-05-17-Insomnia-GraphQL.png)

Then pop to headers and populate your credentials

![Insomnia Populate Credentials](/images/2020/05/2020-05-17-Insomnia-API-Key.png)

Then you can refresh the schema.
![Insomnia Refresh Schema](/images/2020/05/2020-05-17-Insomnia-RefreshSchema.png)

After which you're free to browse the API schema (Schema > Show Documentation), write queries, submit them, and see the way the payloads are passed around and what headers are expected/received.

![Insomnia Write Queries](/images/2020/05/2020-05-17-Insomnia-WriteQueries.png)

# Further Reading
If you want to learn more about AppSync's capabilities I suggest you check out this introductory video from AWS here: 
* [Introduction to AWS AppSync and GraphQL - AWS Online Tech Talks - Richard Threlkeld - 2018](https://www.youtube.com/watch?v=olAPj6EIlag)