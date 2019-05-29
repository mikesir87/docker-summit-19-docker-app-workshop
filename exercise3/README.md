# Exercise #3 - Customizing your App with Parameters

> **Time**: Approximately 15 minutes
>
> **Difficulty**: Easy

## Table of Contents

1. [Docker App Parameters](#docker-app-parameters)
1. [Using a Parameter to Change Voting Options](#using-a-parameter-to-change-voting-options)
1. [Upgrading a Deployed Application](#upgrading-a-deployed-application)

## Exercise Objectives

By the end of this exercise, you will have:

- Learned about Docker App parameters
- Added parameters to the voting app application
- Learned how to deploy an update to an already-deployed application


## Docker App Parameters

In addition to using environment variables, Docker App allows you to define parameters for an application that can be overridden at deploy-time. Examples of parameters might include port mappings, cpu/memory limits, number of replicas, label definitions, or network names. _Almost_ everything in the Docker App compose file can be parameterized, **except for the image names**.


## Using a Parameter to Change Voting Options

By default, the vote and results services let you vote between Dogs and Cats. However, wouldn't it be neat if we could change that? Conveniently, both the `vote` and `results` services allow the definition of two environment variables (`OPTION_A` and `OPTION_B`) to let us change the voting choices. This sounds like a great opportunity for parameters!

1. In both the `vote` and `results` services, add environment variables named `OPTION_A` and `OPTION_B` with a value of `${options.A}` and `${options.B}`. The values are simply placeholders for the parameters.

    <details>
      <summary>Solution</summary>
    
    ```yaml
    services:
      vote:
        environment:
          OPTION_A: ${options.A}
          OPTION_B: ${options.B}
      results:
        environment:
          OPTION_A: ${options.A}
          OPTION_B: ${options.B}
    ```
    </details>

2. Before setting the parameter, run `docker app inspect` to see what happens.

    <details>
      <summary>Full Output</summary>
    
    ```console
    $ docker app inspect
    inspect failed: Action "com.docker.app.inspect" failed: failed to load Compose file: invalid interpolation format for services.vote.environment.OPTIONS_A: "required variable options.A is missing a value". You may need to escape any $ with another $.
    ```
    </details>

    What happened? The command fails because "required variable options.A is missing a value." In other words, **all parameters are required to have default values.** Those default values are specified in the `parameters.yml` file.

3. Specify the default values for `options.A` and `options.B` in the `parameters.yml` file. Let's set `options.A` to "Cats" and `options.B` to "Dogs". Remember that this file is a YAML file.

    <details>
      <summary>Solution</summary>
    
    ```yaml
    options:
      A: Cats
      B: Dogs
    ```
    </details>

4. Now, re-run the `docker app inspect` command to look at the output. You should see the parameters with their default values in the output now!

    <details>
      <summary>Full output</summary>
    
    ```console
    $ docker app inspect
    voting-app 0.1.0

    Maintained by: root

    Services (5) Replicas Ports Image
    ------------ -------- ----- -----
    db           1              postgres:9.4
    worker       1              dockersamples/examplevotingapp_worker
    result       1        5001  mikesir87/examplevotingapp_result
    vote         2        5000  mikesir87/examplevotingapp_vote
    redis        1              redis:alpine

    Networks (2)
    ------------
    backend
    frontend

    Volume (1)
    ----------
    db-data

    Parameters (2) Value
    -------------- -----
    options.A      Cats
    options.B      Dogs
    ```
    </details>

5. Now, try running the same `docker app inspect`, but add the `-s` flag to set options.A to "Moby" and optionB to "Molly". You should see the parameter values now have the replaced values.

    <details>
      <summary>Full output</summary>
    
    ```console
    $ docker app inspect -s options.A=Moby -s options.B=Molly
    voting-app 0.1.0

    Maintained by: root

    Services (5) Replicas Ports Image
    ------------ -------- ----- -----
    db           1              postgres:9.4
    worker       1              dockersamples/examplevotingapp_worker
    result       1        5001  mikesir87/examplevotingapp_result
    vote         2        5000  mikesir87/examplevotingapp_vote
    redis        1              redis:alpine

    Networks (2)
    ------------
    backend
    frontend

    Volume (1)
    ----------
    db-data

    Parameters (2) Value
    -------------- -----
    options.A      Moby
    options.B      Molly
    ```
    </details>

6. At this point, go ahead and deploy the Docker App with these parameters (options.A=Moby and options.B=Molly)

    <details>
      <summary>Full output</summary>
    
    ```console
    $ docker app deploy voting-app -s options.A=Moby -s options.B=Molly
    Creating network back-tier
    Creating network front-tier
    Creating service voting-app_redis
    Creating service voting-app_db
    Creating service voting-app_worker
    Creating service voting-app_results
    Creating service voting-app_vote
    Application "voting-app" installed on context "default"
    ```
    </details>

    Once it's deployed, go ahead and check out the vote and results apps. You should see that we're now using Moby vs Molly!


## Upgrading a Deployed Application

With the app deployed, let's change the settings by "upgrading" the application bundle. To do so, we can use the `docker app upgrade` command. While we will change settings in the upgrade here, you can use this command to actually deploy an updated version of the app.

1. Let's pretend that Moby had gotten more votes, but we really want Molly to win (since she's cuter anyways)! Let's swap the values, making `options.A=Molly` and `options.B=Moby`.

    <details>
      <summary>Solution/Output</summary>
    
    ```console
    $ docker app upgrade voting-app -s options.A=Molly -s options.B=Moby
    Updating service voting-app_results (id: tpugiytt4eq9p88lvb8900pmq)
    Updating service voting-app_vote (id: d49hxltgvg5faie0kc735oy42)
    Updating service voting-app_redis (id: x9hpof20yumf2gv3mbbd9g1i5)
    Updating service voting-app_db (id: nwssvpk4r8gklcfnvd47w7tzx)
    Updating service voting-app_worker (id: qoyl03yaxtdyefb5oh6u9m698)
    Application "voting-app" upgraded on context "default"
    ```
    </details>

    After a moment, you should be able to open either service and see the options have been swapped. Now, Molly is guaranteed to win the vote! :tada:


## More Practice

While we added parameters to change the options, we can set parameters for almost anything in the compose file. As practice, turn the exposed ports into parameters (`vote.exposedPort` and `results.exposedPort`) and allow them to be overridden. Then, actually override them and validate it worked!
