# Understanding Nix Flakes

Nix Flakes are a highly praised feature in the Nix ecosystem, but finding straightforward information about them can be challenging. This guide will clarify what Flakes are, why you need them, and how to use them.

## Introduction to Nix Flakes

### What Are Nix Flakes?

In simple terms, Nix Flakes are a system for managing your Nix ecosystem, including configurations, development environments, derivations, and more. Currently, Nix Flakes and the `nix` command are experimental features, which means you need to explicitly enable them.

### Enabling Nix Flakes

On NixOS, you can enable Flakes by setting the experimental features option:

```nix
# Add this to your /etc/nixos/configuration.nix
nix.extraOptions = ''
  experimental-features = nix-command flakes
'';
```

On other distributions and macOS, use the following commands to create the package manager configuration directory (if it doesn't exist) and add the required settings:

```sh
mkdir -p ~/.config/nix
echo "experimental-features = nix-command flakes" >> ~/.config/nix/nix.conf
```

Don't forget to rebuild your NixOS system afterwards with:

```sh
sudo nixos-rebuild switch
```

Or restart the Nix daemon on other distributions:

```sh
sudo systemctl restart nix-daemon
```

## Using Nix Shell

### Regular Nix Shell

Using the `nix-shell` command, you can create temporary shell environments with packages not present on your system. For example:

```sh
nix-shell -p python
```

You can also create a `shell.nix` file for more complex shells:

```nix
{ pkgs ? import <nixpkgs> {} }:
pkgs.mkShell {
  buildInputs = [ pkgs.neovim ];
  shellHook = ''
    echo "Welcome to your development environment!"
  '';
}
```

Running `nix-shell` in the directory with this file will provide you with a shell environment including Neovim and a custom shell hook.

### Problems with Regular Nix Shell

One major issue with this approach is that it depends on the system's current channel version. Different channel versions can lead to different environments, which can break projects or desktop environments when updated.

## Introducing Nix Flakes

### Initializing a Flake

To start using Flakes, initialize a Flake in an empty directory:

```sh
nix flake init
```

This command creates a `flake.nix` file. Here's an example structure of the file:

```nix
{
  description = "A simple flake";
  outputs = { self, nixpkgs }: {
    packages.x86_64-linux.default = nixpkgs.legacyPackages.x86_64-linux.hello;
  };
}
```

### Understanding the `flake.nix` File

- **description**: A text field for describing the flake.
- **outputs**: A set of outputs, including packages and other configurations.

Example of a modified `flake.nix` for a development shell:

```nix
{
  description = "My development shell";
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
  };
  outputs = { self, nixpkgs }: {
    devShells.x86_64-linux.default = nixpkgs.legacyPackages.x86_64-linux.mkShell {
      buildInputs = [ nixpkgs.neovim ];
      shellHook = ''
        echo "Welcome to your development environment!";
      '';
    };
  };
}
```

### Using the Flake

To use the Flake, run:

```sh
nix develop
```

This command generates a `flake.lock` file and provides the specified development environment. To update the Flake's inputs, use:

```sh
nix flake update
```

## Benefits of Nix Flakes

- **Consistency**: Flakes provide a consistent environment by locking versions of dependencies.
- **Ease of Use**: While the syntax may seem complex initially, it provides a robust and reproducible way to manage environments.
- **Flexibility**: Flakes can manage home configurations, Docker files, packages, and more.

## Best Practices

- **Use Git**: Create a Git repository for each of your Flakes to avoid losing important configurations and dependencies.
- **Commit Regularly**: Flakes require all files in the directory to be committed or staged.

## Nix Flakes Knowledge Base

### Overview
A Nix flake is a modern way to manage and build projects with Nix. This example demonstrates a simple C++ program with dependencies on Boost and SDL2 libraries, using CMake for the build system, and Nix flakes to manage dependencies and builds.

### C++ Demo Program
- **Purpose**: Print version numbers of Boost and SDL2 libraries.
- **Dependencies**: Boost headers library, SDL2 library.
- **Build System**: CMake.

### CMake Integration
- **CMake** finds dependencies (Boost, SDL2) in the system.
- **CMake** configures and builds the executable.

### Nix Flake Components
- **flake.nix**: Main configuration file for the flake.
- **flake.lock**: Locks dependencies to specific versions.

### Commands and Outputs
- **nix flake show**: Visualizes the contents of the Nix flake.
- **nix run**: Builds and runs the default package.
- **nix build**: Builds the application and creates a symlink in the directory.
- **nix develop**: Opens a Nix shell with all dependencies for development.

### Nix Flake Structure
- **Inputs**: Specifies dependencies like nixpkgs and flake-utils.
- **Outputs**: Defines packages and other outputs.
- **Packages**: Defines how to build the default package using `stdenv.mkDerivation`.

### Sample `flake.nix`
```nix
{
  description = "OpenTechLab C++ Application Example";

  inputs = {
    nixpkgs.url = "nixpkgs/nixos-22.11";
    nixpkgsUnstable.url = "nixpkgs/nixos-unstable";
    flake-utils.url = "github:numtide/flake-utils";
  };

  outputs = { self, nixpkgs, nixpkgsUnstable, flake-utils }:
    flake-utils.lib.eachDefaultSystem (system: let
        pkgs = import nixpkgs { inherit system; };
        pkgsUnstable = import nixpkgsUnstable { inherit system; };
      in {
        packages = rec {
          default = pkgs.stdenv.mkDerivation {
            name = "otl_nix_test_app";
            src = ./.;

            nativeBuildInputs = with pkgs; [
              cmake
            ];

            buildInputs = with pkgs; [
              boost
              SDL2
            ];
          };

          dockerImage = pkgsUnstable.dockerTools.buildNixShellImage {
            tag = "latest";
            drv = default.overrideAttrs (old: { src = null; });
          };
        };
      });
}
```

### Explanation of `flake.nix`
- **description**: Describes the project.
- **inputs**: 
  - `nixpkgs`: The stable Nix packages collection.
  - `nixpkgsUnstable`: The unstable Nix packages collection.
  - `flake-utils`: Utilities for managing flakes.
- **outputs**: 
  - `packages`:
    - `default`: The main package for the application.
    - `dockerImage`: A Docker image for the application using the unstable Nix packages.

### Sample `main.cpp`
```cpp
#include <iostream>
#include <cstdlib>

#include <boost/version.hpp>
#include <SDL2/SDL.h>

int main(int argc, const char *const *const argv) {
    std::cout << "OpenTechLab C/C++ Example Application\n";

    std::cout << "Boost v" <<
        (BOOST_VERSION / 100000) << '.' <<
        ((BOOST_VERSION / 100) % 1000) << '.' <<
        (BOOST_VERSION % 100) <<
        '\n';

    {
        SDL_version version;
        SDL_GetVersion(&version);
        std::cout <<
            "SDL v" <<
            static_cast<int>(version.major) << '.' <<
            static_cast<int>(version.minor) << '.' <<
            static_cast<int>(version.patch) <<
            '\n';
    }

    std::cout << std::endl;

    return EXIT_SUCCESS;
}
```

### Key Features and Improvements
1. **Avoid Repetition**: Use `with pkgs;` to avoid repeating `pkgs.`.
2. **Local Variables**: Defined with `let ... in` to simplify expressions.
3. **Portability**: Use `flake-utils.lib.eachDefaultSystem` to support multiple platforms.

### Summary
- **Nix flakes** offer a powerful way to manage dependencies and builds.
- **CMake** integrates smoothly with Nix flakes for C++ projects.
- The **flake.nix** file is versatile, supporting multiple platforms and improving code maintainability through features like `let ... in` and `flake-utils`.

## Advanced Usage

### Using Inputs

Flakes can declare dependencies on other flakes using inputs. Here is an example of a `flake.nix` file with inputs:

```nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    my-flake.url = "github:my-org/my-flake";
  };

  outputs = { self, nixpkgs, my-flake }:
    let
      system = "x86_64-linux";
    in {
      packages.${system}.my-package = nixpkgs.lib.mkDerivation {
        pname = "my-package";
        version = "0.1.0";
        src = self;
        buildInputs = [ my-flake.packages.${system}.some-dependency ];
      };
    };
}
```

### Pinning Dependencies

To ensure reproducibility, you can pin the versions of your dependencies by using a `flake.lock` file. This file is generated and updated by the `nix flake update` command.

### Customizing Outputs

Flakes can define multiple outputs for different purposes. For instance, you can have separate outputs for packages, apps, and devShells:

```nix
{
  outputs = { self, nixpkgs }:
    let
      system = "x86_64-linux";
    in {
      packages.${system}.default = nixpkgs.lib.mkDerivation {
        pname = "my-package";
        version = "0.1.0";
        src = self;
      };
      
      apps.${system}.default = {
        type = "app";
        program = "${self.packages.${system}.default}/bin/my-app";
      };
      
      devShells.${system}.default = nixpkgs.mkShell {
        buildInputs = [ self.packages.${system}.default ];
      };
    };
}
```
