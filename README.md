The tutorial assumes some basic familiarity with Kubernetes and kubectl but does not assume any pre-existing deployment.

It also assumes that you are familiar with the usual Terraform plan/apply workflow. If you're new to Terraform itself, refer first to the Getting Started tutorial.

For this tutorial, you will need:

an AWS account with the IAM permissions listed on the EKS module documentation,
a configured AWS CLI
AWS IAM Authenticator
kubectl
wget (required for the eks module)

Install and configure Jenkins:
Add the Jenkins repo using the following command:

[ec2-user ~]$ sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
Import a key file from Jenkins-CI to enable installation from the package:

[ec2-user ~]$ sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
[ec2-user ~]$ sudo yum upgrade
Install Jenkins:

[ec2-user ~]$ sudo yum install jenkins java-1.8.0-openjdk-devel -y
[ec2-user ~]$ sudo systemctl daemon-reload
Start Jenkins as a service:

[ec2-user ~]$ sudo systemctl start jenkins
You can check the status of the Jenkins service using the command:

[ec2-user ~]$ sudo systemctl status jenkins


=====================================================================================================
#Installing or updating the latest version of the AWS CLI

#The AWS CLI uses glibc, groff, and less. These are included by default in most major distributions of Linux.

#We support the AWS CLI on 64-bit versions of recent distributions of CentOS, Fedora, Ubuntu, Amazon Linux 1, Amazon Linux 2 and Linux ARM.

#Because AWS doesn't maintain third-party repositories, we can’t guarantee that they contain the latest version of the AWS CLI.

#Installation instructions
#Follow these steps from the command line to install the AWS CLI on Linux.

#We provide the steps in one easy to copy and paste group based on whether you use 64-bit Linux or Linux ARM. See the descriptions of each line in the steps that follow.

#Linux x86 (64-bit)
#Linux ARM
$ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
Download the installation file in one of the following ways:

Linux x86 (64-bit)
Linux ARM
Use the curl command – The -o option specifies the file name that the downloaded package is written to. The options on the following example command write the downloaded file to the current directory with the local name awscliv2.zip.

$ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
Downloading from the URL – To download the installer with your browser, use the following URL: https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip

(Optional) To verify the integrity and authenticity of your downloaded installation file before you unpack the package, follow the instructions in (Optional) Verifying the integrity of your downloaded zip file.

Unzip the installer. If your Linux distribution doesn't have a built-in unzip command, use an equivalent to unzip it. The following example command unzips the package and creates a directory named aws under the current directory.

$ unzip awscliv2.zip
Run the install program. The installation command uses a file named install in the newly unzipped aws directory. By default, the files are all installed to /usr/local/aws-cli, and a symbolic link is created in /usr/local/bin. The command includes sudo to grant write permissions to those directories.

$ sudo ./aws/install
You can install without sudo if you specify directories that you already have write permissions to. Use the following instructions for the install command to specify the installation location:

Ensure that the paths you provide to the -i and -b parameters contain no volume name or directory names that contain any space characters or other white space characters. If there is a space, the installation fails.

--install-dir or -i – This option specifies the directory to copy all of the files to.

The default value is /usr/local/aws-cli.

--bin-dir or -b – This option specifies that the main aws program in the install directory is symbolically linked to the file aws in the specified path. You must have write permissions to the specified directory. Creating a symlink to a directory that is already in your path eliminates the need to add the install directory to the user's $PATH variable.

The default value is /usr/local/bin.

$ ./aws/install -i /usr/local/aws-cli -b /usr/local/bin
Note
To update your current installation of the AWS CLI, add your existing symlink and installer information to construct the install command with the --update parameter.

$ sudo ./aws/install --bin-dir /usr/local/bin --install-dir /usr/local/aws-cli --update
To locate the existing symlink and installation directory, use the following steps:

Use the which command to find your symlink. This gives you the path to use with the --bin-dir parameter.

$ which aws
/usr/local/bin/aws
Use the ls command to find the directory that your symlink points to. This gives you the path to use with the --install-dir parameter.

$ ls -l /usr/local/bin/aws
lrwxrwxrwx 1 ec2-user ec2-user 49 Oct 22 09:49 /usr/local/bin/aws -> /usr/local/aws-cli/v2/current/bin/aws
Confirm the installation with the following command.

$ aws --version
aws-cli/2.3.2 Python/3.8.8 Linux/4.14.133-113.105.amzn2.x86_64 botocore/2.0.0
If the aws command cannot be found, you may need to restart your terminal or follow the instructions in Adding the AWS CLI to your path.

=======================================================================================================================================================
To install aws-iam-authenticator on Linux

Download the Amazon EKS vended aws-iam-authenticator binary from Amazon S3. To download the Arm version, change <amd64> to arm64 before running the command.

curl -o aws-iam-authenticator https://amazon-eks.s3.us-west-2.amazonaws.com/1.21.2/2021-07-05/bin/linux/amd64/aws-iam-authenticator
(Optional) Verify the downloaded binary with the SHA-256 sum provided in the same bucket prefix.

Download the SHA-256 sum for your system. To download the Arm version, change <amd64> to arm64 before running the command.

curl -o aws-iam-authenticator.sha256 https://amazon-eks.s3.us-west-2.amazonaws.com/1.21.2/2021-07-05/bin/linux/amd64/aws-iam-authenticator.sha256
Check the SHA-256 sum for your downloaded binary.

openssl sha1 -sha256 aws-iam-authenticator
Compare the generated SHA-256 sum in the command output against your downloaded aws-iam-authenticator.sha256 file. The two should match.

Apply execute permissions to the binary.

chmod +x ./aws-iam-authenticator
Copy the binary to a folder in your $PATH. We recommend creating a $HOME/bin/aws-iam-authenticator and ensuring that $HOME/bin comes first in your $PATH.

mkdir -p $HOME/bin && cp ./aws-iam-authenticator $HOME/bin/aws-iam-authenticator && export PATH=$PATH:$HOME/bin
Add $HOME/bin to your PATH environment variable.

echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
Test that the aws-iam-authenticator binary works.

aws-iam-authenticator help
=============================================================================================================================
Install kubectl binary with curl on Linux
Download the latest release with the command:

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
Note:
To download a specific version, replace the $(curl -L -s https://dl.k8s.io/release/stable.txt) portion of the command with the specific version.

For example, to download version v1.22.0 on Linux, type:

curl -LO https://dl.k8s.io/release/v1.22.0/bin/linux/amd64/kubectl
Validate the binary (optional)

Download the kubectl checksum file:

curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
Validate the kubectl binary against the checksum file:

echo "$(<kubectl.sha256) kubectl" | sha256sum --check
If valid, the output is:

kubectl: OK
If the check fails, sha256 exits with nonzero status and prints output similar to:

kubectl: FAILED
sha256sum: WARNING: 1 computed checksum did NOT match
Note: Download the same version of the binary and checksum.
Install kubectl

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
Note:
If you do not have root access on the target system, you can still install kubectl to the ~/.local/bin directory:

chmod +x kubectl
mkdir -p ~/.local/bin/kubectl
mv ./kubectl ~/.local/bin/kubectl
# and then add ~/.local/bin/kubectl to $PATH
Test to ensure the version you installed is up-to-date:

kubectl version --client
Install using native package management
Debian-based distributions
Red Hat-based distributions
Update the apt package index and install packages needed to use the Kubernetes apt repository:

sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
Download the Google Cloud public signing key:

sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
Add the Kubernetes apt repository:

echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
Update apt package index with the new repository and install kubectl:

sudo apt-get update
sudo apt-get install -y kubectl
=========================================================================================================
    
