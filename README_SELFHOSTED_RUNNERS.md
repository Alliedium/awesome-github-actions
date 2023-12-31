 # Self-hosted runners

 A runner is a server that runs your workflows when they're triggered. Each runner can run a single job at a time. GitHub provides Ubuntu Linux, Microsoft Windows, and macOS runners to run your workflows; each workflow run executes in a fresh, newly-provisioned virtual machine. If you need a different operating system or require a specific hardware configuration, you can host your [own runners](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners).
 A self-hosted runner is a system that you deploy and manage to execute jobs from GitHub Actions on GitHub.com. 

## Prerequisites:

  ### 1. Create VMs to use as self-hosted runners

  - To create VMs in `Proxmox` use [scripts](https://github.com/Alliedium/awesome-proxmox/tree/main/vm-cloud-init-shell#vm-provisioning-scripts-based-on-cloud-init-images)

  - To create VMs in `Hyper-V` use [steps](https://github.com/Alliedium/awesome-devops/tree/main/03_virtualization_on_windows_and_zfs_11-aug-2022#create-vms-in-hyper-v)

  ### 2. Install `docker` if you want to run workflow jobs in containers

  -  [`Manjaro`](https://github.com/Alliedium/awesome-linux-config/blob/master/manjaro/basic/install_docker.sh)

  -  [Ubuntu](https://docs.docker.com/engine/install/ubuntu/)


  ### 3. Install [tmate](https://tmate.io/) for debugging

 
## Repository-level runners

### 1. Create a fork from the [main repository](https://github.com/Alliedium/awesome-github-actions)

![fork](./images/fork.png)

### 2. Clone [project](https://github.com/Alliedium/awesome-github-actions)

```
git clone https://github.com/Alliedium/awesome-github-actions.git $HOME/awesome-github-actions
```

Replace the `https://github.com/Alliedium/awesome-github-actions.git`  repository URL with the fork URL you created

### 3. Add a self-hosted Linux runner to the `GitHub` repository.

![add_runner](./images/add_runner.png)

Run commands on the Linux VM `runner` 

- Create a `actions-runner` folder

```
mkdir actions-runner && cd actions-runner
```

- Download the latest runner package      

  If you prefer to use the last version of `action runners`, run the command using tool [lastversion](https://github.com/dvershinin/lastversion):     
```shell
lastversion --assets https://github.com/actions/runner/releases/download --filter "actions-runner-linux-x64-(\d{0,3}\.\d{0,3}\.\d{0,3}).tar.gz"
```
The output of the command is the URL to the actions-runner-linux-x64 resource with the latest version of the downloading file. Use this URL in the following command and the version of the actions-runner from the output in the other commands below. In our case, all commands are provided with the version `2.305.0`.
```shell
curl -o actions-runner-linux-x64-2.305.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.305.0/actions-runner-linux-x64-2.305.0.tar.gz
```

- Extract the installer

```shell
tar xzf ./actions-runner-linux-x64-2.305.0.tar.gz
```

- Create the runner and start the configuration experience:

```shell
./config.sh --url https://github.com/Alliedium/awesome-github-actions --token AL24AM7L6GS3RRSBBTGSSILESXOW6
```
Replace the `https://github.com/Alliedium/awesome-github-actions`  repository URL with the fork URL you created

- Run it!

```
./run.sh
```

### 3. Add a self-hosted Windows runner to the `GitHub` repository.

- In `Windows` machine open `PowerShell`
- Create a folder under the drive root

```
mkdir actions-runner; cd actions-runner
```

- Download the latest runner package

```
Invoke-WebRequest -Uri https://github.com/actions/runner/releases/download/v2.305.0/actions-runner-win-x64-2.305.0.zip -OutFile actions-runner-win-x64-2.305.0.zip

```

- Extract the installer

```
Add-Type -AssemblyName System.IO.Compression.FileSystem ; [System.IO.Compression.ZipFile]::ExtractToDirectory("$PWD/actions-runner-win-x64-2.305.0.zip", "$PWD")
```

- Create the runner and start the configuration experience

```
./config.cmd --url https://github.com/Alliedium/awesome-github-actions --token AL24AM56SRAXQCYI7PF7KW3ES5ILY
```

Replace the `https://github.com/Alliedium/awesome-github-actions`  repository URL with the fork URL you created

- Run it!

```
./run.cmd
```

- Use this YAML in your workflow file for each job
  
```yaml
runs-on: self-hosted
```

### 3. Add a self-hosted docker runner to the `GitHub` repository.

- Navigate to `awesome-github-actions/07-job-matrix/` folder

```
cd $HOME/awesome-github-actions/07-job-matrix
```

- Build a `docker` image


```
docker build -t runner:0.1 \
	--build-arg REPO_URL=https://github.com/Alliedium/awesome-github-actions \
	--build-arg TOKEN='AL24AMZNNIJVECQ34ANMH23ESNYCW' \
    --build-arg LABELS='ubuntu-18.04' \
    --build-arg RUNNER_NAME='docker-runner' \
    -f $HOME/awesome-github-actions/07-job-matrix/Dockerfile \
    $HOME/awesome-github-actions/07-job-matrix
 ```

Replace the `https://github.com/Alliedium/awesome-github-actions`  repository URL with the fork URL you created

- Run `docker` image

```
docker run --name runner -d runner:0.1
```

### 4. Run workflow jobs on self-hosted Linux runners

We used the `$HOME/awesome-github-actions/.github/workflows/07-job-matrix.yml` file as a workflow example for self-hosted runners.

Copy `$HOME/awesome-github-actions/07-job-matrix/workflows/self-hosted-wf.yml` file content to `$HOME/awesome-github-actions/.github/workflows/07-job-matrix.yml` file.

Workflow will execute on any runner that matches all the specified runs-on values
This `runs-on: [ self-hosted, Linux ]` matches all self-hosted Linux runners.

As you can see, the job runs on any self-hosted Linux runner, regardless of the version of the Linux distribution specified in the job

### 5. Run workflow jobs on self-hosted Linux runners that match the distribution version, specified in the job.

Copy `$HOME/awesome-github-actions/07-job-matrix/workflows/self-hosted-labels-wf.yml` file content to `$HOME/awesome-github-actions/.github/workflows/07-job-matrix.yml` file.

The job runs on a runner that matches the version of the Linux distribution specified in the job

### 6. Run workflow jobs in containers on Linux runners on which the docker is installed

Copy `$HOME/awesome-github-actions/07-job-matrix/workflows/self-hosted-containers-wf.yml` file content to `$HOME/awesome-github-actions/.github/workflows/07-job-matrix.yml` file.

### 7. Debugging running workflow

[Tmate](https://chat.openai.com/) is a terminal sharing software that allows users to share their command-line interface (CLI) sessions with others over the internet. 
Copy `$HOME/awesome-github-actions/07-job-matrix/workflows/self-hosted-labels-tmate-wf.yml` file content to `$HOME/awesome-github-actions/.github/workflows/07-job-matrix.yml` file.

```
cp $HOME/awesome-github-actions/07-job-matrix/workflows/self-hosted-labels-tmate-wf.yml $HOME/awesome-github-actions/.github/workflows/07-job-matrix.yml
```

This workflow will fail because running `npm test` without `npm ci`.
`Tmate` will pause the job and establish a terminal session with the runner.

![tmate](./images/tmate.png)

Fix the issue in the terminal. 
To exit the terminal, create a file with `continue` name.

### 8. Deleting runner from the repo

![delete_runner](./images/delete_runner.png)

Navigate to the `actions-runner` folder and in the Runner terminal run the command

```
./config.sh remove --token AL24AM5N3UAUQDRAKNU5XJDETDEBC
```