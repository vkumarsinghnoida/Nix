Certainly! Let's adjust the example to import `nixpkgs` from a specific git commit for reproducibility:

**example.nix**

```nix
{ pkgs ? import (fetchTarball {
    url = "https://github.com/NixOS/nixpkgs/archive/<commit>.tar.gz";
    sha256 = "<sha256-hash>";
  }) {} }:

pkgs.stdenv.mkDerivation {
  pname = "hello-world";
  version = "1.0.0";

  src = pkgs.fetchurl {
    url = "https://example.com/hello-world-1.0.0.tar.gz";
    sha256 = "sha256-0000000000000000000000000000000000000000000000000000";
  };

  buildInputs = [ pkgs.gcc ];

  buildPhase = ''
    gcc -o hello hello.c
  '';

  installPhase = ''
    mkdir -p $out/bin
    mv hello $out/bin
  '';
}
```

### Steps to Use This Adjusted Example:

1. **Update `commit` and `sha256`**:
   - Replace `<commit>` with the specific commit hash of `nixpkgs` you want to use.
   - Replace `<sha256-hash>` with the SHA256 hash of the tarball fetched from GitHub. You can use a dummy value initially and correct it after the first build attempt.

2. **Build the Package**:
   - Run `nix-build example.nix`.
   - Nix will verify the hash of the `nixpkgs` tarball and the `src` archive.

3. **Reproducibility**:
   - This setup ensures that the package is built against a specific version of `nixpkgs`, making the build reproducible across different systems.

4. **Using on Another PC**:
   - Copy `example.nix` to another PC with Nix installed.
   - Run `nix-build example.nix` there to build the package using the same `nixpkgs` version.

This approach maintains consistency by fixing the `nixpkgs` version, ensuring the package behaves predictably across different environments. Adjust the `<commit>` and `<sha256-hash>` placeholders with actual values appropriate for your use case.