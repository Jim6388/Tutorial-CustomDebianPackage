# Tutorial - Custom Debian Package

Build Your Own Custom Debian Packages!

Ever wanted to make your own version of an existing Debian package and conveniently share it with others? If so, then this tutorial is for you! I'll guide you through the step-by-step process of building and hosting your own customized packages, allowing you to:

- Download the source code of any package from the Ubuntu archive.
- Modify the code to add your own personal patch.
- Modify the Debian package specification files and build the package
- Create a Personal Package Archive (PPA) to store your customized package.
- Upload your customized package to your PPA for easy download and installation via `apt`.

By the end of this tutorial, you'll have a solid foundation of knowledge for crafting and sharing your own Debian packages. This opens up a world of possibilities for customizing your system and contributing to the Debian ecosystem.

## Before we get started

## Hierarchy

``` bash
.
├── README.md       # Tutorial: Building a Customized Hello Package with Debian Packaging
└── source_package  # Source code and packaging materials for the customized hello package. See the commit history for detailed build steps: 
```

### Key terms

- Debian package:
  - A Debian package contains the metadata and conditions for installing software on a Debian GNU/Linux distribution or a Debian derivative such as Ubuntu.
  - There are two kinds of Debian package: binary (`.deb`) and source package.
  - Dive deeper into this [Manual](https://www.debian.org/doc/manuals/debian-faq/pkg-basics.en.html) for more details.
- Launchpad: A platforms like GitHub, GitLab or BitBucket.
- Personal Package Archives (PPAs): As the name suggests, ppa represents an environment, platform, or infrastructure for individual developers to publish software packages.
- `sources.list`: It dictates the repositories `apt` can utilize for package downloads and installations.
- Ubuntu archive: The Ubuntu archive, typically referred to as `archive.ubuntu.com`, is a comprehensive repository of software packages for Ubuntu and its derivatives. It serves as the central hub where users can find and download official packages for installing and updating software on their systems.
- upstream source: Refers to the original source or repository of a software program or package.

### Scenario

Let's get an overall picture of what I'm going to do in this tutorial:

- Step 1: Download the source package from Ubuntu Archive:
  - Obtain the source of the existing `hello` package from the Ubuntu archive.
- Step 2: Modify the upstream source:
  - Add a test script: Create a script named `testing.sh` that echoes "this is a test from ChunChia Tsao" to standard error (STDERR).
- Step 3: Modify the Debian package specification files and rebuild:
  - Debian package specification files modifications: 
    - Modify the specification file to unpack `testing.sh` to `/usr/bin` during installation.
    - Modify the specification file to automatically execute `testing.sh` upon installation.
  - Rebuild the package:
    - Create your customized `hello` source package.
- Step 4: Create a Personal Package Archive (PPA):
  - Establish a PPA to store your customized package.
- Step 5: Upload your customized package:
  - Upload your customized hello package to your PPA for easy download and installation via apt.
- Step 6: Install your customized package from the PPA

### Prerequisite

#### Gearing Up Your Local Device

- Essential Tools:
  - Check for these packages:
    - `build-essential`
    - `devscripts`
    - `openssh-client`
  - Install any missing packages: Use `apt` to install them.
- Secure Your Connection:
  - Create an SSH key pair: This is vital for publishing your code to PPA.
  - Follow these clear [steps](https://help.launchpad.net/YourAccount/CreatingAnSSHKeyPair)
- Verify Your Identity:
  - Create an OpenPGP key pair: This is crucial for:
    - Bug tracker interaction
    - Code of conduct signing
    - Package signing
  - Generation methods:
    - [GUI tool](https://launchpad.net/+help-registry/openpgp-keys.html)
    - Command line: `gpg --gen-key`
- Tweak Environment Variables:
  - Debian package building optimization: Append these lines to your `~/.bashrc` file:

```bash
DEBEMAIL="your email address"
DEBFULLNAME="your fullname"
export DEBEMAIL DEBFULLNAME
DEBSIGN_KEYID="Your_GPG_keyID"
export DEBSIGN_KEYID
```

#### Setup Your PPA on Launchpad.net

- Add an SSH Public Key:
  - Purpose: Enables secure communication with Launchpad.
  - Guide: Follow this [key creating and registering guide](https://help.launchpad.net/YourAccount/CreatingAnSSHKeyPair).
- Add an OpenPGP Public Key:
  - Purpose: Verifies your identity for tasks like:
  - Guide: Follow this [Launchpad OpenPGP key import guide](https://launchpad.net/+help-registry/import-pgp-key.html).

## Let's start our journey

### Step 0: Set Up Your Workspace

#### Establishing a Clean Slate

It's a best practice to conduct experiments within a dedicated, empty directory. This ensures organization and avoids unintended file modifications. Let's create a working directory named `package_test` under your home directory for this purpose.

#### Creating the Directory

Open a terminal window. Execute the following command to navigate to your home directory and create the package_test directory:

```bash
cd ~
mkdir package_test
```

### Step 1: Download the source package from Ubuntu Archive

#### Acquiring Source Packages with Apt

While multiple methods exist for obtaining source packages, we'll utilize the `apt` package manager for this tutorial. However, before downloading packages, we need to configure the sources list file to ensure `apt` can access the necessary repositories.

#### Modifying the Sources List

- Understanding `sources.list`:
  - This file, typically found at `/etc/apt/sources.list`, dictates the repositories `apt` can utilize for package downloads and installations.
  - Each line within represents a repository, using the format:
    `package_type [option1=value1 option2=value2 ...] uri suite components`
  - Breakdown:
    - `package_type`: Indicates whether it's a binary (`deb`) or source (`deb-src`) package repository.
    - `option`: Optional fields for attributes like architecture (e.g., `arch=amd64`).
    - `uri`: The base URL of the repository (usually a network address or local path).
    - `suite`: The codename of the Ubuntu release (e.g., `jammy`, `focal`, `bionic`).
    - `components`: A list of package categories within the repository (e.g., `main`, `universe`, `restricted`, `multiverse`).
- Activating Source Repositories:
  - To enable source repositories for `apt`, you can manually uncomment the relevant lines in `/etc/apt/sources.list`. For the hello package, uncomment the line: `# deb-src http://archive.ubuntu.com/ubuntu/ jammy main restricted`
  - Alternatively, use add-apt-repository:
  `sudo add-apt-repository -s 'deb-src http://tw.archive.ubuntu.com/ubuntu/ jammy main'`
  - After modifying sources.list, update package lists: `sudo apt update`
- Retrieving the Source Package:
  - Downloading with apt source:
    - Use `apt source PACKAGE-NAME` to download the source package. For hello: `sudo apt source hello`
  - The download generates several files:
    - `hello-2.10`: Directory containing upstream source code and Debian package specification files.
    - `hello_2.10-2ubuntu4.dsc`: File describing the source package.
    - `hello_2.10-2ubuntu4.debian.tar.xz`: Contains modifications made by the Debian maintainer, including instructions for constructing Debian packages.
    - `hello_2.10.orig.tar.gz`: The original, unmodified source code in gzip-compressed tar format.

### Step 2: Modify the upstream source

#### Adding a Custom Patch

Navigate to the `hello-2.10` directory (the home of the upstream source) and create a `testing.sh` script that will echo a greeting message to standard error (STDERR):

```bash
cd hello-2.10
echo '#!/bin/bash
echo "this is a test from ChunChia Tsao" >&2' > testing.sh
```

#### Confirming Functionality

- Granting Execution Permissions:
  - Ensure the script is executable: `sudo chmod +x testing.sh`
- Executing the Script:
  - Verify its behavior by running it: `./testing.sh`
  - You should observe the following output: `this is a test from ChunChia Tsao`

### Step 3: Modify the Debian package specification files and rebuild

Now, let's customize the Debian package specification files to integrate our `testing.sh` script and then rebuild the package for installation.

#### Adding Installation Files

- The `install` file controls what gets installed beyond the upstream source.
- It can be named either `install` or `binary-package-name.install`, and resides in the `debian` subdirectory.
- Each line specifies a file to copy, with the format: `source/path destination/path.`

In our case, we want to copy `testing.sh` to `/usr/bin` during installation. Create an install file in the debian directory: 

```bash
# Before executing this command, navigate to the debian subdirectory within the upstream source
echo "testing.sh /usr/bin" > install
```

Verify the content with cat: `cat install`

#### Automating Script Execution

- The `postinst` script runs automatically after package installation.
- To execute `testing.sh` upon installation, let's add it to the post installation script

Create a `postinst` script in the debian directory:

```bash
# Before executing this command, navigate to the debian subdirectory within the upstream source
echo '#!/bin/sh
/usr/bin/testing.sh' > postinst
```

Verify the content with cat: `cat postinst`

#### Creating Modification Patch

- Changes to the upstream source need to be stored as patches in the `debian/patches/` directory.
- Run `dpkg-source --commit` to create a patch and write a proper commit message. 
- Follow the guidelines in this [documentation](https://www.debian.org/doc/manuals/debmake-doc/ch05.en.html#native) to write a clear commit message.

#### Updating Changelog

The `changelog` file records package history and versions. We'll use the `debchange` command (alias `dch`) to edit it. For local builds, use `sudo -E dch --local ""`. Write a clear and concise changelog entry and save it.

Tips:

- `sudo -E:` Grants elevated privileges for the build process and preserves environment variables.
- You can also edit `debian/changelog` manually, following the format used by `debchange`.

#### Building the Package

Finally, let's rebuild the package with `debuild`:

```bash
# Ensure you're in the upstream source directory before executing this command
sudo -E debuild -S -k$DEBSIGN_KEYID
```

Upon successful completion, you'll find the new source package residing within the parent directory of upstream source directory.

Tips:

- `sudo -E:` Grants elevated privileges for the build process and preserves environment variables.
- `-S`: Instructs `debuild` to create a source package only, suitable for distribution.
- `-k$DEBSIGN_KEYID`: signs the package with your GPG key. Without correct signature, you can't upload the source package to PPA.

### Step 4: Create a Personal Package Archive (PPA)

Head over to [launchpad](https://launchpad.net/) in your web browser and log in to your account. Within your account dashboard, locate the section named "Personal package archives" and click the "Create a new PPA" button. This button will redirect you to a new page. Follow the instructions on the new page to activate your new Personal Package Archive.

### Step 5: Upload your customized package to PPA

#### Uploading the Package

Use the `dput` command to upload your custom source package to the PPA:

```bash
dput ppa:<username>/<ppa-name> <package_change_file>
```

Replace placeholders with our PPA details and change file. In our example, it will be:

```bash
dput ppa:jim6388/testppa hello_2.10-2ubuntu5_source.changes
```

Tips:

Please remember to replace placeholders correctly with your PPA details and change file.

#### Waiting for Review and Build

- Review Period: Upon successful upload, a review process ensures package quality and security. This might take some time.
- Notification: You'll receive an email regarding package acceptance or rejection to the address associated with your Launchpad account.
- Building Binary Package: If accepted, the PPA platform will automatically build the binary package, which also requires time.

#### Checking Build Status

Before attempting installation, verify the package's build status on Launchpad to ensure it's ready.

### Step 6: Install your customized package from the PPA

#### Add the PPA to Your Sources List

- Visit the PPA's webpage in your web browser.
- Locate the section titled "Adding this PPA to your system."
- Copy and paste the provided commands into your terminal.

In our example it will be:

```bash
sudo add-apt-repository ppa:jim6388/testppa
sudo apt update
```

#### Install the Package

Use apt install to download and install the package:

```bash
sudo apt install <package_name>
```

In our example, the command and result are:

```bash
$ sudo apt install hello
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  hello
0 upgraded, 1 newly installed, 0 to remove and 8 not upgraded.
Need to get 53.6 kB of archives.
After this operation, 281 kB of additional disk space will be used.
Get:1 https://ppa.launchpadcontent.net/jim6388/testppa/ubuntu jammy/main amd64 hello amd64 2.10-2ubuntu5 [53.6 kB]
Fetched 53.6 kB in 2s (35.7 kB/s)
Selecting previously unselected package hello.
(Reading database ... 41419 files and directories currently installed.)
Preparing to unpack .../hello_2.10-2ubuntu5_amd64.deb ...
Unpacking hello (2.10-2ubuntu5) ...
Setting up hello (2.10-2ubuntu5) ...
this is a test from ChunChia Tsao
Processing triggers for man-db (2.10.2-1) ...
Processing triggers for install-info (6.8-4build1) ...
```

## Reference

- [Guide for Debian Maintainers](https://www.debian.org/doc/manuals/debmake-doc/index.en.html): This guide focuses on the modern packaging style and comes with many simple examples.
- [Ubuntu Packaging Guide](https://canonical-ubuntu-packaging-guide.readthedocs-hosted.com/en/latest/): This is the official place for learning all about Ubuntu Development and packaging.
- [Repositories](https://help.ubuntu.com/community/Repositories/CommandLine): This wiki page show detail steps about how to add .repositories into source list.
- [GPG Keys Management](https://docs.fedoraproject.org/en-US/quick-docs/create-gpg-keys/): This documents tells how to use `gpg` command.
- [GnuPrivacyGuardHowto](https://help.ubuntu.com/community/GnuPrivacyGuardHowto): This wiki page gives a quick guide of how to use OpenPGP keys.
