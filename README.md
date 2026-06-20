## AWS Docker Compose Build Project (Week 15)

### Project Overview

This project demonstrates how to build and deploy a custom Docker image using Docker Compose on an AWS EC2 instance.

A simple HTML website was containerized using a Dockerfile, deployed through Docker Compose, and made accessible through a public EC2 instance.

## Architecture
<img width="2880" height="1800" alt="Week15 Architecture" src="https://github.com/user-attachments/assets/c4eb7229-c994-4549-b1c1-d32ba7546ca7" />


### Technologies Used

•	AWS EC2
•	Docker
•	Docker Compose
•	Docker Buildx
•	Git
•	GitHub
•	Linux (Amazon Linux 2023)
•	Nginx

### Project Files

Dockerfile
Builds a custom Docker image using the Nginx base image and copies the website content into the container.

docker-compose.yml
Defines and deploys the application container using Docker Compose.

index.html
Simple web application served by Nginx.

### Deployment Steps

Install Docker

sudo dnf install docker -y
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker ec2-user


Verify Docker

docker version


Install Docker Compose
Docker Compose was installed manually because the standard package was unavailable on Amazon Linux 2023.

Verify Docker Compose

docker compose version


Build and Deploy

docker compose up -d


Verify Running Container

docker ps


### Website Verification

The deployed website was successfully accessed through the EC2 public IP address.

### Output:

<img width="1440" height="900" alt="Screenshot 2026-06-13 at 3 46 50 AM" src="https://github.com/user-attachments/assets/84f4083c-448c-40ab-8fd6-d79d2e34441c" />
Note: During documentation review, the website title initially referenced a “Multi-Container Project.” The title was later corrected to “Docker Compose Build Project” to accurately reflect the project’s scope. The deployment used a single custom Nginx container managed by Docker Compose rather than multiple interconnected containers.

### Issues Encountered

Issue 1: Docker Compose Plugin Not Available

Problem

Attempting to install Docker Compose using the expected package manager command failed.

sudo dnf install docker-compose-plugin -y

Error:

No match for argument: docker-compose-plugin

Cause

Amazon Linux 2023 did not include the Docker Compose plugin package in the available repositories. The standard installation method used in many Linux distributions was not supported on this EC2 instance.

Fix

Docker Compose was installed manually by downloading the binary from the official Docker GitHub release page and placing it in Docker’s CLI plugins directory.

Verification:

docker compose version


Issue 2: Docker Permission Denied Error

Problem

Docker commands failed even though Docker was installed successfully.

Error:

permission denied while trying to connect to the Docker daemon socket


Cause

The EC2 user account did not belong to the Docker group. As a result, Docker commands could not communicate with the Docker daemon without elevated privileges.

Fix

Added the EC2 user to the Docker group:

sudo usermod -aG docker ec2-user


Logged out and reconnected to the EC2 instance to refresh group membership.

Verification:

docker version


Note: This issue has appeared in previous weeks and serves as a reminder that Docker group configuration is a required setup step on every new EC2 instance. It is not a one-time fix — it must be repeated each time a new instance is provisioned.

Issue 3: Docker Compose Build Failed

Problem

Running Docker Compose produced the following error:

Compose build requires buildx 0.17.0 or later


Cause

The EC2 instance contained Docker Buildx version 0.12.1, which was below the minimum version required by the installed version of Docker Compose.

Verification:

docker buildx version


Output:

github.com/docker/buildx 0.12.1


Fix

Downloaded and installed Buildx version 0.17.1 manually from the official Docker Buildx release repository.

Verification:

docker buildx version


Output:

github.com/docker/buildx v0.17.1


Issue 4: Incorrect Buildx Binary Download

Problem

The first Buildx installation attempt failed and produced:

Not: command not found


Cause

An invalid file was downloaded instead of the actual Buildx binary.

Rather than downloading executable software, the request returned a text-based error response from the server. When Linux attempted to execute the file, it interpreted the first word of the text response — “Not” — as a command, resulting in the error message: Not: command not found

This indicated that the downloaded file was not a valid executable binary.

Fix

Removed the incorrect file, verified the EC2 architecture using:

uname -m


Confirmed the system was running:

x86_64


Downloaded the correct Buildx binary for the x86_64 architecture and reinstalled it successfully.

Issue 5: Git Repository Pointed to the Wrong Project

Problem

Git operations were being performed against the Week 14 repository instead of the Week 15 repository.

Verification:

git remote -v


showed the wrong repository URL.

Cause

The local repository inherited the remote configuration from a previous project and remained connected to the Week 14 repository.

Fix

Updated the Git remote:

git remote set-url origin https://github.com/olusola-systems/aws-docker-compose-build-project.git


Verified the new configuration:

git remote -v


Successfully pushed the Week 15 project to the correct repository.

Issue 6: Week 14 Content Appeared in Week 15 Repository

Problem

The Week 15 repository displayed Week 14 website content.

Cause

The existing HTML file from the previous project was reused and not updated before committing and pushing to GitHub.

Fix

Updated the website content to reflect the Week 15 project objectives, committed the changes, and pushed the corrected version to GitHub.

Verification was performed by reviewing the repository contents and deployed webpage.

### Lessons Learned

- Docker Compose Is More Than a Convenience Tool

Before this project, I viewed Docker Compose as a shorter way to run Docker commands. Building this project changed that perspective.

Docker Compose allows infrastructure to be defined declaratively. Instead of remembering a sequence of commands to build images, create containers, expose ports, and manage restarts, the entire application configuration is stored in a version-controlled file.

The deployment process becomes repeatable and predictable because the desired state is documented in code.

- Container Tooling Dependencies Matter

One of the most valuable lessons came from troubleshooting Docker Compose failures.

Docker was installed correctly and functioning normally, yet deployments still failed because Buildx was outdated.

Initially, I assumed that if Docker was working, Docker Compose would work automatically. That assumption was wrong.

Container workflows depend on multiple components working together — Docker Engine, Docker Compose, Buildx, and CLI Plugins. A deployment can fail even when the container runtime itself is healthy.

This reinforced an important engineering principle: successful systems depend on understanding dependencies, not just the primary tool.

- Operating System Packages Are Not Always Current

Amazon Linux 2023 did not provide the Docker Compose plugin through the expected package repositories.

Cloud engineers cannot always rely on package managers to provide the latest tooling. In many production environments, software must be installed manually, upgraded independently, or sourced directly from official releases.

Understanding alternative installation methods becomes an operational skill rather than a convenience.

- Successful Troubleshooting Is Systematic

The project failed multiple times for different reasons — missing Docker Compose plugin, outdated Buildx version, incorrect Buildx binary download, repository configuration issues.

The solution was never guessing. Each issue required reading the exact error message, identifying the failing component, verifying assumptions, applying a targeted fix, and retesting.

Troubleshooting is a structured process, not trial and error.

- Infrastructure Knowledge Extends Beyond AWS

Although the project ran on AWS EC2, the challenges were not AWS-specific. The majority of the work involved Linux administration, Docker internals, software installation, dependency management, version compatibility, and Git workflows.

Cloud engineering is not only about cloud services. A significant portion of the role involves understanding the operating systems, tooling, and automation layers that run on top of cloud infrastructure.

### The Bigger Takeaway

The biggest lesson from this project was understanding that modern deployments are ecosystems of interconnected tools.

Docker alone was not enough. Docker Compose depended on Buildx. Buildx depended on the correct binary version. The deployment process depended on Git. The application depended on Linux permissions and configuration.

Each layer introduced its own failure points. Learning how those layers interact is what transformed this project from a simple container deployment into a practical cloud engineering exercise.

### Screenshots
<img width="1440" height="900" alt="Screenshot 2026-06-13 at 4 11 49 AM" src="https://github.com/user-attachments/assets/a2aa403a-8fa9-4da6-9ecc-9d8a197a8acf" />

<img width="1440" height="900" alt="Screenshot 2026-06-13 at 4 13 08 AM" src="https://github.com/user-attachments/assets/e5a387b7-1b04-49ae-b502-2e388dc3ebc6" />

### Next Project

Container Registries with Docker Hub and Amazon ECR

Next project introduces container image registries and image lifecycle management.

Instead of building Docker images directly on an EC2 instance, images will be built once, stored in a registry, and pulled onto deployment servers when needed.

## Future Improvements

- Deploy container images from Amazon ECR into Amazon ECS.
- Implement task definitions and ECS services.
- Integrate Application Load Balancers with ECS workloads.
- Configure CloudWatch logging and monitoring.
- Automate image builds and ECR deployments through GitHub Actions.
- Explore ECS Fargate for serverless container hosting.

This represents an important shift from building containers on servers to distributing standardized images across environments.
