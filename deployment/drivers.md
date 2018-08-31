# Drivers

## Introduction

Masonite comes with several drivers that all work great out of the box and in development. Some drivers are better than others in production and some cannot be used in production because of how servers are setup.

## Upload Drivers

For example, the caching or upload drivers should not store anything on the server itself if you are deploying onto an ephemeral deployment platform like Heroku. In these cases you should use a caching driver like Redis and an upload driver like Amazon S3.

## Session Driver

The session driver has two options out of the box: `memory` and `cookie`. The cookie driver will store session data such as success messages, user data or anything else you store with Session.store\(\).

The memory driver will store all session data in a giant dictionary while the server is running and store all data under the IP address. The data will completely lost when the server stops running. The memory driver is great for development and instances where you need to test features without always deleting cookie data.

You can imagine that if you have the `memory` driver set and 10,000 users then thats 10,000 dictionary keys each containing several values of session data.

For production, this setting should be set to `cookie` to maximize server performance and server resources.

