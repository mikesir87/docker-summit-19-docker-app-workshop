# Exercise #2 - Docker App Hello World

> **Time**: Approximately 30 minutes
>
> **Difficulty**: Easy

## Table of Contents

1. [Docker Application Overview](#docker-application-overview)
1. [Initialize the Application](#initialize-the-application)
1. [Inspecting our Docker Application](#inspecting-our-docker-application)
1. [Validating our Application](#validating-our-application)
1. [Deploying the Docker App](#deploying-the-docker-app)
1. [Viewing Installed Applications](#viewing-installed-applications)
1. [Uninstalling the Docker App](#uninstalling-the-docker-app)

## Exercise Objectives

By the end of this exercise, you will have:

- Created a Docker Application package
- Learned about the application package components
- Learned how to inspect and validate the application
- Deployed the Docker App using the locally-sourced app bundle
- Learned how to see existing app installations
- Removed an app installation
- Tagged and pushed the Docker App to Docker Hub
- Uninstalled a Docker App



## Docker Application Overview

Application packages are a construction above compose files to improve application lifecycle and workflow, from development to test to production. An application package is a set of 3 documents:

1. A `metadata.yml` file describing the application metadata (name, version, description, ...)
2. A `docker-compose.yml` file describing the application structure (what we have right now)
3. A `parameters.yml` file with key/value parameters (we will use this in the next exercise)

You can also include any of your own custom files, including config files. These additional files are called `attachments`.

## Initialize the Application

1. Let's look at the `docker app init` command and see it's options.

    ```bash
    $ docker app init --help

    Usage:  docker app init APP_NAME [--compose-file COMPOSE_FILE] [--description DESCRIPTION] [--maintainer NAME:EMAIL ...] [OPTIONS]

    Start building a Docker Application package. If there is a docker-compose.yml file in the current directory it will be copied and used.

    Examples:
    $ docker app init myapp --description "a useful description"

    Options:
          --compose-file string      Compose file to use as application base (optional)
          --description string       Human readable description of your application (optional)
          --maintainer stringArray   Name and email address of person responsible for the application
                                    (name:email) (optional)
          --single-file              Create a single-file Docker Application definition
    ```

2. Let's now create our Docker app! We will use the `docker app init` command and specify the description and maintainer. Feel free to change these values. Be sure to run this in the same directory as the `docker-compose.yml` file we were working on in the last exercise. Otherwise you can specify a path to your compose file using `--compose-file path/to/my/docker-compose.yml`.

    ```bash
    # Swap out the maintainer email with your own email address
    $ docker app init voting-app --description "Voting App" --maintainer "dapworkshop:dapworkshop@docker.com"
    Created "voting-app.dockerapp"
    ```

3. If you run `tree`, you'll see a new directory. The name is your app name with a `.dockerapp` suffix. Let's look inside the directory.

    ```bash
    $ tree
    .
    ├── docker-compose.yml
    └── voting-app.dockerapp
        ├── docker-compose.yml
        ├── metadata.yml
        └── parameters.yml

    1 directory, 4 files
    ```

4. The compose file is a copy of the file you were working with earlier. If you open the `metadata.yml` file, you'll see the config we specified during initialization.

    ```bash
    $ cat voting-app.dockerapp/metadata.yml
    # Version of the application
    version: 0.1.0
    # Name of the application
    name: voting-app
    # A short description of the application
    description: Voting App
    # List of application maintainers with name and email for each
    maintainers:
      - name: dapworkshop
        email: dapworkshop@docker.com
    ```



## Inspecting our Docker Application

We can use the `docker app inspect` command to get a quick output of all of the services, number of replicas, ports, and the image being used. This could allow an ops admin to quickly check all the elements before deploying to production, without having to manually parse the compose file.

1. Run the `docker app inspect` command to inspect our application. You shouldn't need to provide the app name, as there is only one in the directory.

    ```bash
    $ docker app inspect
    voting-app 0.1.0

    Maintained by: dapworkshop <dapworkshop@docker.com>

    Voting App

    Services (6) Replicas Ports   Image
    ------------ -------- -----   -----
    vote         2        5000    mikesir87/examplevotingapp_vote
    redis        1                redis:alpine
    worker       1                dockersamples/examplevotingapp_worker
    db           1                postgres:9.4
    results      1        5001    mikesir87/examplevotingapp_result

    Networks (3)
    ------------
    frontend
    backend
    proxy

    Volume (1)
    ----------
    db-data
    ```



## Validating our Application

Before we're ready to ship our application, we should validate it to make sure everything is set. Specifically, validation does the following:

- Ensures our compose file is valid (correct syntax, etc.)
- Ensures required metadata is provided (name, version) and is the correct format
- Ensures all parameters (which we'll talk about next) have default values

1. Run the `docker app validate` command to make sure our application is valid.

    ```bash
    $ docker app validate
    Validated "/root/voting-app.dockerapp"
    ```

2. Let's make a change to invalidate the application. In the `voting-app.dockerapp/metadata.yml` file, comment out the `version` field using a `#`. Then, revalidate.

    ```bash
    $ docker app validate
    failed to validate metadata:
    - version: version is required
    ```

    We have an error! Go ahead and fix it and revalidate.



## Deploying the Docker App

There are two different ways Docker App can locate an application bundle

- **Locally** - Docker App will use a merged or split application bundle definition found on your local filesystem
- **Remote** - Docker App will pull the application bundle from Docker Hub (or another registry)

For our first deployment, we will simply use the locally available app definitions.

1. Let's first look at the `docker app install` options. Run `docker app install --help` to see all options.

    <details>
      <summary>Full console output</summary>

    ```console
    $ docker app install --help

    Usage:  docker app install [APP_NAME] [--name INSTALLATION_NAME] [--target-context TARGET_CONTEXT] [OPTIONS]

    Install an application.
    By default, the application definition in the current directory will be
    installed. The APP_NAME can also be:
    - a path to a Docker Application definition (.dockerapp) or a CNAB bundle.json
    - a registry Application Package reference

    Aliases:
      install, deploy

    Examples:
    $ docker app install myapp.dockerapp --name myinstallation --target-context=mycontext
    $ docker app install myrepo/myapp:mytag --name myinstallation --target-context=mycontext
    $ docker app install bundle.json --name myinstallation --credential-set=mycredentials.yml

    Options:
          --credential-set stringArray    Use a YAML file containing a credential set or a credential set
                                          present in the credential store
          --insecure-registries strings   Use HTTP instead of HTTPS when pulling from/pushing to those registries
          --kubernetes-namespace string   Kubernetes namespace to install into (default "default")
          --name string                   Installation name (defaults to application name)
          --orchestrator string           Orchestrator to install on (swarm, kubernetes)
          --parameters-file stringArray   Override parameters file
          --pull                          Pull the bundle
      -s, --set stringArray               Override parameter value
          --target-context string         Context on which the application is installed (default: <current-context>)
          --with-registry-auth            Sends registry auth
    ```
    </details>


2. Let's deploy the application bundle using the `docker app install` command. Specify the name of your app as `voting-app`

    <details>
      <summary>Solution/Full Output</summary>

      ```console
      $ docker app install voting-app
      Creating network front-tier
      Creating network back-tier
      Creating service voting-app_vote
      Creating service voting-app_redis
      Creating service voting-app_db
      Creating service voting-app_worker
      Creating service voting-app_result
      Application "voting-app" installed on context "default"
      ```
    </details>

    :tada: The application stack is now deployed! Hooray!


## Viewing Installed Applications

Another handy tool is the `docker app ls` command, which allows us to see all currently installed applications.

1. On the **manager1** node, run the `docker app ls` command. We should see our newly installed application!

    <details>
      <summary>Full output</summary>
      
    ```console
    $ docker app ls
    INSTALLATION APPLICATION        LAST ACTION RESULT  CREATED   MODIFIED  REFERENCE
    voting-app   voting-app (0.1.0) install     success 2 minutes 2 minutes
    ```
    </details>


## Uninstalling the Docker App

Let's practice uninstalling our deployed application.

1. Let's look at the full options for `docker app uninstall`.

    <details>
      <summary>Full output</summary>
      
    ```console
    $ docker app uninstall --help
    Usage:  docker app uninstall INSTALLATION_NAME [--target-context TARGET_CONTEXT] [OPTIONS]

    Uninstall an application

    Examples:
    $ docker app uninstall myinstallation --target-context=mycontext

    Options:
          --credential-set stringArray   Use a YAML file containing a credential set or a credential set
                                        present in the credential store
          --force                        Force removal of installation
          --target-context string        Context on which the application is installed (default: <current-context>)
          --with-registry-auth           Sends registry auth
    ```
    </details>

    We'll see that we need to specify the `INSTALLATION_NAME` in order to remove the app.

2. Use the `docker app uninstall` command to remove the application.

    <details>
      <summary>Solution/Full Output</summary>
    
    ```console
    $ docker app uninstall voting-app
    Removing service voting-app_db
    Removing service voting-app_redis
    Removing service voting-app_result
    Removing service voting-app_vote
    Removing service voting-app_worker
    Removing network back-tier
    Removing network front-tier
    Application "voting-app" uninstalled on context "default"
    ```
    </details>

    :tada: It's gone now!

3. To verify, you can run `docker app ls` and validate that it's gone.

    <details>
      <summary>Full output</summary>
    
    ```console
    $ docker app ls
    INSTALLATION APPLICATION LAST ACTION RESULT CREATED MODIFIED REFERENCE
    ```
    </details>


## Pushing our Docker App

Now that we've learned how to deploy a locally-sourced application, let's push our app to Docker Hub and deploy from there!

1. Let's first take a look at the various options and flags on the `docker app push` command.

    <details>
      <summary>Full output</summary>
    
    ```console
    $ docker app push --help

    Usage:  docker app push [APP_NAME] --tag TARGET_REFERENCE [OPTIONS]

    Push an application package to a registry

    Examples:
    $ docker app push myapp --tag myrepo/myapp:mytag

    Options:
          --insecure-registries strings   Use HTTP instead of HTTPS when pulling from/pushing to those registries
          --platform strings              For multi-arch service images, only push the specified platforms
      -t, --tag string                    Target registry reference (default: <name>:<version> from metadata)
    ```
    </details>

2. When pushing, we will tag the application to include your Docker Hub account (`<your-hub-username>/voting-app.dockerapp`) and specify the tag as the current version (which defaulted to `0.1.0`). While not required, we do encourage you to keep the `.dockerapp` suffix on the image name to help make it more obvious that it is a repo containing a Docker App bundle.

    <details>
      <summary>Solution/Full Output</summary>
    
    ```console
    $ docker app push --tag mikesir87/voting-app.dockerapp:0.1.0
    docker.io/mikesir87/voting-app.dockerapp:0.1.0-invoc
    mikesir87/examplevotingapp_vote
    sha256:a0d4d29d...: Skip (already present)
    redis:alpine
    sha256:ef67270b...: Skip (already present)
    postgres:9.4
    sha256:094e3a9e...: Skip (already present)
    dockersamples/examplevotingapp_worker
    sha256:55753a7b...: Skip (already present)
    mikesir87/examplevotingapp_result
    sha256:69198c25...: Skip (already present)
    WARN[0003] reference for unknown type: application/vnd.cnab.config.v1+json
    Successfully pushed bundle to docker.io/mikesir87/voting-app.dockerapp:0.1.0. Digest is sha256:ca706ede7e387173cf28b20e65336299873c072deb422c35a0fd57379b46932e.
    ```
    </details>

    The output may look slightly different, as we had already previously pushed the app before capturing the output to display above. In addition, you'll see a different sha256, which can be used for addressing apps too. However, you should see a "Successfully pushed bundle" message at the end.

3. Now, let's deploy our remotely-pushed application bundle to the cluster. All we have to do is specify the remote application as the application name and Docker App will fetch the image and then use the bundled name as the _actual_ name of the installed app (which can be overridden by using the `--name` option).

    <details>
      <summary>Solution/Full Output</summary>
    
    ```console
    $ docker app install mikesir87/voting-app.dockerapp:0.1.0
    Creating network back-tier
    Creating network front-tier
    Creating service voting-app_db
    Creating service voting-app_worker
    Creating service voting-app_result
    Creating service voting-app_vote
    Creating service voting-app_redis
    Application "voting-app" installed on context "default"
    ```
    </details>

    And that's it! You can go onto either of the Swarm nodes and use the port badges to open the app (may take a few seconds for it pull the images and start the containers).

5. Once you've validated that it works, go ahead and uninstall the application so we have a clean slate for our next exercise.

    <details>
      <summary>Output</summary>
    
    ```console
    $ docker app uninstall voting-app
    Application "voting-app" uninstalled on context "default"
    ```
    </details>