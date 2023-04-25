# Deploy a NodeJS application with Render

This guide will walk you through the process of deploying an NodeJS application in the [cloud][cloud]. You will use [Render][render], a [Platform-as-a-Service][paas] cloud, to run your application;

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Requirements](#requirements)
- [Create & Push to a Git repository](#create-a-git-repository)
- [Create a Render repository](#create-render-account)
- [Deploy to Render](#deploy-to-render)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Requirements

- [Node.js][node] 12+
- [Git][git]
- A [GitHub][github] account
- A [Render][render] account

## Create & Push to a Git repository

To deploy code on Render, you will need to use [Git][git].

Register a [GitHub][github] account if you haven't already.


Add a `.gitignore` file to ignore the `node_modules` directory (dependencies
will be automatically installed by Render when you push), `package-lock.json` file and `.env` file (if you have an .env file):

The contents of your `.gitignore` file should look like this:

```txt
/node_modules
package-lock.json
# add this if you have an .env file
.env
```

Create a **private** repository on GitHub.

Initialize a Git repository in the application's directory:

```bash
$> git init
```

Commit all the app's files:

```bash
$> git add .
$> git commit -m "Initial commit"
```

Create `main` branch:

```bash
$> git branch -M main
```

Add a new remote repository to your local Git repository:

```bash
$> git remote add origin git@github.com:your-github-username/your-repo-name.git
```

Push to GitHub:

```bash
$> git push -u origin main
```

## Deploy the application to Render

Register a [Render][render] account if you haven't already. If you register through GitHub, you will not have to link the two accounts together later.

![Render: register using an exisiting GitHub Account](./images/render-01-signup.png)

Go to your dashboard and create a new Web Service:

![Render: dashboard](./images/render-02-create.png)

Connect your GitHub repository to Render by selecting the one the contains your app from the list.

![Render: connect to repo](./images/render-03-connect.png)

Name the application, choose the region and enter the commands used to build and start your app. The branch name should automatically be set to "main" or "master", depending on how your repository is setup.

![Render: setup your application](./images/render-04-setup.png)

Select the free plan and finish the creation process.

![Render: end the web service creation process](./images/render-05-plans.png)

Once you submit the form, Render will automatically try to deploy your app. You will be able to see live logs. Pretty cool, but be aware that deploys on the free plan can take a little while. Be patient.

![Render: first deploy](./images/render-06-deploy.png)

The deployment process should eventually succeed. But... **Oh no there seems to be a some weird error in the logs!** Think about it for a second. What could've gone wrong?

```bash
2020-09-13T09:46:13.394704+00:00 app[web.1]: (node:23) UnhandledPromiseRejectionWarning: MongoNetworkError: failed to connect to server [localhost:27017] on first connect [Error: connect ECONNREFUSED 127.0.0.1:27017
2020-09-13T09:46:13.394707+00:00 app[web.1]: at TCPConnectWrap.afterConnect [as oncomplete] (net.js:1141:16) {
2020-09-13T09:46:13.394708+00:00 app[web.1]: name: 'MongoNetworkError'
2020-09-13T09:46:13.394709+00:00 app[web.1]: }]
2020-09-13T09:46:13.394710+00:00 app[web.1]: at Pool.<anonymous> (/app/node_modules/mongodb/lib/core/topologies/server.js:438:11)
2020-09-13T09:46:13.394711+00:00 app[web.1]: at Pool.emit (events.js:315:20)
2020-09-13T09:46:13.394712+00:00 app[web.1]: at /app/node_modules/mongodb/lib/core/connection/pool.js:562:14
2020-09-13T09:46:13.394712+00:00 app[web.1]: at /app/node_modules/mongodb/lib/core/connection/pool.js:995:11
2020-09-13T09:46:13.394712+00:00 app[web.1]: at /app/node_modules/mongodb/lib/core/connection/connect.js:32:7
2020-09-13T09:46:13.394713+00:00 app[web.1]: at callback (/app/node_modules/mongodb/lib/core/connection/connect.js:280:5)
2020-09-13T09:46:13.394713+00:00 app[web.1]: at Socket.<anonymous> (/app/node_modules/mongodb/lib/core/connection/connect.js:310:7)
2020-09-13T09:46:13.394714+00:00 app[web.1]: at Object.onceWrapper (events.js:422:26)
2020-09-13T09:46:13.394714+00:00 app[web.1]: at Socket.emit (events.js:315:20)
2020-09-13T09:46:13.394715+00:00 app[web.1]: at emitErrorNT (internal/streams/destroy.js:92:8)
2020-09-13T09:46:13.394716+00:00 app[web.1]: at emitErrorAndCloseNT (internal/streams/destroy.js:60:3)
2020-09-13T09:46:13.394716+00:00 app[web.1]: at processTicksAndRejections (internal/process/task_queues.js:84:21)
```

Remember this piece of code?

```js
mongoose.connect(
  process.env.DATABASE_URL || "mongodb://localhost/your-app-name"
);
```

At this point, our app is looking for a `DATABASE_URL` variable environment. Unfortunately we have not configured it yet and are therefore trying to connect to our local Mongo instance which is obviously inaccessible from remotely.

We must therefore setup a database elsewhere and provide its URL to Render. Let's start by setting up a [MongoDB Atlas][mongodb-atlas] cluster.

## Create a MongoDB cluster on MongoDB Atlas

Register a free [MongoDB Atlas][mongodb-try] account for a cloud deployment:

![MongoDB Atlas: register an account](./images/mongodb-atlas-01-register.png)

Choose the free shared cluster plan:

![MongoDB Atlas: choose the free plan](./images/mongodb-atlas-02-plan.png)

Create a cluster if one has not already been created for you:

![MongoDB Atlas: create a cluster](./images/mongodb-atlas-03-create-cluster.png)

Configure your cluster. The provider and region are unimportant as long as you
choose one that is free, but you should at least change the default name:

![MongoDB Atlas: configure the cluster](./images/mongodb-atlas-04-configure-cluster.png)

You must configure network access to your cluster to allow connections from the
outside world:

![MongoDB Atlas: configure network access](./images/mongodb-atlas-05-network-access.png)

For the purposes of this guide, you can allow access from anywhere, which should
set the access list entry to `0.0.0.0/0` (i.e. any source IP address is allowed
to access the cluster):

![MongoDB Atlas: allow access from anywhere](./images/mongodb-atlas-06-whitelist-ip.png)

> In a real production environment, you should whitelist the exact IP addresses
> of your servers so that only they can connect to your cluster, for improved
> security.

You must then create a database user to connect with:

![MongoDB Atlas: configure database access](./images/mongodb-atlas-07-database-access.png)

Set the credentials for the new database user:

![MongoDB Atlas: add a database user](./images/mongodb-atlas-08-add-user.png)

To obtain the connection URL for your cluster, go to Clusters and click on your
cluster's Connect button:

![MongoDB Atlas: connect to the cluster](./images/mongodb-atlas-09-connect.png)

You want to connect an application:

![MongoDB Atlas: connect your application](./images/mongodb-atlas-10-connect-app.png)

And you are using a Node.js driver. You should copy the provided connection URL:

![MongoDB Atlas: connect a Node.js application](./images/mongodb-atlas-11-connect-nodejs.png)

Note that the connection URL is in the format
`mongodb+srv://admin:<password>@your-cluster-name.abcd.mongodb.net/<dbname>?retryWrites=true&w=majority`.
There are two placeholders in this URL, `<password>` and `<dbname>`, which you
should replace:

- `<password>` is the password of the database user you just created.
- `<dbname>` is the name of a MongoDB database to connect to. You should name it
  after your project. The exact name is unimportant, since MongoDB will
  automatically create the database the first time you connect to it.

> If you have the `mongo` executable available in your command line, you can
> connect to your new MongoDB cluster from your machine with the command:
>
>     mongosh "mongodb+srv://admin:<password>@your-cluster-name.abcd.mongodb.net/<dbname>?retryWrites=true&w=majority"
>
> (Use the full path to `mongo.exe` on Windows instead of `mongo`.)

## Provide your database URL to your Render application

Now that you have a connection URL for a MongoDB database, you should give it
to your Render application.

This is trivially done by adding `DATABASE_URL` to your application's
environment variables in the Environment section:

![Render: configure environment variables](./images/render-08-variables.png)

These changes will only be taken into account the next time we deploy. New deploys will automatically happen when you push commits on your main branch to GitHub.

```bash
$> git add .
$> git commit -m "<exciting changes>"
$> git push origin master
```

Once your deploy is live, you should be able to test your API at the URL generated by Render. It should look something like: `https://app_name-4vxg.onrender.com`

🎉

[cloud]: https://en.wikipedia.org/wiki/Cloud_computing
[express]: https://expressjs.com
[express-generator]: https://www.npmjs.com/package/generator-express-api-es
[git]: https://git-scm.com
[github]: https://github.com
[render]: https://render.com
[mongodb-atlas]: https://www.mongodb.com/cloud/atlas
[mongodb-try]: https://www.mongodb.com/try
[mongoose]: https://mongoosejs.com
[node]: https://nodejs.org
[paas]: https://en.wikipedia.org/wiki/Platform_as_a_service
[two-hard-things]: https://martinfowler.com/bliki/TwoHardThings.html
