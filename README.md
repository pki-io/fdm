# fdm
Forked Dependency Manager

An experimental dependency manager for Go, based on vendoring using GitHub forks.

# Overview

Vendoring is the concept of taking a third-party dependency under your control. There are a number of different package managers out there for Go, some support vendoring while others don't. The typical approach seems to involve copying third-party code into your repository, probably with some sort of import rewriting.

fdm takes a different approach and assumes that you have forked the dependency. It uses symlinks to avoid having to do import rewriting.

The advantages of this approach are (hopefully):

* Follows GitHub friendly fork/PR workflow
* Keeps third-party code outside of your code repository resulting in a clean separation (gitignore \_vendor)
* No import rewriting required thanks to symlinks
* Controlled by a Vendors file, making third-party dependencies explicit
* Pass arguments to git so you have full control over which is clone (branch, tag, commit etc)

The disadvantages of this approach are:

* Doesn't automatically download all dependecies (by design)

# Installation

fdm is just a bash script, so simply download and stick it somewhere in PATH:

    $ curl -O https://raw.githubusercontent.com/pki-io/fdm/master/fdm && chmod +x fdm && mv fdm /usr/local/bin

# Usage

In your repository, create a file called Vendors

    vendor "github.com/pki-io/docopt.go" "github.com/docopt/docopt-go"
    vendor "github.com/pki-io/seelog" "github.com/cihub/seelog"
    vendor "github.com/pki-io/toml" "github.com/BurntSushi/toml"
    vendor "github.com/pki-io/go-homedir" "github.com/mitchellh/go-homedir"
    vendor "github.com/pki-io/ecies" "obscuren/ecies"

    vendor "github.com/pki-io/gojsonpointer" "github.com/xeipuuv/gojsonpointer"
    vendor "github.com/pki-io/gojsonreference" "github.com/xeipuuv/gojsonreference"
    vendor "github.com/pki-io/gojsonschema" "github.com/xeipuuv/gojsonschema"

    vendor_clone "github.com/pki-io/crypto" "golang.org/x/crypto"
    vendor_build "github.com/pki-io/crypto" "golang.org/x/crypto" "golang.org/x/crypto/pbkdf2"

    unforked_vendor_clone "github.com/pki-io/core"
    if [[ "${FDM_ENV:-}" != "DEV" ]]; then
      for d in config crypto document entity fs index node x509; do
        unforked_vendor_build "github.com/pki-io/core" "./$d/"
      done
    fi

This is just a bash script, so you can do loops etc as required.

__vendor__ SOURCE ALIAS [ARGS]

This clones the git repo from SOURCE and creates a symlink to ALIAS. It then builds using go install. All imports are assumed to be referring to ALIAS.

ARGS are passed to git clone.

__vendor\_clone__ SOURCE ALIAS [ARGS]

This only clones and create the symlink. ARGS are passed to git clone.

__vendor\_build__ SOURCE ALIAS [ARGS]

This only builds, but ARGS are passed to go install. This is required if the repo doesn't contain Go source files, but multiple package directories.

__unforked\_vendor__ SOURCE [ARGS]

This clones the git repo, but doesn't create any symlinks as it isn't a fork. It then installs using go install.

ARGS is passed to git.

__unforked\_vendor\_clone__ SOURCE [ARGS]

Only clones the repo. ARGS is passed to git.

__unforked\_vendor\_build__ SOURCE [ARGS]

Only builds, ARGS is passed to go install.

## It's just bash

Because the Vendors file is just bash, we can do clever things like group based on environment variable. In the above example, if fdm was called with FDM\_ENV set to DEV, then it would skip the unforked\_vendor\_build code loop.

## Calling fdm

Running the fdm command without any arguments tells fdm to process the Vendors file.

If you pass any arguements, fdm will rewrite GOPATH as required and will run go with the supplied arguments.

For example, you might run the following:

    # Process the Vendors, installing all dependencies...
    $ fdm

    # Run the go tests
    $ fdm test
