# Surf: A Build Server for GitHub

Surf is a multi-platform, language-agnostic, GitHub oriented server for building your apps continuously that is easy to set up, works on every operating system, and is way less of a pain in the ass than anything else out there. Since Surf is built around Git and GitHub, its configuration is vastly simpler than other build servers and since it's built on node.js, installing it is really easy.

Philosophically, Surf tries to be really simple - it gives you the reusable pieces you need to easily create simple build systems, and have the ability to make more complicated ones if you need to. Architecturally, Surf's design is similar to [BuildBot](http://buildbot.net), but Git / GitHub focused. Some design inspiration, from BuildBot's website:

> Many CI tools, such as CruiseControl or Jenkins, are structured as ready-to-use applications. Users fill in specific details, such as version control information and build process, but the fundamental design is fixed and options are limited to those envisioned by the authors. This arrangement suits the common cases quite well: there are cookie-cutter tools to automatically build and test Java applications, Ruby gems, and so on. Such tools embody assumptions about the structure of the project and its processes. They are not well-suited to more complex cases, such as mixed-language applications or complex release tasks, where those assumptions are violated.

> Buildbot's design allows your installation to grow with your requirements, beginning with simple processes and growing to meet your unique needs. 

### Why would I use this over $ALTERNATIVE?

* Surf works on every platform, really easily
* Surf works even if you can't set up a WebHook (either because of GitHub privileges, firewall issues, whatever).
* Surf works inside your company firewall, without needing to carve out exceptions. As long as you can make outgoing HTTP requests, it'll work.

### Why should I use $ALTERNATIVE instead?

* Surf only works with Git and GitHub - if you use Subversion or Bitbucket, you're out of luck
* Surf doesn't have any of its own UI, so if you like lots of knobs and buttons to click, you might be happier with Jenkins
* Surf is very early and doesn't have a huge plugin community (or, plugins!)  Language-specific build servers might be easier to set up.

### Try it out by building Surf itself:

First, install Surf:

```sh
npm install -g surf-build
```

Now, try running `surf-build`, which is a command-line app that knows how to build different kinds of projects.

```sh
surf-build --repo https://github.com/surf-build/surf -s 805230d579cb49ffd7e33ee060023baebaf203e5
```

Tada! You made a build. 

### Setting up Continuous Build

Creating a continuous build isn't much harder - first, do the following:

1. Go to https://github.com/settings/tokens to get a token
1. Make sure to check `repo` and `gist`.
1. Generate the token and save it off

Open a Console tab and run:

```sh
export GITHUB_TOKEN='<< your token >>'
surf-server surf-build/surf
```

This is the equivalent of a Jenkins Master, but really only exists so that you won't run out of GitHub API calls per-hour.

Now, open up another tab and set up a client that will run builds for us:

```sh
export GITHUB_TOKEN='<< your token >>'
surf-client -s http://localhost:3000 -r https://github.com/surf-build/surf -- surf-build
```

That's it! Every time someone pushes a PR or change to Surf, your computer will clean-build the project. Since you (probably) don't have write permission on the Surf repo, you can't save the results to GitHub. 

## How does Surf know to build my project?

The most straightforward way (and one of the only ways at the moment), is to have a script at the root of the repo called `build.sh` and `build.cmd/ps1` for Windows. Future versions of Surf will know how to build common project types automatically, so you won't have to do this.

## How to set up builds against GitHub PRs

Surf is great at running builds to verify your PRs, which show up here on the GitHub UI:

![](http://cl.ly/0Q0S0A233I0u/Fix_miscellaneous_Windows_bugs_by_paulcbetts__Pull_Request_7__surf-buildsurf_2016-01-27_21-51-35.png)

To set this up, all we need to do is pass `-n` to `surf-build`: 

```sh
export GITHUB_TOKEN='<< your token >>'
surf-client -s http://localhost:3000 -r https://github.com/surf-build/example-csharp -- surf-build -n 'surf-win32-x64'
```

Pass a descriptive name as your parameter to `-n`, usually the platform / architecture that you're building on. The build output will be a link on the checkmark, and posted to your account as a GitHub Gist. Check out an example: https://gist.github.com/paulcbetts/b6ab52eeb43d0c551516.

## Available Commands

### `surf-server`

Sets up a build server which allows clients to query GitHub without running out of API calls. In the future, this will also run a small status page.

```
Usage: surf-server owner/repo owner2/repo owner/repo3...
Runs a web service to monitor GitHub commits and provide them to Surf clients

Options:
  -h, --help  Show help                                                [boolean]
  -p, --port  The port to start the server on

Some useful environment variables:

SURF_PORT - the port to serve on if not specified via -p, defaults to 3000.
GITHUB_ENTERPRISE_URL - the GitHub Enterprise URL to use.
GITHUB_TOKEN - the GitHub API token to use. Must be provided.
```

### `surf-client`

Monitors a GitHub repo and runs a command on every changed ref, constrained to a certain number of processes in parallel.

```
sage: surf-client -s http://some.server -r https://github.com/some/repo --
command arg1 arg2 arg3...
Monitors a GitHub repo and runs a command for each changed branch / PR.

Options:
  -h, --help    Show help                                              [boolean]
  -s, --server  The Surf server to connect to
  -r, --repo    The URL of the repository to monitor
  -j, --jobs    The number of concurrent jobs to run. Defaults to 2

Some useful environment variables:

GITHUB_ENTERPRISE_URL - the GitHub Enterprise URL to use instead of .com.
GITHUB_TOKEN - the GitHub (.com or Enterprise) API token to use. Must be
provided.
```

`surf-client` will set a few useful environment variables to the command that it runs for every changed branch:

* `SURF_REPO` - the repository URL to use
* `SURF_SHA1` - the commit to build

### `surf-build`

Clones a repo from GitHub, checks out the specified commit, and builds the project (currently via `build.sh` / `build.cmd` but in the future will know how to build various projects).

```
Usage: surf-build -r http://github.com/some/repo -s SHA1
Clones a repo from GitHub and builds the given SHA1

Options:
  -r, --repo  The repository to clone
  -s, --sha   The sha to build
  -n, --name  The name to give this build on GitHub

Some useful environment variables:

GITHUB_ENTERPRISE_URL - the GitHub Enterprise URL to (optionally) post status
to.
GITHUB_TOKEN - the GitHub (.com or Enterprise) API token to use. Must be
provided.
GIST_ENTERPRISE_URL - the GitHub Enterprise URL to (optionally) post Gists to.
GIST_TOKEN - the GitHub (.com or Enterprise) API token to use to create the
build output Gist.

SURF_SHA1 - an alternate way to specify the --sha parameter, provided
            automatically by surf-client.
SURF_REPO - an alternate way to specify the --repo parameter, provided
            automatically by surf-client.
```
