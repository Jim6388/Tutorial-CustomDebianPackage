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

- Step 1: Download the source package:
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