# Podman Nix Development Shell Flake

[![NixOS][nixos-badge]][nixos]
[![Build and Test][ci-badge]][ci]
[![Update][update-badge]][update]

This flake should enable you to inject podman as a development environment dependency.

## Usage

### Directly

*Warning: Manual configuration of `/etc/subuid` and `/etc/subgid` per the podman installation instructions is requried and cannot be performed by Nix.*  
*Warning: Manual installation of `newuidmap` is required and cannot be performed by Nix.*  
*Warning: shellHooks are not run via this direct method and therefore `.config/containers/{registries.conf,policy.json}` must be created manually.*  

To fire off podman in a Nix shell quickly, just use this command to run a `hello-world` container
from the Docker Hub:

```bash
nix run github:christianharke/podman-shell-flake -- run hello-world
```

Or to get a shell with podman available in it:
```bash
nix shell github:christianharke/podman-shell-flake
```

### Nix Overlay

*Warning: Manual configuration of `/etc/subuid` and `/etc/subgid` per the podman installation instructions is requried and cannot be performed by Nix.*  
*Warning: Manual installation of `newuidmap` is required and cannot be performed by Nix.*  

For providing the `podman-shell` in a Nix development shell, this flake needs to be added to the
`inputs` and its `overlay` registered in the `nixpkgs` overlay. Afterwards it can just be added to the
`buildInputs` - but don't forget to integrate its `shellHook` as well.

**Example**

```nix
# flake.nix

{
  description = "Podman shell flake demo";

  # include flake
  inputs.podman-shell.url = "github:christianharke/podman-shell-flake";

  outputs = { self, nixpkgs, podman-shell }:
    let
      # System types to support.
      supportedSystems = [ "aarch64-linux" "x86_64-linux" ];

      # Helper function to generate an attrset '{ x86_64-linux = f "x86_64-linux"; ... }'.
      forAllSystems = nixpkgs.lib.genAttrs supportedSystems;

      # Nixpkgs instantiated for supported system types.
      nixpkgsFor = forAllSystems (system: import nixpkgs {
        inherit system;
        config = { allowUnfree = true; };
        overlays = [        
          # include default overlay
          podman-shell.overlays.default
        ];
      });
    in
    {
      devShells = forAllSystems (system:
        let
          pkgs = nixpkgsFor.${system};
        in
        {
          default = pkgs.mkShell {
            name = "my-dev-shell";

            buildInputs = [
              # add packages
              pkgs.podman-shell
              pkgs.podman-shell.dockerCompat # optional - for use as a `docker` drop-in replacement
            ];

            # include shellHook from podman-shell
            inherit (pkgs.podman-shell) shellHook;
          };
        });
    };
}
```

## References

Highly inspired by [adisbladis' podman-shell.nix](https://gist.github.com/adisbladis/187204cb772800489ee3dac4acdd9947).

[nixos]: https://nixos.org/
[nixos-badge]: https://img.shields.io/badge/NixOS-blue.svg?logo=NixOS&logoColor=white
[ci]: https://github.com/christianharke/podman-shell-flake/actions/workflows/ci.yml
[ci-badge]: https://github.com/christianharke/podman-shell-flake/actions/workflows/ci.yml/badge.svg
[update]: https://github.com/christianharke/podman-shell-flake/actions/workflows/update.yml
[update-badge]: https://github.com/christianharke/podman-shell-flake/actions/workflows/update.yml/badge.svg

