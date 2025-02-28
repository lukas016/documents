# Overview

This document describes the standardization and minimal requirements for software development under Open Sovereign Cloud.

This isn't a bible. Everything here can be improved, but every change must always be discussed.

**Mandatory:** These must be followed.
    If you cannot follow the mandatory requirements or wish to change/improve them, it must be discussed with OSC maintainers and documented (a comment will suffice).

**Recommendation:** This section contains best practices and solutions that we already use in other projects and that should be followed.

OSC Maintainer:

- Lukas Koszegy

## Logging

**Mandatory:**

- The application must support JSON log format.
- You must be able to set a minimal log level for logs to be printed.
- The timestamp must support RFC3339 format, ideally with milliseconds or nanoseconds.
- The default log output must be stderr or configurable.
- All logs and panics must be logged in the same format.
- The application must print the application name, version, and build date during startup.
- Do not use log.Fatal or any other methods that call os.Exit.
- The following fields and their names are required:
    - level
    - ts (format: RFC3339Nano)
    - msg
    - error (only for logging errors)
    - Examples:
        {"level":"info","ts":"2025-02-21T15:44:29.438958073Z","logger":"setup","msg":"pprof server stopped serve new connections"}
        {"level":"error","ts":"2025-02-21T15:44:28.435820546Z","logger":"resource-manager","msg":"machine class libvirt-provider-dev-t3-small will be ignore","error":"missing source for resource sgx.intel.com/epc: resource isn't supported"}
- Logging should be configurable with the following command line arguments:
    --zap-encoder encoder
    --zap-log-level level
    --zap-time-encoding time-encoding

**Recommendation:**

- We use the zap library (zapr is a wrapper for controller-runtime with the logr interface).
- You can use plain zap if you don't need the logr interface.
- We recommend printing errors at the top of the function (not where the error was created, but where the decision or logic occurs).
    - We strongly recommend avoiding double logging of the same log entries.
- Ideally, all errors should be logged.
- All errors should be trackable.

## Configuration

**Mandatory:**

- Command line arguments must follow [GNU-style](https://www.gnu.org/software/libc/manual/html_node/Argument-Syntax.html) format.
    - Example: --host, -h

**Recommendation:**

- Applications should be configured through files or command line arguments.
- We only support [GNU-style](https://www.gnu.org/software/libc/manual/html_node/Argument-Syntax.html) command line arguments format.
    - We often use the [pflag](https://github.com/spf13/pflag) library in other projects.
    - We use the library [koanf](https://github.com/lukas016/koanf) in some projects, but it is more of a proof of concept than a final solution.
- We prefer configuring applications via command line arguments and configuration files.
- We prefer YAML format for application configuration files.
- We don't recommend configuring applications via environment variables, but it isn't forbidden.

## Folder structure

**Mandatory:**

- Complex projects with many packages have to follow the [project layout](https://github.com/golang-standards/project-layout/tree/master).
- If your packages should not be imported into other projects, they must be stored in a subfolder called `internal`.
- If your packages should be imported into other projects, they must be stored in a subfolder called `pkg`.
- Use a `bin` subfolder for installing additional tools or building binaries.
- The `bin` subfolder must be added to `.gitignore`.
- All e2e files have to be store inside test/e2e folder (follow kubebuilder pattern)

**Recommendation:**

- Simple projects can use a simpler folder structure (e.g., without subfolders).
- If application runs in kubernetes, folder should contain folder `config` with k8s manifest for [kustomize](https://kustomize.io/)
    - You can follow structure from kubebuilder project. Example: <https://github.com/ironcore-dev/libvirt-provider/tree/main/config>
- If application is deployed as helmchart, helmchart has to be store inside charts folder.

## Unit tests

**Recommendation:**

- We recommend using the [testify](https://github.com/stretchr/testify) framework or [ginkgo](https://github.com/onsi/ginkgo) for unit, integration, and e2e tests.
- You can also use the standard Go testing library for unit tests.

## Application

**Mandatory:**

- Applications running in Kubernetes must support graceful shutdown triggered by INT and TERM signals.
- Applications with parallel goroutines and recovery handlers for panic should log the panic error and expose metrics with the count of caught panics.
- Applications must contain version, name, and build date information, which should be printed during startup.
- Use Prometheus metrics for the app.

**Recommendations:**

- Ideally, all applications should implement graceful shutdown.

## Makefile

**Mandatory:**

- The makefile must be possible override makefile variables.
    - If the makefile contains container image builds:
        - You should be able to use either Docker or Podman.
        - You should be able to inject custom command line arguments into the build process.
    - The makefile must not affect other projects or the development environment of another project, either directly or indirectly.
        - Example: If you install a linter with a specific version, you cannot install it into a global PATH.
            It must be installed into a project subfolder (e.g., `bin`) to avoid negative impacts on the developer environment.

**Recommendation:**

- Ideally, the makefile should be usable in CI/CD.
- We strongly recommend using the [Kubebuilder makefile](https://github.com/kubernetes-sigs/kubebuilder/blob/master/pkg/plugins/golang/v4/scaffolds/internal/templates/makefile.go#L76)
- for Go projects as a base Makefile and modifying it for your project.

## Git

**Mandatory:**

- The review process should follow these rules:
    - The author of a thread should close it once it is solved or irrelevant.
        - It isn't applied to some suggestion feature.
    - The maintainer of the repository can close threads from other users if they are long-term open and without updates.
    - Reviews are meant to be discussions.
        - You should ask for clarification if something is unclear.
        - You should **constructively** argue in threads.
        - You shouldn't blindly implement suggestions from threads.
        - If a thread is discussed on a call, the result of the call must be written in the thread, including a list of participants.
        - Not every thread has to be "solved" in PR.
            Some threads can be solved with creating follow up issues. In this case follow up issue link has to be put into thread as comment.

**Recommendation:**

- A fast-forward strategy is preferred for PR/MR (no merge commits).
- You can follow <https://www.conventionalcommits.org/en/v1.0.0/> for better commit conventions.

## Golang

**Mandatory:**

- Don't use shadowing, if it isn't neccessary
    - Reason for shadowing has to commented
- During the build of a Go app, the `-trimpath` argument must be used.

**Recommendation:**

- Do not use the Cobra library if you don't need subcommands for your application.
- You should follow in most cases style from <https://google.github.io/styleguide/go/>
- It isn't recommended creating variables in if blocks, Junior developers have problem with that and it can create shadowing.

## Documentation

**Mandatory:**

- Documentation must be written in markdown.
- We use [mkdocs](https://www.mkdocs.org/) to generate a website from markdown documentation.
- We strongly require the use of 4 spaces for markdown files.
    - 4 spaces are the most universal across different markdown implementations and work well with other markdown features like auto-increment.
- Diagrams created by us should be created with mermaid (simple diagram) or draw.io (complex diagram).
- Draw.io diagrams have to follow these rules:
    - they have to be store with diagram information for simple edition later (not just bitmap image).
    - Image format has to be PNG with diagram information.
    - Image name has to end with suffix `.drawio.png`.
    - Don't use tabs in drawio images.
        - One file one image!!!
- Unordered list should be always created with `-`
- Headers in one document/file has to be unique. Headers are converted into HTML ID which can be used in links,
    and duplicated headers can create trouble in communication because other section can be render between users.

**Recommendation:**

- You can use vscode extension <https://marketplace.visualstudio.com/items?itemName=hediet.vscode-drawio> for direct editing of draw.io diagrams in vsdode editor.
- we recommend use reference-style links and manage links on single place in file.

    ``` markdown
    See more in the [documentation][doku] or the [other place][otherplace].

    [doku]: http://example.com/doku
    [otherplace]: http://example.com/other-place
    ```

- You can use some features from mkdocs for writing documentation.
- We recommend using the auto-increment feature for ordered lists:
    1. First item
    1. Second item
- Try keep raw markdown line length around 80 characters (max 120).
    - It is easier scroll document vertically only.
