---
title: "Subscriptions to show online users"
metaTitle: "Update last seen of user with Mutation | GraphQL React Native Apollo Tutorial"
metaDescription: "GraphQL Mutation to update last seen of user to make them available online. Use setInterval to trigger mutation every few seconds "
---

import GithubLink from "../src/GithubLink.js";

We cruised through our GraphQL queries and mutations. We queried for todos, added a new todo, updated an existing todo, removed an existing todo.

Now let's get to the exciting part.

GraphQL Subscriptions
---------------------

We have a section of UI which displays the list of online users. So far we have made queries to fetch data and display them on the UI. But typically online users data is dynamic.

We can make use of GraphQL Subscription API to get realtime data from the graphql server to efficiently handle this.

But but but...

We need to tell the server that the user who is logged in is online. We have to poll our server to do a mutation which updates the `last_seen` timestamp value of the user.

We have to make this change to see yourself online first. Remember that you are already logged in, registered your data in the server, but not updated your `last_seen` value.?

The goal is to update every few seconds from the client that you are online. Ideally you should do this after you have successfully authenticated with Auth0. So let's do in the entrypoint of the app i.e. `src/navigation/Main.js`. We instantiate `client` after after the component's first mount. Thats when we want to start polling. Firstly, lets define the mutation that sets `last_seen` to the current timestamp.

<GithubLink link="https://github.com/hasura/learn-graphql/blob/master/tutorials/mobile/react-native-apollo/app-final/src/navigation/Main.js" text="Main.js"/>

```javascript
+ import gql from "graphql-tag";


+ // GraphQL mutation to update last_seen
+ const EMIT_ONLINE_EVENT = gql`
+ mutation {
+   update_users(
+     _set: {
+       last_seen: "now()"
+     },
+     where: {}
+   ) {
+     affected_rows
+   }
+ }
+`;
```


In the `fetchSession` function, we will create a `setInterval` to update the last_seen of the user every 30 seconds.



```javascript
const fetchSession = async () => {
  // fetch session
  const session = await AsyncStorage.getItem('@todo-graphql:session');
  const sessionObj = JSON.parse(session);
  const { token, id } = sessionObj;

  const client = makeApolloClient(token);

  setClient(client);
+  setInterval(
+    () => client.mutate({
+      mutation: EMIT_ONLINE_EVENT,
+      variables: {
+        userId: id
+      }
+   }),
+    30000
+  );
}
```

Again, we are making use of `client.mutate` to update the `users` table of the database.

Great! Now the metadata about whether the user is online will be available in the backend. Let's now do the integration to display realtime data of online users.
