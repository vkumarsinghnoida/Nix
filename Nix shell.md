
## Overview
Nix allows users to create temporary shell environments where programs can be used without permanent installation. These environments work across Linux distributions, WSL, and macOS.

### Creating a Shell Environment
- **Installation**: After installing Nix, you can create environments with desired programs.
- **Example Programs**: Programs like `cowsay` and `lolcat` can be run temporarily using:

  ```sh
  nix-shell -p cowsay lolcat
  ```

- **Usage**: Inside the Nix shell, you can run these programs. Exiting the shell removes access to them.

  ```sh
  cowsay Hello, Nix! | lolcat
  ```

  ```sh
  exit
  ```

#### Running Programs Once
- **Direct Execution**: Use the following to run a program directly without entering a shell:

  ```sh
  nix-shell -p cowsay --run "cowsay Nix"
  ```

  If the command consists only of the program name, no quotes are needed:

  ```sh
  nix-shell -p hello --run hello
  ```

#### Searching for Packages
- **Finding Packages**: Use [search.nixos.org](https://search.nixos.org) to find packages by name.
- **Examples**: For programs like `git`, `nvim`, and `npm`, you can find and specify their package names.

#### Running Multiple Programs
- **Combined Environments**: Start a shell with multiple packages using:

  ```sh
  nix-shell -p git neovim nodejs
  ```

#### Checking Package Versions
- **Verification**: Check the versions of programs to ensure the correct ones are used by Nix.

  ```sh
  which git
  git --version
  ```

  ```sh
  which nvim
  nvim --version | head -1
  ```

  ```sh
  which npm
  npm --version
  ```

#### Nested Shell Sessions
- **Additional Programs**: Add programs temporarily to an existing Nix shell using:

  ```sh
  nix-shell -p python3
  ```

  ```sh
  python --version
  ```

  Exit the shell as usual to return to the previous environment:

  ```sh
  exit
  ```

#### Reproducibility
- **Consistency**: Ensure identical environments by specifying exact package versions with the `-I` option and using specific Git revisions. Here's a detailed breakdown of the command:

  ```sh
  nix-shell -p git --run "git --version" --pure -I nixpkgs=https://github.com/NixOS/nixpkgs/archive/2a601aafdc5605a5133a2ca506a34a3a73377247.tar.gz
  ```

  - `nix-shell -p git`: Starts a Nix shell with the `git` package.
  - `--run "git --version"`: Executes the `git --version` command within the Nix shell and exits after running it.
  - `--pure`: Discards most of the environment variables set on your system when running the shell, ensuring that only the `git` provided by Nix is available.
  - `-I nixpkgs=https://github.com/NixOS/nixpkgs/archive/2a601aafdc5605a5133a2ca506a34a3a73377247.tar.gz`: Specifies a specific Git revision of `nixpkgs` to use, ensuring that the exact version of `git` (and other dependencies) from this snapshot is used, guaranteeing reproducibility.

#### References and Next Steps
- **Further Reading**: Explore the Nix manual and additional resources to learn more about reproducible scripts, Nix language, and declarative shell environments.
- **Cleanup**: Free up disk space with:

  ```sh
  nix-collect-garbage
  ```

### Summary of Reproducible Interpreted Scripts with Nix

#### Overview
This tutorial covers how to use Nix to create and run reproducible interpreted scripts (shebang scripts).

#### Requirements
- A working Nix installation.
- Familiarity with Bash.

#### A Trivial Script with Non-Trivial Dependencies
Consider the following Bash script that fetches XML content from a URL, converts it to JSON, and formats it using `jq`:

```bash
#!/bin/bash

curl https://github.com/NixOS/nixpkgs/releases.atom | xml2json | jq .
```

This script requires `curl`, `xml2json`, and `jq`, as well as the Bash interpreter. If these dependencies are not present, the script will fail.

With Nix, you can declare all dependencies explicitly to ensure the script runs on any machine with Nix installed.

#### The Script
A shebang determines which program to use for running an interpreted script. The tutorial uses `#!/usr/bin/env nix-shell` to make the script reproducible.

- **Shebang with Nix**: The `env` program finds and runs executables in the directories listed in `$PATH`. Using `nix-shell` as the shebang interpreter ensures dependencies are managed by Nix.

- **Parameters**:
  - `-i` specifies the interpreter for the script.
  - `--pure` excludes most environment variables, ensuring a clean environment.
  - `-p` lists the required packages.
  - `-I` sets the search path for packages explicitly.

#### Creating the Script
Create a file named `nixpkgs-releases.sh` with the following content:

```sh
#!/usr/bin/env nix-shell
#! nix-shell -i bash --pure
#! nix-shell -p bash cacert curl jq python3Packages.xmljson
#! nix-shell -I nixpkgs=https://github.com/NixOS/nixpkgs/archive/2a601aafdc5605a5133a2ca506a34a3a73377247.tar.gz

curl https://github.com/NixOS/nixpkgs/releases.atom | xml2json | jq .
```

- **First Line**: Standard shebang.
- **Additional Shebang Lines**: Nix-specific constructs.
  - `-i bash`: Specifies Bash as the interpreter.
  - `--pure`: Ensures no external environment variables interfere.
  - `-p bash cacert curl jq python3Packages.xmljson`: Lists the required packages.
  - `-I`: Ensures the exact versions of packages by referring to a specific Git commit of the Nixpkgs repository.

#### Making the Script Executable
Make the script executable:

```sh
chmod +x nixpkgs-releases.sh
```

Run the script:

```sh
./nixpkgs-releases.sh
```

#### Next Steps
- **Nix Language Basics**: Learn the Nix language for declaring packages and configurations.
- **Declarative Shell Environments**: Use `shell.nix` for reproducible shell environments.
- **Garbage Collection**: Free up storage used by programs with:

  ```sh
  nix-collect-garbage
  ```

### Summary of Declarative Shell Environments with `shell.nix`

#### Overview
Declarative shell environments in Nix allow you to:
- Automatically run bash commands during environment activation.
- Automatically set environment variables.
- Put the environment definition under version control for reproducibility on other machines.


### Entering a Temporary Shell
You can quickly create a temporary environment with `cowsay` and `lolcat` using:

```sh
nix-shell -p cowsay lolcat
```

However, this has drawbacks such as repetitiveness and limited customization.

#### A Basic `shell.nix` File
Create a `shell.nix` file for a more permanent solution:

```nix
let
  nixpkgs = fetchTarball "https://github.com/NixOS/nixpkgs/tarball/nixos-23.11";
  pkgs = import nixpkgs { config = {}; overlays = []; };
in

pkgs.mkShellNoCC {
  packages = with pkgs; [
    cowsay
    lolcat
  ];
}
```

Enter the environment by running `nix-shell` in the directory containing `shell.nix`:

```sh
nix-shell
cowsay hello | lolcat
```

#### Environment Variables
To set environment variables automatically, modify `shell.nix`:

```nix
let
  nixpkgs = fetchTarball "https://github.com/NixOS/nixpkgs/tarball/nixos-23.11";
  pkgs = import nixpkgs { config = {}; overlays = []; };
in

pkgs.mkShellNoCC {
  packages = with pkgs; [
    cowsay
    lolcat
  ];

  GREETING = "Hello, Nix!";
}
```

Exit and re-enter the shell, then check the variable:

```sh
echo $GREETING
```

#### Startup Commands
To run commands upon entering the shell, use `shellHook`:

```nix
let
  nixpkgs = fetchTarball "https://github.com/NixOS/nixpkgs/tarball/nixos-23.11";
  pkgs = import nixpkgs { config = {}; overlays = []; };
in

pkgs.mkShellNoCC {
  packages = with pkgs; [
    cowsay
    lolcat
  ];

  GREETING = "Hello, Nix!";

  shellHook = ''
    echo $GREETING | cowsay | lolcat
  '';
}
```

Exit and re-enter the shell to see the greeting.

#### References
- [mkShell documentation](https://nixos.org/manual/nixpkgs/stable/#chap-shells)
- [Nixpkgs shell functions and utilities documentation](https://nixos.org/manual/nixpkgs/stable/#sec-pkgs-mkShell)
- [nix-shell documentation](https://nixos.org/manual/nix/stable/#sec-nix-shell)

### Summary of Pinning Nixpkgs for Reproducibility

#### Overview
Pinning Nixpkgs ensures that Nix expressions are fully reproducible by referencing exact versions of Nix packages.

#### Non-Reproducible Nix Expressions
Using `<nixpkgs>` in Nix expressions is common and convenient but does not guarantee reproducibility. For example:

```nix
{ pkgs ? import <nixpkgs> {} }:

...
```

This imports Nix packages without specifying a fixed version, leading to potential variability.

#### Pinning Packages with URLs
To create reproducible Nix expressions, you can pin an exact version of Nixpkgs by fetching a specific tarball via its Git commit hash. For example:

```nix
{ pkgs ? import (fetchTarball "https://github.com/NixOS/nixpkgs/archive/06278c77b5d162e62df170fec307e83f1812d94b.tar.gz") {} }:

...
```

This method fetches the specified version of Nixpkgs, ensuring that the same package versions are used each time the expression is evaluated.

#### Choosing a Commit
You can choose a specific commit by referring to [status.nixos.org](https://status.nixos.org), which lists all releases and the latest commits that have passed tests. It is recommended to follow:
- The latest stable NixOS release, e.g., `nixos-21.05`.
- The latest unstable release, e.g., `nixos-unstable`.

#### Next Steps
For more detailed examples and different methods to pin Nixpkgs, refer to the documentation on [Pinning Nixpkgs](https://nixos.org/manual/nixpkgs/stable/#sec-pinning-nixpkgs).

