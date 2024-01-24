# Tutorial-CustomDebianPackage

Build Your Own Custom Debian Packages!

Ever wanted to make your own version of an existing Debian package and conveniently share it with others? If so, then this tutorial is for you! We'll guide you through the step-by-step process of building and hosting your own customized packages, allowing you to:

- Download the source code of any package from the Ubuntu archive.
- Modify the code to add your own personal patch.
- Modify the Debian package specification files and build the package
- Create a Personal Package Archive (PPA) to store your customized package.
- Upload your customized package to your PPA for easy download and installation via `apt`.

By the end of this tutorial, you'll have a solid foundation of knowledge for crafting and sharing your own Debian packages. This opens up a world of possibilities for customizing your system and contributing to the Debian ecosystem.

## Before We Get Started

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

### Prerequisite

#### Gearing Up Your Local Device

- Essential Tools:
  - Check for these packages:
    - `build-essential`
    - `devscripts`
    - `openssh-client`
  - Install any missing packages: Use `apt` to install them.
- Secure Your Connection:
  - Create an SSH key pair: This is vital for publishing your code on to PPA.
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
  - Guide: Follow these [key creating and registering guide](https://help.launchpad.net/YourAccount/CreatingAnSSHKeyPair).
- Add an OpenPGP Public Key:
  - Purpose: Verifies your identity for tasks like:
  - Guide: Follow these [Launchpad OpenPGP key import guide](https://launchpad.net/+help-registry/import-pgp-key.html).

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
    - `hello_2.10-2ubuntu4.debian.tar.xz`: Contains modifications made by the Debian maintainer, including instructions for constructing Debian binary packages.
    - `hello_2.10.orig.tar.gz`: The original, unmodified source code in gzip-compressed tar format.
