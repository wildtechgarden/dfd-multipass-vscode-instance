# DFD-Zen Multipass workspace

## Overview

Daniel F. Dickinson's VSCode workspace for configuring a 'Remote - SSH'
environment using [Multipass](https://multipass.run/) for hypervisor
independent reproducible Linux instances; not quite as ephemeral as
devcontainers in VSCode, but intended not gather cruft, and to be updated
by recreating the instance.

## Status

TBD

## Repository

<https://github.com/wildtechgarden/dfd-multipass-vscode-instance>

## Why use Multipass

Daniel uses Multipass rather than VSCode devcontainers because he still uses
some Windows machine that do not support Hyper-V, and therefore cannot run
Docker Desktop, on which devcontainers under Windows are based. In addition
Daniel is very familiar and comfortable with virtualized Linux environments,
so he finds using Multipass (which primarily launches
[Ubuntu](https://ubuntu.com) instances) quite easy.

The purpose of using devcontainers or Multipass is to avoid one's desktop
interfering with one's development environment(s) and vice versa.

In addition, if done right, launching a pristine development environment
and adding one's code is easy and avoids previous sessions causing issues
with the current instance.

## Documentation

Daniel has documented the how he [(mostly) automates the creation / recreation
of Multipass instances including provisioning them, and adding project
code](multipass-instance-creation/README.md).

## Colophon

* [Copyright and licensing](LICENSE) (unless otherwise noted)
* [Inspirations, information, and source material](ACKNOWLEDGEMENTS.md)
* [Notes](README-NOTES.md)
