# Yoke CLI

The yoke CLI is the command line companion to the Aerobatic cloud platform. It is used to create new apps, deploy apps, run the local development server, and more. It's easily installed via npm:

```bash
npm install -g yoke-cli
```

## Commands

### login
Required the first time `yoke` is run. It will prompt you for your `userId` and `secretKey` which are written to a `.aerobatic` file in your user home directory. 

```
yoke login
```

### create-app
Create a new app. You will be presented with the option to either create a new app from one of our canned starter templates or from an existing codebase. 

```
yoke create-app
```

### sim
Run the app in [simulator mode](/docs/simulator-mode). The index page will first be uploaded to the cloud, then a local development server is started on localhost. Your asset URLs in the index page will be pointed back to //localhost:3000. The following options can optionally be passed to the sim command:

#### --open, -o
Automatically launch a browser to the simulator URL.

#### --port
Override the port number which defaults to 3000

#### --release
Run the simulator in release mode. This will cause your build step to be run first. Note this doesn't deploy anything to production, it's a way for you to test your app with the compiled assets as they will be served when you actually do deploy.

### deploy
Deploy a new version of the app to production. The command will present prompts for the version name and an optional deploy message. This works well when a developer is deploying manually, but for automated deployments from a CI system the unattended flag can be set which will instead read inputs as command line args.

#### --unattended, -u
Indicates `yoke` is being invoked by an unattended process.

#### --version, -v
The name of this version. By default the version is read from package.json.

#### --message, -m
An optional deploy message

#### --force, -f
Immedietely direct all production traffic to this new version. This happens automatically if traffic control is not enabled on the app.

#### --userId, -u
Pass in the userId. Generally only used by a CI process. Developers will find it more convenient to run the `login` command to avoid having to re-enter the userId and secretKey repeatedly.

#### --secretKey, -k
The secretKey corresponding to the `userId`.

### bind-app
Bind an existing codebase to an existing Aerobatic app. Simply writes the _aerobatic section to the package.json.










