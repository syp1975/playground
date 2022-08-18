# .playground
![Version](https://img.shields.io/github/v/release/syp1975/playground?include_prereleases&style=for-the-badge&label=)
![License](https://img.shields.io/github/license/syp1975/playground?color=gray&style=for-the-badge&label=)
![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-%23121011.svg?style=for-the-badge&logo=gnu-bash&logoColor=white)
![GitHub](https://img.shields.io/badge/github-%23121011.svg?style=for-the-badge&logo=github&logoColor=white)

Retrieves scripts, snippets and playbooks stored in a remote repository. For Ubuntu, Bash and Ansible.

It may work on other debian based systems, but it will only work with bash.

It has been quite a long time for me playing around linux, keeping snippets on many places, searching for installation instructions, discovering ansible,
but at the end I was tired of searching the same info over an over and copying pasting.
That is the moto that brings us here, a simple simple solution (one bash script) to keep all this information on github and no need to even clone the repo to use it.
Of course, don't expect any performance running small scripts with this, you will have to wait for the script to be downloaded each time you run it.

**\[ [Usage](#usage) | [Some examples](#some-examples) \]\[ [Installing, upgrading and uninstalling](#installing-upgrading-and-uninstalling) \]\[ [Configuration](#configuration) | [Adding repositories](#adding-repositories) ]**

# Usage
To run a bash script from a github repo you would exec something like this on your command line:
```sh
curl -sL https://raw.githubusercontent.com/syp1975/playground/main/helloworld | bash
```

where:
* ``syp1975`` is the user or organization name.
* ``playground`` is the repository name.
* ``main`` is the tag name.
* ``helloworld`` is the path to the script.

To simplify, I had created the ``pg`` command to run the scripts right away from the command line. If you run:
```sh
pg
```
It will display its usage:
```
Playground.
Retrieves scripts, snippets and ansible playbooks from a remote repository/website.
Usage:
  pg [@<repo>] <file> [...]: executes a script (with bash).
  pg [@<repo>] debug <file> [...]: debugs a script (with bash -x).
  pg [@<repo>} sudo <file> [...]: executes a script (with sudo bash)
  pg [@<repo>] help <file>: shows help for a script.
  pg [@<repo>] url <file>: writes to stdout the full url that points to the file.
  pg [@<repo>] cat <file>: writes to stdout the contents of the file.
  pg [@<repo>] play <<file> ..> [...]: runs with ansible-playbook a list of playbooks.
  pg src [<repo> [--default]]: selects a source repository.
    --default: also sets the default selected repository at login.
    With no parameters, prints the selected repository name.
  pg add <repo> [--force] [--auth <token>] [--download] [<uri>] [...]: adds a source repository.
    --force: overwrites an existing configuration file.
    --download: downloads the configuration file from the uri instead of using the uri as the source.
    --auth <token>: github personal access token to access a private repository.
    If no <uri> is specified, creates an empty file and opens the editor.
  pg edit [<repo>]: opens the configuration file of the repository with the editor.
  pg push [message]: commits and pushes (with git) all modifications, deletions and additions in the working tree of the current folder.
Where:
  repo: name of a file located in ~/.playground/rc/ that contains the configuration of the source repository.
    Only alphanumerics, dashes and underscores are accepted in the name.
  uri: base url to download a file from the repository, including ending slashes if needed.
  file: path of the file to be retrieved from the repository.
  ...: optional parameters passed to the script/ansible-playbook/curl.
```

Not yet implemented:
* ``pg [@repo] list [filter]``: displays a list of files from the repository.

When a repository is selected, either with the ``@repo`` parameter or with the ``pg src repo`` command, *playground* will *source* the configuration for a repository from this shell scripts:
1. ``~/.playground/rc/<repo>``
2. ``~/.playground/rc/<repo>.<hostname>``

The values of the variables defined on the per host configuration file, will overwrite the values of the variables of the main configuration file.

When executing scripts trough privileged scalation (sudo), remember to use ``pg sudo`` command instead of ``sudo pg``.
This is because the playground and repository configurations files are sourced instead of being executed as a command and sudo can not execute shell builtin commands.

## Some examples
```sh
#play an ansible playbook from a repository shared with company coworkers that has basic http authentication:
pg add company https://user:password@mycompany.com/playground/
pg @company play my-company-playbook.yaml

#download repository configuration files from a private repository on github:
pg add private --auth "mygithub-personal-access-token" --download https://raw.githubusercontent.com/myuser/myprivaterepo/mytag/rc/private
pg add private.$(hostname) --auth "mygithub-personal-access-token" --download https://raw.githubusercontent.com/myuser/myprivaterepo/mytag/rc/private.$hostname
pg @private cat secrets.txt

#use company repository from here ...
pg src company
pg my-company-bootstrap-script
pg my-company-it-tools-install-script
#... to here

#return to playground repo
pg src playground
pg helloworld
```

# Installing, upgrading and uninstalling
Execute this command to install *playground*:
```sh
curl -sL https://raw.githubusercontent.com/syp1975/playground/main/install | bash
```

Test the installation with this:
```sh
pg helloworld
```
It should output ``Hello world!`` if everything is fine.

Upgrade with:
```sh
pg @playground install
```

For uninstalling, remove the .playground folder:
```sh
rm -r ~/.playground
```

# Configuration
The install script creates a ``.playground`` folder in your home folder. You can go there with:
```sh
cd ~/.playground
```

It contains the following:
* **[playground](https://raw.githubusercontent.com/syp1975/playground/main/playground)**<br>
  This is the only one script downloaded to your system.
* **rc/**<br>
  A folder to keep all the configuration files for your repositories (**r**epo **c**onfigs).
  Keep this folder on a private github repository or link it to a cloud storage,
  this would serve as backup and would ease the deploy on other hosts.
* **rc/playground**<br>
  The configuration file for the playground repository.
  
## Adding repositories
If we want to add a repository, we could:
* Start from scratch:
  ```sh
  pg add newrepo
  ```
  This will create an empty configuration file and open the editor on it. Start with something like:
  ```sh
  #!/usr/bin/env bash
  #the base url or your repository is required (include ending slash if required)
  PGSRC=""
  #your personal access token (for github private repos)
  PGPAT=""
  #additional parameters passed to curl
  PGEXTRA=""

  #add additional variables used by your scripts here
  ```
* Add a source repository and customize its configuration file:
  ```sh
  pg add publicrepo https://raw.githubusercontent/organization/publicrepo/
  pg edit publicrepo
  ```
* Download a configuration file from a private repository that do not needs customization:
  ```
  pg add privaterepo --auth "your_github_personal_access_token_here" --download http://raw.githubusercontent.com/yourorganization/yourprivaterepo/rc/privaterepo
  ```
  Optionally we could add a per host configuration file:
  ```
  pg add privaterepo.$(hostname) --auth "your_github_personal_access_token_here" --download http://raw.githubusercontent.com/yourorganization/yourprivaterepo/rc/privaterepo.$(hostname)
  ```
* For development purposes, clone a repository to a local folder and use this folder as a source repository for *playground*.<br>
  The minimum contents of the repository configuration file should be something like:
  ```sh
  #!/usr/bin/env bash
  PGSRC=file:///full/path/to/your/local/repository/
  ```
