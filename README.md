# Podman Nix Development Shell Flake

[![NixOS][nixos-badge]][nixos]
[![Build and Test][ci-badge]][ci]
[![Update][update-badge]][update]

This flake should enable you to inject podman as a development environment dependency.

## Usage

### Directly

*WARNING: This direct usage still requires manual configuation of the `/etc/subuid` and `/etc/subgid` just like a regular podman installation, manual installation of `newuidmap`, and manual creation of the `.config/containers/{registries.conf,policy.json}` files.*

To fire off podman in a Nix shell quickly, just use this command to run a `hello-world` container
from the Docker Hub:

```bash
nix run github:christianharke/podman-shell-flake -- run hello-world
```

Or to create an ad-hoc subshell with podman available:
```
nix shell github:christianharke/podman-shell-flake
```

### Nix Overlay

*WARNING: This usage requires manual configuation of the `/etc/subuid` and `/etc/subgid` just like a regular podman installation, and manual installation of `newuidmap`.*

For providing the `podman-shell` in a Nix development shell, this flake needs to be added to the
`inputs` and its `overlay`(`overlays.default`) registered in the `nixpkgs` overlay. Afterwards it can just be added to the
`buildInputs` - but don't forget to integrate its `shellHook` as well.

**Example**

```nix
# flake.nix

{
  description = "Podman shell flake demo";

  # include the podman-shell flake
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
        overlays = [
          # include the podman-shell overlay
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
              # include the podman-shell package
              pkgs.podman-shell
            ];

            # include specialty shellHook from podman-shell
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

