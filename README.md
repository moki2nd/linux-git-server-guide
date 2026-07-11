# Building a Private Git Server from Scratch

This guide walks through the process of setting up a standalone Git Server on a Linux machine. While cloud platforms like GitHub are fantastic, setting up your own local server offers some unique benefits:

- **Complete Control and Privacy**: Your code lives entirely on your own hardware. You don't have to share your personal or experimental projects with any third-party service.
- **Work Offline**: You can commit, push, and collaborate with other computers on the same local network (LAN) even if your internet connection goes down.
- **No Limits**: You get unlimited space for your private repositories without having to worry about pricing plans or file size limits.

This project is a simple, step-by-step educational guide to help you build your own independent version control infrastructure.

*Note: For the Italian version of this guide, please see [README.it.md](README.it.md).*

---

## Prerequisites
Before starting, make sure you have:
- A Linux machine (physical or virtual) where you have access to a terminal.
- Basic knowledge of the command line (navigating directories, running commands).
- `sudo` privileges on the machine (i.e., you can run commands as administrator).

---

## 1. Installing Git
First, ensure Git is installed on your system. The installation command depends on your Linux distribution:

**Debian/Ubuntu:**
```bash
sudo apt update
sudo apt install git
```

**CentOS/RHEL/Fedora:**
```bash
sudo dnf install git  # Use 'yum' on older systems
```

**Arch Linux:**
```bash
sudo pacman -S git
```

## 2. Configuring the Server User (Server Side)
To isolate projects and ensure system security, we create a dedicated system user exclusively for Git operations. This user will not have root (`sudo`) privileges.
```bash
# Create the user and set a password
sudo useradd git-user
sudo passwd git-user

# Create the home directory for this user and assign ownership
sudo mkdir -p /home/git-user
sudo chown -R git-user:git-user /home/git-user
```

## 3. SSH Key Generation and Authentication (Client Side)
Using SSH keys allows for a secure connection to the server without needing to type a plaintext password for every Git operation.

On the developer's local machine:
```bash
# Generate the cryptographic key pair (press ENTER to accept default settings)
ssh-keygen -t rsa -C "developer@example.com"

# Send the public key to the server
cat ~/.ssh/id_rsa.pub | ssh git-user@localhost 'mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys'
```
*(During this operation, you will be asked for the `git-user` password one last time to authorize the transfer).*

## 4. Creating a Bare Repository (Server Side)
A repository on the server must be "bare". It serves as a central database and does not contain a working tree (the physical files you edit).
```bash
# Switch to the Git server administrator user
su git-user

# Create the folder and initialize the bare repository
mkdir -p /home/git-user/my-project.git
cd /home/git-user/my-project.git
git init --bare
```

## 5. Setting up the Local Repository (Client Side)
Now, let's switch back to the developer's perspective to create the actual local project.
```bash
# Exit the server user to return to your standard user
exit

# Create the working directory on your PC
mkdir -p /home/youruser/projects/my-project
cd /home/youruser/projects/my-project

# Initialize Git and configure the author's signature
git init
git config --global user.name "Your Name"
git config --global user.email "developer@example.com"

# Create an initial file to test the save process
echo "# Test Project" > readme.md
```

## 6. Committing and Pushing to the Private Server
Let's prepare our code and send it to our newly configured private server.
```bash
# Add files to the staging area
git add .
git commit -m "Initial project commit"

# Configure the "remote" to point to our private server
git remote add origin git-user@localhost:/home/git-user/my-project.git

# Push the code to the server
# Note: depending on your Git version, the default branch may be called 'main' instead of 'master'.
# If the command below fails, try: git push origin main
git push origin master
```

## 7. Verifying the Architecture
To confirm that the server is successfully acting as a central hub, let's simulate the arrival of a new developer by cloning the project from scratch into a different folder:
```bash
cd /home/youruser/projects
git clone git-user@localhost:/home/git-user/my-project.git
```
If the `clone` operation succeeds and the downloaded folder contains the `readme.md` file, your Git Server infrastructure is correctly configured and fully operational!

---

## 8. Additional Resources
To deepen your understanding of the concepts covered in this guide, here are some useful official resources:
- [Pro Git Book - Official Git Documentation](https://git-scm.com/book/en/v2)
- [Git on the Server - Setting Up the Server](https://git-scm.com/book/en/v2/Git-on-the-Server-Setting-Up-the-Server)
- [SSH Documentation (OpenSSH)](https://www.openssh.com/manual.html)
- [Linux File Permissions Explained](https://linuxcommand.org/lc3_lts0090.php)
