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

    $ fdm --help
    Usage: fdm [--help] [--debug] [--dev] [--exec COMMAND] [GO_ARGS]

    If no arguments are given, fdm loads the Vendors file
    Options:
        --help              This help
        --debug             Print debug info
        --dev               Enable dev mode. Use this to ignore the --git switch and use master
        --exec COMMAND      Execute COMMAND with modified GOPATH
        GO_ARGS             Any other arguments passed to go with modified GOPATH

    Vendors file
    ------------

    The vendors file is just a bash script, but fdm provides the 'vendor' function which works as follows:

    vendor --repo REPO [--clone|--build] [--fork REPO] [--git COMMAND] [--pkg PACKAGE]

    Options:
        -r, --repo REPO     Repository or upstream. E.g. github.com/them/their-project or github.com/you/your-project
        -c, --clone         Only perform clone actio
        -b, --build         Only perform build action
        -f, --fork REPO     Fork (vendored) repository E.g. github.com/you/their-project
        -g, --git COMMAND   Git command to run after clone. E.g. 'checkout 5c68cfdf2a545b5ff576c075b459d1fc0c606f82'
        -p, --pkg PACKAGE   Package to build if repo contains multiple packages
        --force             Force git command even in dev mode

# Vendors file

In your repository, create a file called Vendors. E.g.

    vendor -f "github.com/pki-io/docopt.go" -r "github.com/docopt/docopt-go" -g "checkout 854c423c810880e30b9fecdabb12d54f4a92f9bb"
    vendor -f "github.com/pki-io/seelog" -r "github.com/cihub/seelog" -g "checkout 8762324945b8f31888f8bfa3bd7fca04c20e2639"
    vendor -f "github.com/pki-io/toml" -r "github.com/BurntSushi/toml" -g "checkout 443a628bc233f634a75bcbdd71fe5350789f1afa"
    vendor -f "github.com/pki-io/go-homedir" -r "github.com/mitchellh/go-homedir" -g "checkout 7d2d8c8a4e078ce3c58736ab521a40b37a504c52"
    vendor -f "github.com/pki-io/ecies" -r "github.com/obscuren/ecies" -g "checkout 582e689ca8661237e08b02068435199ea8a55318"

    vendor -f "github.com/pki-io/gojsonpointer" -r "github.com/xeipuuv/gojsonpointer" -g "checkout 636edb2500d21f2ed09ea96a00deb36bbd07cf70"
    vendor -f "github.com/pki-io/gojsonreference" -r "github.com/xeipuuv/gojsonreference" -g "checkout 2df3c0c802434c5cb984dbc21425f5960bda4d16"
    vendor -f "github.com/pki-io/gojsonschema" -r "github.com/xeipuuv/gojsonschema" -g "checkout 71b85f61a135e79143f3d3238d5175a4f29b6689"

    vendor --clone -f "github.com/pki-io/crypto" -r "golang.org/x/crypto" -g "checkout 5c68cfdf2a545b5ff576c075b459d1fc0c606f82"
    vendor --build -f "github.com/pki-io/crypto" -r "golang.org/x/crypto" -p "pbkdf2"

    vendor --clone -r "github.com/pki-io/core" -g "checkout 03b63a18d74d2050828f7ab7d6943b114fdc1946"
    if [[ "${FDM_ENV:-}" != "DEV" ]]; then
      for d in config crypto document entity fs index node x509; do
        vendor --build -r "github.com/pki-io/core" -p "$d"
      done
    fi

This is just a bash script, so you can do loops etc as required.


## It's just bash

Because the Vendors file is just bash, we can do clever things like group based on environment variable. In the above example, if fdm was called with FDM\_ENV set to DEV, then it would skip the unforked\_vendor\_build code loop.

# Examples

    # Process the Vendors, installing all dependencies at the specified versions...
    $ fdm

    # Process the Vendors, but using master
    $ fdm --dev

    # Run the go tests
    $ fdm test
    
    # Run something else with the correct GOPATH
    $ fdm --exec gover
