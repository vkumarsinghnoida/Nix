To create a flake with a fixed Git commit from nixpkgs, generate the `flake.lock` file, and use it with `nix profile install`, follow these steps:

### Step 1: Create a Flake

1. **Create a Directory for Your Flake**:

   ```bash
   mkdir my-flake
   cd my-flake
   ```

2. **Create a `flake.nix` File**:

   ```nix
   {
     description = "A simple flake with fixed nixpkgs commit";

     inputs.nixpkgs.url = "github:NixOS/nixpkgs/commit/<commit-hash>";

     outputs = { self, nixpkgs }:
       let
         pkgs = import nixpkgs { system = "x86_64-linux"; };
       in {
         packages.x86_64-linux.hello = pkgs.hello;
       };
   }
   ```

   Replace `<commit-hash>` with the desired commit hash from the nixpkgs repository.

### Step 2: Generate the `flake.lock` File

1. **Run the Following Command**:

   ```bash
   nix flake update
   ```

   This command will create the `flake.lock` file, locking dependencies to specific versions.

### Step 3: Use `nix profile install`

1. **Install a Package from the Flake**:

   ```bash
   nix profile install .#hello
   ```

   This command installs the `hello` package defined in your flake from the specified nixpkgs commit.

### Summary

- **Create** a `flake.nix` specifying the fixed commit.
- **Generate** `flake.lock` using `nix flake update`.
- **Install** packages using `nix profile install .#package-name`.