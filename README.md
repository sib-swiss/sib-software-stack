# SIB software stack builder and documentation

The **SIB software stack** is a collaborative effort between different SIB units to deploy a common
software stack based on the [EasyBuild project](https://easybuild.io).

The current version of the SIB software stack is based on the `foss-2021a` toolchain that uses
`GCC 10.3.0`.

The full documentation for EasyBuild [can be found here](https://docs.easybuild.io/en/latest).


<br>
<br>


## Quick-start guide
The following steps should get your started, but you are encouraged to read the full documentation.

1. **Clone the the different git repositories** needed to build the SIB software stack. Note that
   cloning via SSH (as below) requires that you have an SSH key associated with your GitHub
   account (alternatively, you may clone via HTTPS):
    ```
    git clone git@github.com:sib-swiss/sib-software-stack sib-software-stack.git
    git clone git@github.com:sib-swiss/easybuild-easyconfigs.git sib-easyconfigs.git
    git clone git@github.com:sib-swiss/stack-builder.git stack-builder.git
    ```
   For convenience, you can add the executable `stack-builder.git/sb.py` to your `PATH`.

2. **Install *EasyBuild* and dependencies of the *stack-builder*:**
    ```
    pip3 install --user easybuild GitPython
    # For python 3.6 only: pip3 install --user dataclasses
    ```
   To verify that your EasyBuild installation is working, run: `EB_VERBOSE=1 eb --version`.

3. **Install configuration files:**
    ```
    [[ ! -d  .config/easybuild ]] && mkdir .config/easybuild
    cp ./sib-software-stack.git/config_easybuild.cfg.template ~/.config/easybuild/config.cfg
    cp ./stack-builder.git/config_stackbuilder.cfg.template ~/.config/easybuild/config_stackbuilder.cfg
    ```
   To configure the `config.cfg` and `config_stackbuilder.cfg` files, please refer to
   [the configuration files section](#configuration-files) below.

   To verify your config files, run:
    ```
    eb --show-config
    ```

5. **Run the *stack-builder* tool:**
    ```
    sb.py update       # Updates the content of the sib-easyconfigs and sib-software-stack repos.
    sb.py build -s     # Summary of the software packages to build.
    sb.py build -d     # Performs an EasyBuild "--dry-run" of all packages to build.
    sb.py build        # Build all software not yet present in the local EasyBuild stack.
    ```


<br>
<br>


## Setting-up of the local build environment

### Installing/cloning of easyconfig, stack-builder and documentation repositories
The SIB software stack's tools, documentation and build recipes (easyconfig files) are distributed
across 3 Git repositories:

* **[`sib-software-stack`](https://github.com/sib-swiss/sib-software-stack):**
  this repository contains configuration file templates for *EasyBuild* and the *stack-builder*,
  as well as the "private" easyconfigs files of the SIB software stack. **Private easyconfigs** are
  the easyconfigs that are not intended to be contributed back to the EasyBuild project,
  typically because the scope of the software is narrow (i.e. only a very limited number of people
  are likely to use it).
  Specifically, this repo contains the following files/directories:
    * `config_easybuild.cfg.template`: EasyBuild configuration file template.
    * `config_stackbuilder.cfg.template`: stack-builder configuration file template.
    * `sib_stack_package_list.txt`: list of software packages (easyconfig file names) to be built.
    * `easyconfigs`: directory for storing private easyconfigs.

* **[`sib-easyconfigs`](https://github.com/sib-swiss/easybuild-easyconfigs):**
  fork of the original [EasyBuild project easyconfig repo](https://github.com/easybuilders/easybuild-easyconfigs).
  This repo is dedicated to hosting easyconfig files that are planned to be contributed back to the
  EasyBuild project. Each institution participating in the SIB software stack can have their own
  branch in this repo, where new easyconfig files can be added before a pull-request is made to the
  EasyBuild project.

* **[`stack-builder`](https://github.com/sib-swiss/stack-builder):**
  tool to automate the update and build of the SIB software stack. Please see the repo's
  [README file](https://github.com/sib-swiss/stack-builder/blob/master/README.md) for more details
  about its usage.

To clone these 3 Git repos, use the following commands (cloning either via HTTPS or SSH):
```
# Cloning via SSH:
git clone git@github.com:sib-swiss/sib-software-stack sib-software-stack.git
git clone git@github.com:sib-swiss/easybuild-easyconfigs.git sib-easyconfigs.git
git clone git@github.com:sib-swiss/stack-builder.git stack-builder.git

# Cloning via HTTPS:
git clone https://github.com/sib-swiss/sib-software-stack sib-software-stack.git
git clone https://github.com/sib-swiss/easybuild-easyconfigs.git sib-easyconfigs.git
git clone https://github.com/sib-swiss/stack-builder.git stack-builder.git
```

<br>

### Installing EasyBuild
*EasyBuild* is distributed as a python module and can be installed and upgraded with `pip`.
```
pip3 install --user easybuild              # Install EasyBuild
pip3 install --upgrade --user easybuild    # Upgrade EasyBuild
```
More details on EasyBuild installation
[are given here](https://docs.easybuild.io/en/latest/Installation.html#installing-easybuild),
in particular, make sure that you also have a `lmod` installed on your system.

*Note:* upon installation, EasyBuild provides a number of easyconfig files (files that contain the
build recipe for a given software) stored in `~/.local/easybuild/easyconfigs`. These are in
principle not used as part of the SIB software stack since we use files from the more up-to-date
[sib-easyconfigs](https://github.com/sib-swiss/easybuild-easyconfigs.git) repo instead.

The following environment variables might also need to be added to your `.bashrc`.
```
# EasyBuild configuration
export EB_PYTHON=python3                                                     # Make sure that EasyBuild uses python3.
export EASYBUILD_CONFIGFILES=~/sib_software_stack.git/config_easybuild.cfg   # Path to the EasyBuild config file.
```
*Note:* an alternative to setting `EASYBUILD_CONFIGFILES` is to store the config file at the location
where EasyBuild searches for it by default: `~/.config/easybuild/config.cfg`.

To test the installation of EasyBuild, run: `EB_VERBOSE=1 eb --version`.
To show display your EasyBuild configuration values, run `eb --show-config` or
`eb --show-full-config`.

More details on how to configure EasyBuild are given in
[this section below](#configuring-the-EasyBuild-config).

<br>

### Installing the stack-builder tool
The **stack-builder** tool allows to automate the deployment and updating of the SIB software
stack.

To install the stack-builder tool, clone its
[git repo](https://github.com/sib-swiss/stack-builder) - this has in principle already been done -
and add the **`sb.py`** executable to your `PATH`.

The stack-builder has currently 2 main commands:
* **`build`:** builds/updates the local instance of the software stack by building the
  software packages listed in the `sib_stack_package_list.txt` file (as well as all of their
  dependencies).
* **`update`:** update the local instances of the `sib-software-stack.git` and `sib-easyconfigs.git`
  repositories.

All available options and their shortcuts can be displayed with `sb.py --help`. For more
information on how to install and use the stack-builder tool see
https://github.com/sib-swiss/stack-builder.

<br>

### Installing the EasyBuild and stack-builder configuration files
Two possibilities exists for installing the *EasyBuild* and *stack-builder* configuration files:

1. **Option 1:** install the configuration files in **` ~/.config/easybuild/`**, the default
   EasyBuild location for config files. Templates for both the EasyBuild and the stack-builder
   configuration files are available from the `sib-software-stack` and `stack-builder` Git
   repositories:
   ```
   cp ./sib-software-stack.git/config_easybuild.cfg.template ~/.config/easybuild/config.cfg
   cp ./stack-builder.git/config_stackbuilder.cfg.template ~/.config/easybuild/config_stackbuilder.cfg
   ```
   **Important:** the config file for EasyBuild must be named `config.cfg` and the config file
   for the stack-builder must be named `config_stackbuilder.cfg`.

2. **Option 2:** install the configuration files at any custom location (e.g. in
  `sib-software-stack.git` where the `.gitignore` file is already setup for Git to ignore them),
   and to set an `EASYBUILD_CONFIGFILES` environment variable pointing to the EasyBuild config
   file.
   ```
   export EASYBUILD_CONFIGFILES=/path/to/sib-software-stack.git/config_easybuild.cfg
   ```
   **Important:** if the `config_stackbuilder.cfg` file is located in the same directory as the
   `config_easybuild.cfg`, it will be automatically detected. If the `config_stackbuilder.cfg` file
   is in a different location, the `STACKBUILDER_CONFIGFILES` environment variable must be added to
   the shell environment to indicate its location.

<br>

### Configuration files

#### Configuring the EasyBuild config
A large number of configuration options are available for EasyBuild
[see here](https://docs.easybuild.io/en/latest/Configuration.html), but you should set at least
the following values in the `config.cfg` file:
* **`buildpath`:** directory where EasyBuild compiles/builds software. This is not the final
  installation location and the content of this directory will be regularly cleaned.
* **`sourcepath`:** directory where EasyBuild stores the source files used to build software. This
  location should be backed-up so that you are always able to access the source files in case a
  given software needs to be rebuilt at a later point and the original source files have become
  unavailable.
* **`robot-paths`:** column-delimited of `easyconfigs` directories where EasyBuild should search
  for easyconfig files. In principle this list should contain the paths to
  `sib-software-stack.git/easyconfigs` and to `sib-easyconfigs.git/easybuild/easyconfigs` - in
  this order.
* **`installpath`:** location where the final, compiled, software should be installed.
* **`github-user`:** the GitHub user name to use for making contributions (i.e. pull-request) to
  the EasyBuild project.

To test your EasyBuild configuration, run: `eb --show-config`.

#### Configuring the stack-builder config
In the `config_stackbuilder.cfg` file, adapt the following values to your environment:
* **`sib_easyconfigs_repo`:** path to the `sib-easyconfigs.git` directory.
* **`sib_software_stack_repo`:** path to the `sib-software-stack.git` directory.
* **`sib_node`:** the name assigned to your institution, i.e. one of `ibu` `scicore`, `ubelix`, or
  `vitalit`.
* **`allow_reset_node_branch`:** how to handle resetting of your personal work branch (i.e. the
    branch specified under `sib_node`) to its upstream branch (`origin/<branch name>`).
    Possible values for this arguments are:
     * `interactive`: each time a branch reset has to be performed, the user will be prompted for
       an interactive answer.
     * `yes`: always accept branch rebase operations. Please note that this can potentially result
       in commit losses on your local branch if you are not careful to always push new commits on
       your work branch to the remote.
     * `no`: always reject branch rebase operations.
* **`allow_reset_other_nodes_branch`:** same as `allow_reset_node_branch`, but for the work
  branches of nodes other than your own. In principle each node/institution will work on their own
  branch only, and therefore this argument can be set to `yes` fairly safely.

Additional optional argument values can also be set in this configuration, please see the
[stack-builder documentation](https://github.com/sib-swiss/stack-builder/blob/master/README.md)
for details.

To test your stack-builder configuration, run: `sb.py build -s`.

<br>

### System libraries
When building some of the software, **installing a few system libraries is needed** (because some
of the base system libraries are not ported to EasyBuild), or recommended (because the library is
difficult to build with EasyBuild). However, these cases are relatively rare.

As an example, when building `R 4.0.3` on a CentOS 7 machine, the following system dependencies
have to be installed: `sudo yum install openssl-devel libibverbs.x86_64 rdma-core-devel`.

Here is a list of system libraries that are recommended to be installed:
* OpenSSL (with devel files).
* Cairo (graphics library).


<br>
<br>


## The package list file
`sib_stack_package_list.txt` is a **plain text file that contains the list of software packages**
**to be built**, in the order they should be built (top to bottom). Software packages are
identified by their easyconfig file name.

To add a new software package to the SIB software stack, the name of its easyconfig file must be
added to this file.

When the file is parsed by the stack-builder tool, the following rules are applied:
* Text located after a `#` (comments) and empty lines are ignored.
* Software packages are built in the order they appear in the file, from top to bottom.
* Lines prefixed with one or more node name abbreviations (comma separated) indicate that the given
  package should only be built at the specified node/institution (the node value is specified via
  the `sib_node`, and optionally the `optional_software`, values of the stack-builder config
  file).
  Note/institution abbreviations are the following:
    * `HUG`: HUG Geneva hospital cluster.
    * `IBU`: IBU cluster.
    * `SCI`: sciCORE.
    * `UBE`: UBELIX cluster.
    * `VIT`: SIB lausanne.

  **Examples:**
    * The following line would indicates that the `Rgurobi-9.1.2` software should only be built on
      the Lausanne cluster infrastructure (abbreviated `VIT`).
      ```
      VIT Rgurobi-9.1.2-foss-2021a-R-4.1.0.eb
      ```
    * The following line would indicates that the `Rgurobi-9.1.2` software should only be built on
      the Lausanne cluster and the IBU cluster infrastructure.
      ```
      VIT,IBU Rgurobi-9.1.2-foss-2021a-R-4.1.0.eb
      ```

   **Important:** as one of the objectives of the SIB software stack is to standardize the software
   stacks across the different SIB-maintained clusters, it is suggested to keep the site-specific
   packages to a minimum.

When searching for an easyconfig file specified in `sib_stack_package_list.txt`, EasyBuild
searches the different paths listed in the `robot-paths` variable of its configuration file.
In the SIB software stack, this variable contains in principle the following two paths:
* **`.../sib-software-stack.git/easyconfigs`:** the *private easyconfigs* repo of the SIB software
  stack. This repository contains easyconfigs of software that are too specific to be of interest
  to the wider EasyBuild community.

* **`.../sib-easyconfigs.git/easybuild/easyconfigs`:** this is a fork of the "development" repo of
  EasyBuild, enriched with the easyconfig files that are contributed by individual nodes
  participating in the SIB software stack. This repo thus contains *public easyconfigs*, that are
  shared with the EasyBuild community (once they are contributed to the official EasyBuild
  repo). This repo also contain the latest "development" versions of easyconfig files as they get
  updated/added by the EasyBuild project.

If a given easyconfig is found in both of the above paths, the version of the file found in the
path listed first (in our case `.../sib-software-stack.git/easyconfigs`) is used and thus overrides
the version of the file found in the second path.
For this reason, the `.../sib-software-stack.git/easyconfigs` repository should come first in the
`robot-paths` values, so that, if needed, an easyconfig can be placed in the *private easyconfigs*
repo to override an existing easyconfigs from the *public easyconfigs* repo.


<br>
<br>


## Sharing of easyconfig files between nodes

### Public vs. private easyconfigs: the sib-easyconfigs and sib-software-stack repos
As mentioned earlier, the SIB software stack contains both *public* and *private* easyconfig files.
The sharing of these files is done via the two Git repos:

**The `sib-easyconfigs` repo:**
* Contains **public easyconfigs**, that will be contributed to the EasyBuild project at some
  point.
* Hosted at [sib-easyconfigs](https://github.com/sib-swiss/easybuild-easyconfigs), which is
  a fork of the original [EasyBuild easybuild-easyconfigs](https://github.com/easybuilders/easybuild-easyconfigs)
  repo.
* The repo has the following branches:
  * **`develop`** and **`main`**: these branches are dedicated to tracking the `develop`/`main`
    branches of the upstream [EasyBuild easybuild-easyconfigs repo](https://github.com/easybuilders/easybuild-easyconfigs)
    repo and should only be updated via fetching from the upstream - a tasks that is best done using
    the stack-builder tool. **Commits must never be made directly to these branches.**
  * **Institution/node specific branches:** `vitalit`, `ubelix`, `ibu`, `scicore`. These are the
    branches that nodes/institutions use as their **“work branch”**, i.e. where easyconfigs can be
    committed before they are contributed back to the EasyBuild project via a pull-request.

**The `sib-software-stack` repo:**
* Contains **private easyconfigs**, that are only shared among SIB software stack nodes and that
  are not intended to be contributed back to the EasyBuild project because they are not
  relevant to a wider audience, contain private sources, etc.
* Hosted at [`sib-software-stack`](https://github.com/sib-swiss/sib-software-stack).
* The repo has the following branches:
  * **`main`**: the main branch of the repo. Contains the consolidated *private easyconfigs* files
    used in the SIB software stack. New commits on this branch should only be added via
    pull-requests made from the node specific branches (or another temporary branch).
  * **Institution/node specific branches:** `vitalit`, `ubelix`, `ibu`, `scicore`. These are the
    branches that nodes/institutions use as their **“work branch”**, i.e. where easyconfigs can be
    committed before they merged into the `main` branch via a pull-request.

*Note:* the stack-builder tool is able to detect easyconfig files located on your own node
branch, on `develop`, as well as on all other node's branches. Therefore, if an easyconfig file is
e.g. added to the `vitalit` branch, then all other nodes will have access to this easyconfig file
even before it is merged into the `develop`/`main` branch of the repo.

<br>

### Git commit conventions
Whenever possible, **each commit should only contain changes for a single easyconfig** and its
associated files such as patches for instance. This makes rebasing and solving conflicts easier.
In other words, if you are adding/editing multiple easyconfigs, please make sure to commit the
changes in separate commits.

**Commit messages conventions:** (suggestion):
* `Add <name of easyconfig.eb>`: for a commit that adds an easyconfig to the repo.
* `Update <name of easyconfig.eb>`: for a commit that updates an existing easyconfig.
* If the commit is about modifying/adding a file that is not an easyconfig, prefix your commit with
  the component you are modifying. Here are some examples:
  ```
  package_list: add Armadillo-10.7.5-foss-2021a.eb
  config: update robot-path.
  README: update instructions
  ```

**Important:** please include the **full name of the easyconfig file** in the commit message, e.g.:
`Add patsearch-1-GCCcore-10.3.0-Perl-5.32.1.eb` and not `Add patsearch`.

<br>

### Workflow to contribute new easyconfigs to the software stack
The workflow to contribute a new easyconfig file - or update an existing file - is as follows:

* **Update your local instances** of the `sib-easyconfigs` and `sib-software-stack` repos using the
  stack-builder tool (`sb.py update`).  and your local copy of it.
  For more details on updating these two git repositories, see the
  [Updating the sib-easyconfigs repo section](#updating-the-sib-easyconfigs-repo) below.

* **Add the new/updated easyconfig file(s)** to your cluster-specific branch with a new commit.
  Please remember to make a separate commit for each easyconfig file. Commits should - as much as
  reasonably possible - follow the defined [Git commit conventions](#git-commit-conventions).

* **Run the EasyBuild CI/CD tests** on your new commit(s) by pushing changes on your node-specific
  branch to the remote. Tests results can be found under
  [GitHub Actions](https://github.com/sib-swiss/easybuild-easyconfigs/actions).
  *Note:* tests are currently only implemented in the `sib-easyconfigs` repo.

* After the new commit(s) tested successfully, a **pull-request** can be made to either:
    * The EasyBuild project for *public easyconfigs* stored in the `sib-easyconfigs` repo.
      Detailed instructions for this are given in the section
      [Contributing easyconfigs to the EasyBuild project](#contributing-easyconfigs-to-the-EasyBuild-project)
      below.
    * The `main` branch for *private easyconfigs* stored in the `sib-software-stack` repo. New
      pull requests can be [created here](https://github.com/sib-swiss/sib-software-stack/pulls).

* After your pull request is merged into the `develop` branch of the upstream
  `easybuild-easyconfigs` repo (for *public easyconfigs*) or the `main` branch of the
  `sib-software-stack` repo (for *private easyconfigs*), the merged commits will be automatically
  removed from your node branch the next time your run the stack-builder update command.

  *Note:* this auto-updating and removal of merged commit works best if each easyconfig is
  added/updated as a separate commit - which is why this has been emphasized earlier in the
  instructions. If conflicts arise during the rebasing of your node branch, the stack-builder
  tool will raise an error and your will need to perform the rebase manually.


<br>
<br>


## Contributing easyconfigs to the EasyBuild project
After a new easyconfig is successfully built and passes all CI/CD tests, it can be contributed to
the EasyBuild project. In this way, the file will be part of the next EasyBuild release.

Contributing easyconfigs to the main EasyBuild repository is done via GitHub **pull requests**
(PR). The creation of the PR can be done directly via an EasyBuild command, but it requires a
one-time setup to be performed, as explained in the
[pull-request one-time setup](#pull-request-one-time-setup) section below.

The essential commands and environment setup needed to contribute an easyconfig are given below,
but it is worthwhile to have a look at
[EasyBuild's documentation section](https://docs.easybuild.io/en/latest/Contributing.html#contributing-easyconfigs)
specific on this topic.

<br>

### Submitting a new pull request for an easyconfig
Before creating a new pull request (PR) to the EasyBuild project, make sure that all CI/CD tests
pass successfully for your easyconfig.

1. **Check the style and syntax** of the easyconfig file.
   ```
   eb --check-style <easyconfig>       # Run a style check on the easyconfig.
   eb --check-contrib <easyconfig>     # Run a style check and some other checks on the easyconfig.

   # Examples:
   eb --check-contrib HMMER2-2.3.2-GCC-10.3.0.eb
   eb --check-contrib BLAST+-2.11.0-gompi-2021a.eb
   eb --check-contrib easybuild/easyconfigs/h/HMMER2/HMMER2-2.3.2-GCC-10.3.0.eb
   ```
   *Note:* the easyconfig can be passed by its name (EasyBuild will find its location) or its
   full path.

2. **Create a new PR for the easyconfig**. Here the full path to the easyconfig file must be
   passed. If the package requires additional files, e.g. a patch file, these can be passed to the
   command too, as illustrated in the example below.

   *Note:* make sure that you have set the correct values for `github-user` and `github-org` in the
   EasyBuild `config.cfg` file (alternatively, pass the
   `--github-user=<GitHub user name> --github-org=sib-swiss` options to the `eb` command below).

   ```
   eb --new-pr <easyconfig> <additional files>

   # Examples:
   eb ---new-pr easybuild/easyconfigs/h/HMMER2/HMMER2-2.3.2-GCC-10.3.0.eb
   eb --github-user=sib-software --github-org=sib-swiss --new-pr easybuild/easyconfigs/i/InChI/InChI-1.06-GCC-10.3.0.eb easybuild/easyconfigs/i/InChI/InChI-1.06.patch
   ```

3. If the PR was submitted correctly, you should find it in the
   [Pull requests section](https://github.com/easybuilders/easybuild-easyconfigs/pulls)
   of the official EasyBuild repo.

   A PR will also create a new Git branch in the `sib-easyconfigs` repo. These branches have a
   naming scheme that looks like: `20211021150134_new_pr_SAMtools115` (here for SAMtools 1.1.5).

4. After the PR is submitted, someone from the EasyBuild team will review it (this can take a
   while), and either merge it to the main `develop` branch of the EasyBuild project or comment
   on the PR to ask for improvements/fixes. In principle GitHub will send you email notifications
   each time there is activity on the PR.

5. If a PR was submitted and should be updated (e.g. to fix a problem), the `--update-pr` command
   can be used. When using `--update-pr`, the PR number (found on GitHub) must be passed to the
   command.
   In addition, it is also necessary to pass the `--pr-commit-msg` option and give it a commit
   message that explains why the change was made.
   ```
   eb --update-pr <PR number on GitHub> <modified or new files> --pr-commit-msg "Explanation for the change"

   # Example:
   eb --update-pr 14186 easybuild/easyconfigs/r/RDKit/RDKit-2021.03.4-foss-2021a.eb --pr-commit-msg "Update RDKit-2021.03.4-foss-2021a.eb: correct checksum of patch file"
   ```

6. If you need to upload a test report (e.g. when the license does not allow tests by EasyBuild).
   ```
   eb --from-pr <PR number on GitHub> --rebuild --upload-test-report
   # Example:
   eb --from-pr 3153 --rebuild --upload-test-report
   ```

7. Once a PR is merged by the EasyBuild team, its Git branch can be deleted: either via the
   PR's GitHub page - example [here for a SAMtools PR](https://github.com/easybuilders/easybuild-easyconfigs/pull/14185)
   , or via command line.
   ```
   git push origin --delete <PR branch name>
   ```

<br>

### Pull-request one-time environment setup
Before being able to make pull requests (PR) to the EasyBuild project, the following one-time
setup must be performed. Additional info can be found
[here in the documentation](https://docs.easybuild.io/en/latest/Integration_with_GitHub.html#configuration).

1. **Install python dependencies** needed to handle GitHub pull requests that are made when running
   the EasyBuild GitHub integration commands (e.g. `eb --new-pr`).
   ```
   pip3 install --user pep8 GitPython keyring keyrings.cryptfile jeepney keyring_jeepney
   ```

2. If needed, **create a GitHub personal access token (PAT)** for your GitHub account:
   * Go to: https://github.com/settings/tokens/new and log in with your GitHub account.
   * Create a new token with only the **gist** and **public_repo** scopes (in the repo section)
     enabled.
   * Save the token in a password manager.
   <br>

3. **Specify the GitHub user and GitHub org to use**. This is best done by editing the
   `github-user` and `github-org` fields in the EasyBuild `config.cfg` file (note: the
   `github-org` field is actually already correct in the template).

   **Important:** the GitHub user name and organization must be entered all in lower case!

   Alternatively, one can also pass the options `--github-user=<user name> --github-org=sib-swiss`
   to the `eb` command.

4. **Install the GitHub PAT**. During this step, EasyBuild will ask you to enter the personal
   access token (PAT) you created earlier.
   ```
   eb --install-github-token
   eb --check-github --git-working-dirs-path=/home/vitsoft
   ```
   Note: the encrypted keyring storing the PAT is located at:
   `~/.local/share/python_keyring/cryptfile_pass.cfg`


<br>
<br>


## Updating the sib-easyconfigs repo and your local copy of the repo
Fetching updates for the `develop`/`main` branches of the
[sib-easyconfigs](https://github.com/sib-swiss/easybuild-easyconfigs) repo from the official
EasyBuild [easybuild-easyconfigs](https://github.com/easybuilders/easybuild-easyconfigs) repo
should be done regularly so that both your local instance and the remote instance of the
`sib-easyconfigs` repository are kept up-to-date.
It is also important to update before contributing new easyconfigs the EasyBuild project.

### Automated update using the stack-builder tool
Whenever possible, the recommended way to update the `sib-easyconfigs` and `sib-software-stack`
repositories is using the stack-builder `update` command:
```
sb.py update
```

### Manual update
If for some reason you wish/need to update the `sib-easyconfigs` repo manually, the procedure is
illustrated here for the `develop` branch:

1. Go to GitHub web-interface at https://github.com/sib-swiss/easybuild-easyconfigs and make sure
   the `develop` branch is selected.

2. Click on **Fetch upstream** > **Fetch and merge**.

3. Update the `develop` branch of your local copy of the `sib-easyconfigs` repo:
   ```
   git switch develop    # "git checkout develop" if you have an older version of Git.
   git pull
   ```
   *Note:* since the `develop` branch of the `sib-easyconfigs` repo always exactly tracks the
   `develop` branch of the official EasyBuild repo, the `git pull` operation should always lead
   to a *fast-forward* merge. Remember to never directly commit anything on the `develop` branch.

4. Make sure that your local node-specific branch (in the example below: `vitalit`) is up-to-date
   with the remote `origin/<node branch name>`, then rebase the node-specific branch on `develop`.
   Here is an example of commands for the `vitalit` node branch:
   ```
   git switch vitalit    # or "git checkout vitalit" if using an older version of Git.
   git fetch             # check whether history diverges between "vitalit" and "origin/vitalit".
   git pull              # or possibly "git reset --hard origin/vitalit" if local and remote history have diverged.
   git rebase develop
   git push --force      # WARNING: this will rewrite history on the remote. Only perform this
                         #          operation if you are sure the rebase was successful.
   ```

   During the rebase operation, any commit from your node-specific branch that was contributed to
   the `develop` branch (via a PR to the EasyBuild project) will automatically be removed from
   your node-specific branch.

   *Note:* the rebase step can sometimes be a little complicated if conflicts between the
   local node's (here `vitalit`) and `develop` branches occur. Don't hesitate to ask for help if
   needed.


<br>
<br>


## Useful EasyBuild commands
This section provides a **short list of useful EasyBuild commands**. A more comprehensive
documentation is available [here](https://docs.easybuild.io/en/latest/Using_the_EasyBuild_command_line.html).

**Search** for easyconfigs and display their content:
```
eb --search git               # Search for easyconfigs
eb --show-ec GCC-10.3.0.eb    # Show the content of an easyconfig
```

**Build** a software from an easyconfig. Add the `--dry-run` flag to do a test run:
```
eb --robot --rpath <easyconfig.eb> --dry-run
eb --robot --rpath <easyconfig.eb>
```

To **force a rebuild**, add the `--rebuild` flag. By default, EasyBuild will not rebuild an
already built software.
```
eb --robot --rpath --rebuild <easyconfig.eb>
```
