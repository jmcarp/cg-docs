---
menu:
  main:
    parent: advanced
title: Running One Off Tasks
aliases:
- /getting-started/cf-ssh/
---

There are a couple of ways to run one off tasks in Cloud Foundry. The most reliable way to do one-off tasks is by deploying a short-lived app.

## Disclaimers

* Both require a "complete" [application manifest](http://docs.cloudfoundry.org/devguide/deploy-apps/manifest.html) (which needs to include all of your service bindings, etc.) &mdash; if you don't have one yet, generate one from an existing app:

    ```bash
    cf create-app-manifest <APP_NAME> -p manifest.yml
    ```

* Both will spin up an instance of your environment uploading your **local** code.
    * Note that this may not be in sync with your live app.
    * You can leverage this fact to include files that your one-off task might need, such as a data file for importing.

## Short-lived app

The idea here is that we are going to deploy a new application, but running the one-off task, rather than starting a server or whatever it would normally do. Note that this will not work for any command that is interactive.

1. Make a copy of your application manifest:

    ```bash
    cp manifest.yml task_manifest.yml
    ```

1. Modify the `task_manifest.yml`:
    * Change the `name` value to be `task-runner` (or something descriptive).
    * Remove the following attributes, if present:
        * `domain`
        * `domains`
        * `host`
        * `hosts`
        * `random-route`
    * Add [`no-route: true`](https://docs.cloudfoundry.org/devguide/deploy-apps/manifest.html#no-route).
    * Set [`command`](https://docs.cloudfoundry.org/devguide/deploy-apps/manifest.html#start-commands) to be whatever you want to execute.
1. Deploy the one-off app:

    ```bash
    cf push -f task_manifest.yml
    ```

1. If needed, use [`cf files`](http://cli.cloudfoundry.org/en-US/cf/files.html) to collect any artifacts.
1. Run `cf delete task-runner` (or whatever you named the one-off app) to clean it up.

## CF SSH

Another way to run one-off commands is via `cf-ssh`. `cf-ssh` is a shared ssh session with an application container that you can connect to. This allows you to debug the environment and your application without disturbing a running container.

Our `cf-ssh` is customized to our Cloud Foundry installation, so please **do not use the community version or [`cf ssh`](https://docs.cloudfoundry.org/devguide/deploy-apps/ssh-apps.html)**.

### Installation

1. [Download `cf-ssh` for your environment](https://github.com/18F/cloud-foundry-scripts/releases/).
1. Run

    ```bash
    cd ~/Downloads
    chmod a+x cf-ssh
    if [ -w /usr/local/bin ]; then mv cf-ssh /usr/local/bin; else sudo mv cf-ssh /usr/local/bin; fi
    ```

### Usage

1. In your project folder, run

    ```bash
    cf-ssh --verbose -f manifest.yml
    ```

1. The process takes between 2 to 10 minutes to start the session since it is compiling your application in the background. When it completes, you will see a command prompt.
1. When done, run `exit`.