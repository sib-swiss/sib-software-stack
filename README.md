# SIB software stack builder and documentation

The SIB software stack is a collaborative effort between different SIB units to deploy a software
stack based on the [EasyBuild project](https://easybuild.io).

The current version of the SIB software stack is built on EasyBuild version `4.4.2`.

The full documentation for EasyBuild [is found here](https://docs.easybuild.io/en/latest).


<br>
<br>


## Setting-up of the local build environment

### Cloning the software stack builder and documentation repo (this repo)
Clone the present repo to your build machine, so that you can use the EasyBuild config template,
the list of packages to build and the builder script.
```
git clone https://gitlab.sib.swiss/software/sib_software_stack.git sib_software_stack.git
```

This repo contains the following files:
* `config.cfg`: an easybuild config template file. Make a copy of the file, adapt the paths it
  contains to your local environment, and indicate the location of the config file via the
  `EASYBUILD_CONFIGFILES` environment variable (see next section).
* `sib_stack_package_list.txt`: list of software packages (easyconfig file names) to be built.
* `stack_builder.py`: wrapper script (runs with python3) around EasyBuild to automate the build
  of the entire software stack.
* `secrets`: encrypted credentials (see `README.md` inside the directory).

<br>

### Installing and configuring EasyBuild
EasyBuild is distributed as python module and can be installed and upgraded with `pip`. Note that
upon installation, EasyBuild provides a number of easyconfig files (files that contain the recipe
to build a given software) stored in `~/.local/easybuild/easyconfigs`.
```
pip3 install --user easybuild==4.4.1
pip3 install --upgrade --user easybuild
```

Environment variables to add to your `.bashrc`.
```
# EasyBuild configuration
export EB_PYTHON=python3                                           # Make sure that EasyBuild uses python3.
export EASYBUILD_CONFIGFILES=~/sib_software_stack.git/config.cfg   # Path to the EasyBuild config file.
```

To test the installation of EasyBuild, run: `EB_VERBOSE=1 eb --version`.

Note: an alternative to setting `EASYBUILD_CONFIGFILES` is to store the config file at the location
where EasyBuild searches by default: `~/.config/easybuild/config.cfg`.

<br>

### Cloning the sib-easyconfigs repo
The [sib-easyconfigs GitHub repo](https://github.com/sib-swiss/easybuild-easyconfigs) is a fork of
the original [EasyBuild project easyconfig repo](https://github.com/easybuilders/easybuild-easyconfigs).
It contains the easyconfig files that are specific to the SIB software stack as well as the latest
versions of easyconfig files from the EasyBuild project.

To clone the repo (either via HTTPS or SSH):
```
git clone https://github.com/sib-swiss/easybuild-easyconfigs.git sib-easyconfigs.git
git clone git@github.com:sib-swiss/easybuild-easyconfigs.git sib-easyconfigs.git
```

<br>

### Additional configurations
In order to allow for node specific software to be installed - i.e. software that should only be
built and deployed on specific nodes/clusters - the `SIB_SOFTWARE_STACK_NODE` environment variable
must be set. See the [package list file](#the-package-list-file) section for details.

The possible values for this environment variable are:
* `IBU` for the IBU cluster.
* `SCI` for sciCORE.
* `UBE` for the UBELIX cluster.
* `VIT` for the SIB lausanne (Vital-IT/Core-IT).

Example of value to add to your `.bashrc`.
```
export SIB_SOFTWARE_STACK_NODE="VIT"
```

<br>
<br>


## Using the software stack builder script
`stack_builder.py` is a convenience wrapper around EasyBuild that automates the build of the SIB
software stack. Essentially, this script parses the `sib_stack_package_list.txt` file and build the
listed software packages in the specified order (as well as all their required dependencies).

To build the entire software stack, the following commands can be used:
```
stack_builder.py --summary   # Display a summary of the tasks to perform.
stack_builder.py --dry-run   # Do a test-run: this will check that all easyconfig files are found.
stack_builder.py             # Run the actual build.
```

All available options and their shortcuts can be displayed with `./stack_builder.py --help`

**Warning:** in very rare cases, system libraries have to be installed on the base OS of the
machine building the software stack. This is because these libraries are not ported to EasyBuild.
As an example, when building `R 4.0.3` on a CentOS 7 machine, the following system dependencies
have to be installed: `sudo yum install openssl-devel libibverbs.x86_64 rdma-core-devel`.

<br>

### The package list file
`sib_stack_package_list.txt` is a plain text file that contains the list of software packages to be
built, in the order they should be built (top to bottom).

When the file is parsed by `stack_builder.py`, the following rules are applied:
* Text located after a `#` (comments) and empty lines are ignored.
* Lines prefixed with one or more node name abbreviations (comma separated) indicate that the given
  package should only be built if the list of abbreviations in the prefix contains the value
  given in the `SIB_SOFTWARE_STACK_NODE` environment variable or via the `--node` argument of
  `stack_builder.py`. The list of abbreviations is given in the [additional configuration](#additional-configurations)
  section.

  Examples:
    * The following line indicates that the `Rgurobi-9.1.2` software should only be built on the
      Lausanne cluster infrastructure.
      ```
      VIT Rgurobi-9.1.2-foss-2021a-R-4.1.0.eb
      ```
    * The following line indicates that the `Rgurobi-9.1.2` software should only be built on the
      Lausanne cluster and the IBU cluster infrastructure.
      ```
      VIT,IBU Rgurobi-9.1.2-foss-2021a-R-4.1.0.eb
      ```

   **Important:** as one of the objectives of the SIB software stack is to standardize the software
   stacks across the different SIB-maintained clusters, the number of site-specific packages should
   be kept to a minimum.

When searching for an easyconfig file specified in `sib_stack_package_list.txt`, EasyBuild searches
the different paths listed in the `robot-paths` variable of the EasyBuild `config.cfg` file.
In the SIB software stack, this variable contains two paths by default:
* `~/.local/easybuild/easyconfigs`: the "default", stable, version of easyconfig files for the
  installed version of EasyBuild. These files are "frozen" (so to speak) and will only be updated
  if the installed instance of EasyBuild is updated to a new version.
* `~/sib-easyconfigs.git/easybuild/easyconfigs`: this is a fork of the "development" repo of
  EasyBuild, enriched with the easyconfig files that are contributed by individual nodes
  participating in the SIB software stack. Please note that this repo will also contain the latest
  "development" versions of easyconfig files as they get updated/added by the EasyBuild project.

If a given easyconfig is found in both of the above paths, the version of the file found in the
path listed first (in our case `~/.local/easybuild/easyconfigs`) is used and thus overrides the
version of the file found in the second path.

In the SIB software stack, we use this functionality to ensure that we primarily use the "stable"
versions of easyconfig files, and only use our own easyconfigs or "development" versions of
easyconfig files when no "stable" version is available.

To override a "stable" easyconfig with one from the "development" repo, the syntax
`SIB_EB_REPO/path/to/easyconfig.eb` can be be used.
Example:
```
SIB_EB_REPO/r/R/R-4.1.0-foss-2021a.eb
```


<br>
<br>


## Sharing of easyconfig files between nodes
SIB-wide collaboration on the SIB software stack is done via the Git repo
[sib-easyconfigs](https://github.com/sib-swiss/easybuild-easyconfigs). This is a fork of the
original [EasyBuild easybuild-easyconfigs repo](https://github.com/easybuilders/easybuild-easyconfigs)
and contains a branch named `sib` where easyconfig files produced by nodes can be added and shared
before they are contributed back to the main EasyBuild repo.

<br>

### Proposed workflow to share easyconfigs
Steps to share easyconfig files via the [sib-easyconfigs](https://github.com/sib-swiss/easybuild-easyconfigs)
repo:
* New or updated easyconfig files are first added to a node-specific branch with a new commit.
* Commits should follow the [Git commit conventions](#git-commit-conventions) - see below.
* To run the EasyBuild CI/CD tests on new commits, the can be pushed to the remote.
* After the new easyconfig file(s) is(are) tested successfully, the commit(s) are merged into the
  `sib` branch of the repo, and pushed to the remote.
* All participating nodes regularly update/sync their local version of the `sib` branch so that
  they can benefit from files added by other nodes.
* Successfully built easyconfigs can be contributed back to the main EasyBuild project repo via
  pull requests (PR) to the project - unless they are highly specific and are not meaningful for
  the EasyBuild project.
* **Important:** never commit anything directly to the `develop` branch of the repo. This branch is
  dedicated to tracking the `develop` branch of the upstream
  [EasyBuild easybuild-easyconfigs repo](https://github.com/easybuilders/easybuild-easyconfigs)
  repo and is only updated via fetching from the upstream.

<br>

### Git commit conventions:
Each commit should only contain changes for a single easyconfig (and possibly its patches, if any).
This makes rebasing and solving conflicts easier.

Commit messages should be follow the conventions:  
* `Add <name of easyconfig.eb>`: for a commit that adds an easyconfig destined to be contributed
  to the main EasyBuild repo at some point.
* `SIB: add <name of easyconfig.eb>`: for a commit that adds an easyconfig that will only be used
  by SIB (not contributed back to EasyBuild because it is not useful to the project).
* If an existing file is updated, `Update` should be used instead of `Add` in the commit message.

<br>

### Example illustrated with Git commands:
* The Vital-IT node writes an easyconfig file for a new software (or updates an existing file).
  The commit is initially made on Vital-IT's own branch `vital-it` and pushed to the remote so
  that CI/CD tests can be run.
   ```
   git switch vital-it
   git commit -m "SIB: add NewSoftware-1.0.0-GCC-10.3.0.eb"
   git push
    ```
* If the software builds as expected (locally) and all CI/CD tests succeed on GitHub, the changes
  can now be merged to the `sib` branch and pushed to the remote.
   ```
   git fetch
   git switch sib
   git pull             # or possibly "git reset --hard origin/sib" if local and remote history have diverged.
   git switch vital-it
   git rebase sib       # only needed if the "vital-it" branch is behind the "sib" branch.
   git switch sib
   git merge vital-it
   git push sib
   ```
* Other nodes can update their local version of the `sib` branch.
   ```
   git switch sib
   git pull
   ```


<br>
<br>


## Contributing easyconfigs to the EasyBuild project
After a new easyconfig is successfully built, added to the `sib` branch of `sib-easyconfigs`, and
passes all CI/CD tests, it can be contributed to the EasyBuild project. In this way, the file will
be part of the next EasyBuild release.

Contributing easyconfigs to the main EasyBuild repository is done via GitHub **pull requests** (PR).
The creation of the PR can be done directly via an EasyBuild command, but it requires a one-time
setup to be performed, as explained in the [pull-request one-time setup](#pull-request-one-time-setup)
section below.

The essential commands and environment setups needed to contribute an easyconfig are given below,
but it is worthwhile to have a look at the [specific documentation section](https://docs.easybuild.io/en/latest/Contributing.html#contributing-easyconfigs)
on this topic.

<br>

### Updating the SIB fork of "easybuild-easyconfigs"
Before contributing new easyconfigs to the official EasyBuild easybuild-easyconfigs repo, it is
good practice to update the `develop` and `sib` branches of the SIB fork with new content from the
*upstream* repo: [easybuild-easyconfigs](https://github.com/easybuilders/easybuild-easyconfigs).

This can be done with the following steps:

1. Go to https://github.com/sib-swiss/easybuild-easyconfigs and make sure the `develop` branch
   is selected.

2. Click on **Fetch upstream** > **Fetch and merge**.

3. Now that the SIB easybuild-easyconfigs repo is up-to-date with the official repo, update the
   `develop` branch of your local copy of the SIB repo.
   ```
   git switch develop    # "git checkout develop" if you have an older version of Git.
   git pull
   ```
   **Note:** since the `develop` branch of the SIB repo always exactly tracks the `develop` branch
   of the official EasyBuild repo, the `git pull` operation should always leads to a *fast-forward*
   merge. Remember to never directly commit anything on the `develop` branch.

4. Make sure that your local `sib` branch is up-to-date with the remote (`origin/sib`), then rebase
   the `sib` branch on `develop`.
   **Warning:** the rebase step can sometimes be a little complicated if conflicts between the
   `sib` and `develop` branches occur. Don't hesitate to ask for help if needed.

   During the rebase operation, any commit containing an easyconfig that has - in the meantime -
   been accepted by the EasyBuild project and merged by them in the official EasyBuild `develop`
   branch will automatically be removed from the `sib` branch during the rebase.
   ```
   git switch sib
   git fetch             # check whether history diverges between "sib" and "origin/sib".
   git pull              # or possibly "git reset --hard origin/sib" if local and remote history have diverged.
   git rebase develop
   ```

<br>

### Submitting a new pull request for an easyconfig
Before creating a new pull request (PR) to the main EasyBuild repo, verify that the easyconfig
file was merged into the `sib` branch and that all CI/CD tests succeed.

1. **Check the style and syntax** of the easyconfig file.
   ```
   eb --check-style <easyconfig>       # run a style check on the easyconfig.
   eb --check-contrib <easyconfig>     # run a style check and some other checks on the easyconfig.

   # Examples:
   eb --check-contrib HMMER2-2.3.2-GCC-10.3.0.eb
   eb --check-contrib BLAST+-2.11.0-gompi-2021a.eb
   eb --check-contrib easybuild/easyconfigs/h/HMMER2/HMMER2-2.3.2-GCC-10.3.0.eb
   ```
   Note: the easyconfig can be passed by its name (EasyBuild will find its location) or its full path.

2. **Create a new PR for the easyconfig**. Here the full path to the easyconfig file must be
   passed. If the package requires additional files, e.g. a patch file, these can be passed to the
   command as in the example below.

   Make sure that you have set the correct values for `github-user` and `github-org` in the
   EasyBuild `config.cfg` file (alternatively, pass the
   `--github-user=<GitHub user name> --github-org=sib-swiss` options to the `eb` command below).
   ```
   eb --new-pr <easyconfig> <additional files>

   # Examples:
   eb ---new-pr easybuild/easyconfigs/h/HMMER2/HMMER2-2.3.2-GCC-10.3.0.eb
   eb --github-user=sib-software --github-org=sib-swiss --new-pr easybuild/easyconfigs/i/InChI/InChI-1.06-GCC-10.3.0.eb easybuild/easyconfigs/i/InChI/InChI-1.06.patch
   ```

3. If the PR was submitted correctly, you should find it in the [Pull requests section](https://github.com/easybuilders/easybuild-easyconfigs/pulls)
   of the official EasyBuild repo.

   A PR will also create a new Git branch in the `sib-easyconfigs` repo. These branches have a
   naming scheme that looks like: `20211021150134_new_pr_SAMtools115` (here for SAMtools 1.1.5).

4. After the PR is submitted, someone from the EasyBuild team will review it (this can take a
   while), and either merge it to the main `develop` branch of the EasyBuild project or comment on
   the PR to ask for improvements/fixes. In principle GitHub will send email notifications each
   time there is activity on the PR.

5. If a PR was submitted and should be updated (e.g. to fix a problem), the `--update-pr` command
   can be used. When using `--update-pr`, the PR number (found on GitHub) must be passed to the
   command.
   In addition, it is also necessary to pass the `--pr-commit-msg` option and give it a commit
   message that explains why the change was made.
   ```
   eb --update-pr <PR number on GitHub> <modified or new files> --pr-commit-msg "Explanation for the change"

   # Example:
   eb --update-pr 14186 easybuild/easyconfigs/r/RDKit/RDKit-2021.03.4-foss-2021a-Python-3.9.5.eb --pr-commit-msg "Update RDKit-2021.03.4-foss-2021a-Python-3.9.5.eb: correct checksum of patch file"
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

### Pull-request one-time setup
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
   `github-user` and `github-org` fields in the EasyBuild `config.cfg` file (note: the `github-org`
   field is actually already correct in the template).

   **Important:** the GitHub user name and organization must be entered all in lower case!

   Alternatively, one can also pass the options `--github-user=<user name> --github-org=sib-swiss`
   to the `eb` command.

4. **Install the GitHub PAT**.
   ```
   eb --github-user=<GitHub user name> --install-github-token
   eb --check-github --git-working-dirs-path=/home/vitsoft
   ```
   Note: the encrypted keyring storing the PAT is located at:
   `~/.local/share/python_keyring/cryptfile_pass.cfg`


<br>
<br>


## Useful EasyBuild commands
This section provides a short list of useful EasyBuild commands. A more comprehensive documentation
is available [here](https://docs.easybuild.io/en/latest/Using_the_EasyBuild_command_line.html).

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

To **force a rebuild**, add the `--rebuild` flag. By default, EasyBuild will not rebuild an already
built software.
```
eb --robot --rpath --rebuild <easyconfig.eb>
```
