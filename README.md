# harness-lab

TODO: Make sure all styling and formatting are consistent.

TODO: Make sure the instructions are clear and valid.

TODO: Add screenshots where text instructions are not enough.

This lab will guide you through setting up and using Harness Open Source (referred to as "Harness" from now on), with a focus on managing a project, using GitSpaces, creating pipelines, and setting up an artifact registry. By the end of the lab, you'll be able to create a project, import a repository, work with GitSpaces, and automate build pipelines.

## Prerequisites

Before starting, ensure you have the following installed on your local machine:

- Docker
- VS Code (optional but recommended for working with GitSpaces)

If Docker is not already installed on your system, you can set it up using [Colima](https://github.com/abiosoft/colima?tab=readme-ov-file#installation), a lightweight container runtime for macOS and Linux. Note that you'll still need the Docker CLI (Docker client) to interact with Colima, so ensure that the Docker client is installed and configured on your machine.

## Setup and Installation

1. Create a common network:

```bash
docker network create harness
```

2. Run the following command to start a Harness instance that uses the `harness` network:

```bash
docker run -d \
  --network=harness \
  -e GITNESS_CI_CONTAINER_NETWORKS=harness \
  -e GITNESS_URL_REGISTRY=http://gitness:3000 \
  -p 3000:3000 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /tmp/gitness:/data \
  --name harness \
  --restart always \
  harness/harness
```

This command starts the Harness server, exposes it on port 3000, and mounts necessary volumes for Docker and persistent data storage.

3. Follow these steps to create an admin user:
  a. Once the container is running, open http://localhost:3000 in your browser.
  b. Select **Sign Up**.
  c. Enter a User ID (`admin`), Email (`admin@example.com`), and Password (`changeit`).
  d. Select **Sign Up**. (You might see a warning to change your password. You can ignore that warning.)

## Project and Repository

### New Project

1. Select **New Project**.
2. Enter a project Name (**harness-lab**) and optional Description (**Open source code hosting, pipelines, artifact registry, dev environments**).
3. Select **Create Project**.
> [!NOTE]
> Harness can also [import projects](https://docs.gitness.com/administration/project-management#import-a-project) from external sources (such as GitLab groups or GitHub organizations).
4. You can organize your work in Harness by creating labels to categorize pull requests, artifacts, and more. To get started, add three labels to your project: "dev," "staging," and "prod." You can also assign specific values to each of these labels, helping to streamline project management and tracking.

### Import a Repository

1. Click on the drop-down under **Repositories**, and select "Import Repository".
2. Use `harness-community` for the organization and `podinfo` for the repository.
3. Click **Import Repository**.

### Webhook

You can send data to HTTP endpoints from actions in your repository, such as opened pull requests, new branches, and more. For this exercise, you’ll use [webhook.site](https://webhook.site) - a website that offers unique, random URLs to instantly receive and inspect all incoming HTTP requests and webhooks in real-time, facilitating testing and debugging. For free webhook.site users, the URL and its data are kept for 7 days. You can close the browser tab and still return to the same unique webhook.site URL.

1. Navigate to webhook.site and copy your unique URL.
2. Click on **Webhooks** under the podinfo repository and then **+ New Webhook**.
3. Give this webhook a name: `trigger_on_branch_created`.
4. Paste the unique URL you copied under Payload URL. You can leave out the Secret.
5. Choose **Let me select individual events** and select **Branch created**.
6. Click **Create Webhook**.

Continue to the next section to push a new branch. Once a new branch is pushed, you’ll see the trigger in action on this site.

### Create a Branch to Trigger Webhook

TODO: In the previous workshop, we create a new user **developer** and use that profile to raise a PR. But that involves multiple switching back and forth between **admin** and **developer** profiles which delayed the workshop. We need to discuss the best approach to show key features and balance the workshop duration.

1. Within the podinfo repository, create a new branch named "feature".
2. On webhook.site, you should see a notification indicating that the webhook was triggered.

The response will look something like this:

```json
{"trigger":"branch_created","repo":{"id":1,"path":"testproj/podinfo","identifier":"podinfo","description":"","default_branch":"master","url":"http://159.203.33.47:3000/testproj/podinfo","git_url":"http://159.203.33.47:3000/git/testproj/podinfo.git","git_ssh_url":"ssh://git@159.203.33.47:3022/testproj/podinfo.git","uid":"podinfo"},"principal":{"id":4,"uid":"admin","display_name":"Administrator","email":"mail@example.com","type":"user","created":1724895740977,"updated":1724895740977},"ref":{"name":"refs/heads/feature3","repo":{"id":1,"path":"testproj/podinfo","identifier":"podinfo","description":"","default_branch":"master","url":"http://159.203.33.47:3000/testproj/podinfo","git_url":"http://159.203.33.47:3000/git/testproj/podinfo.git","git_ssh_url":"ssh://git@159.203.33.47:3022/testproj/podinfo.git","uid":"podinfo"}},"sha":"dbf831f84f486243998a2f86cda9fa76d9f1b748","head_commit":{"sha":"dbf831f84f486243998a2f86cda9fa76d9f1b748","message":"Updated pipeline testpipe","author":{"identity":{"name":"Administrator","email":"mail@example.com"},"when":"2024-09-03T17:38:34Z"},"committer":{"identity":{"name":"Gitness","email":"system@gitness.io"},"when":"2024-09-03T17:38:34Z"},"added":[],"removed":[],"modified":[]},"commit":{"sha":"dbf831f84f486243998a2f86cda9fa76d9f1b748","message":"Updated pipeline testpipe","author":{"identity":{"name":"Administrator","email":"mail@example.com"},"when":"2024-09-03T17:38:34Z"},"committer":{"identity":{"name":"Gitness","email":"system@gitness.io"},"when":"2024-09-03T17:38:34Z"},"added":[],"removed":[],"modified":[]},"old_sha":"0000000000000000000000000000000000000000","forced":false}
```
3. Within the **feature** branch, modify `pkg/version/version.go` to update the app version to **6.6.2**. 

TODO: Add secret scanning to this workshop. It's already documented in the [Gitness Lab](https://github.com/harness-community/gitness-lab?tab=readme-ov-file#secret-scanning). We can use Harness PAT instead of AWS creds.

## GitSpaces

### Create a GitSpace for VS Code Desktop

1. Create a GitSpace for the `podinfo/master` branch and open it in VS Code Desktop.
2. Build the binary for podinfo by running:

   ```bash
   go build ./cmd/podinfo
   ```

3. Run the app:

   ```bash
   ./podinfo
   ```

4. Open your browser and navigate to [http://localhost:9898](http://localhost:9898) to see the app running version `6.6.1`.

### Create gitspaces for VS Code Browser

1. Create a gitspaces for the `podinfo/feature` branch and open it in VS Code Browser.
2. Build the binary for podinfo by running:

   ```bash
   go build ./cmd/podinfo
   ```

3. Run the app:

   ```bash
   ./podinfo
   ```

4. Open your browser and navigate to [http://localhost:9898](http://localhost:9898) to see the app running version `6.6.2`.

5. Stop both GitSpaces once you’ve verified the versions are running correctly.

TODO: Mention that these gitspaces instances are already configured with git credentials from Harness Open Source so you don't have to configure git credentials.

## Pipeline

### Create a New Pipeline

1. In the podinfo repository, go to **Pipelines** and click **+ New Pipeline**.
2. Click "Generate" to let Harness automatically create a pipeline for your Go project. This pipeline should install dependencies, build the app, and run tests.
3. Click "Run" to execute the pipeline and ensure all steps complete successfully.

TODO: We have a pipeline in later section that automates the build, test, and push process for our application. Are we good with these two pipelines OR do we need to show some additional pipeline features? Keep the time constraint in mind.

## Artifact Registry

TODO: (This is a docs update assuming Harness Open Source instance is running on port 3000) 
- For local installations, add `host.docker.internal:3000` as an insecure-registry in your docker config.
- For cloud VM installations, add `YOUR_IP:3000` as an insecure-registry in your docker config.

### Create an Artifact Registry

1. Navigate to **Artifact Registries --> + New Artifact Registry** and create a new artifact registry named "harness-reg".
2. In the registry settings, click "Set up client" to retrieve the connection credentials to that registry. Make a note of the username.
3. Click "Generate token" and make a note of the token.

### Add Secrets

1. Navigate to Secrets in the Harness dashboard.
2. Add two secrets:
   - `docker_username`: Use the registry username from the previous step.
   - `docker_password`: Use the generated token from the previous step.

### Create a Pipeline for Build, Test, and Push

Create a new pipeline called "build-test-push" and use the following YAML configuration:

```yaml
kind: pipeline
spec:
  stages:
    - name: build-test-push-scan
      spec:
        platform:
          arch: amd64
          os: linux
        steps:
          - name: go_install
            spec:
              container:
                image: golang:1.23
            script:
              - go install ./...
            type: run
          - name: go_test
            spec:
              container:
                image: golang:1.23
            script:
              - go test -v ./...
            type: run
          - name: go_build_push
            type: plugin
            spec:
              name: docker
              inputs:
                insecure: true
                repo: host.docker.internal:3000/harness-lab/harness-reg/podinfo
                registry: host.docker.internal:3000
                username: ${{ secrets.get("docker_username") }}
                password: ${{ secrets.get("docker_password") }}
                tags: ${{ build.number }}
      type: ci
version: 1
```

### Verify

Check the artifact registry to ensure the new image has been successfully pushed.

TODO: Did the above pipeline work? If not, please troubleshoot. This pipeline works on instances launched on cloud VMs.

TODO: How can we show other key artifact registry features? Please update this workshop with those features.

## Add Security Testing

In this step, we use a popular open source image scanning tool called [Grype](https://github.com/anchore/grype). The goal is to integrate Grype into the pipeline to scan the newly built image for vulnerabilities before promoting it to the prod environment.

### Updated Pipeline

Modify the pipeline to include a new step for Grype scanning:

```YAML
kind: pipeline
spec:
  stages:
    - name: build-test-push-scan
      spec:
        platform:
          arch: amd64
          os: linux
        steps:
          - name: go_install
            spec:
              container:
                image: golang:1.23
              script:
                - go install ./...
            type: run
          - name: go_test
            spec:
              container:
                image: golang:1.23
              script:
                - go test -v ./...
            type: run
          - name: go_build_push
            type: plugin
            spec:
              name: docker
              inputs:
                insecure: true
                repo: host.docker.internal:3000/harness-lab/harness-reg/podinfo
                registry: host.docker.internal:3000
                username: ${{ secrets.get("docker_username") }}
                password: ${{ secrets.get("docker_password") }}
                tags: ${{ build.number }}
          - name: Grype_Image_Scan
            type: run
            when: |
              build.event == "pull_request"
              and
              build.target == "master"
            spec:
              container: alpine
              script: |
                apk add --no-cache curl            
                curl -sSfL https://raw.githubusercontent.com/anchore/grype/main/install.sh | sh -s -- -b /tmp/grype-bin            
                /tmp/grype-bin/grype host.docker.internal:3000/harness-lab/harness-reg/podinfo:${{ build.number }}
                echo "Image scan completed!"
      type: ci
version: 1
```

### Create a Pull Request to trigger the pipeline

From the "feature" branch, create a new Pull Request (PR) to the "master" branch. The conditions for the **Grype_Image_Scan** step will be met, and the pipeline will execute the Grype scan as part of the CI process.

### Pipeline Execution

The default trigger should automatically kick off the "build-test-push" pipeline. Ensure that a new image is built and pushed to the image registry.

TODO: The grype image scan step might not work. This is still a work in progress. If this step doesn't work, please troubleshoot. Is the pipeline able to pull the image? If the pipeline fails due to grype finding critical vulnerabilities, we can update the fail-on-severity threshold level.

## Deployment

TODO: Add instructions on how to start a local k3d cluster.

TODO: In the [Gitness Lab](https://github.com/harness-community/gitness-lab), we used the helm3 plugin to deploy the app. But that was not the correct approach since we're not generating a helm chart for this app. Please suggest which drone/gitness plugin can be used to deploy the image from artifact registry to the local k3d cluster. Please add a working example.

### Notifications

TODO: We can reuse [Notify on build or deployment failure](https://github.com/harness-community/gitness-lab) from previous lab.
