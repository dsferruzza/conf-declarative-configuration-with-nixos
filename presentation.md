% How immutable infrastructure and declarative configuration removed the pain of managing my servers
% David Sferruzza
% 21/10/2016


# About me

- [\@d_sferruzza](https://twitter.com/d\_sferruzza)
- [github.com/dsferruzza](https://github.com/dsferruzza)
- Head of Research and Development at [Startup Palace](https://www.startup-palace.com)
- PhD student in software engineering at *University of Nantes*

<figure class="stretch"><img src="img/sp.gif" alt=""></figure>


# Startup Palace

We **iteratively** design and develop web applications. <small>(among other things)</small>

Frequent reviews (by customers and team members) are an important part of the job.

> We need staging/preprod environments

<figure class="stretch"><img src="img/crash.gif" alt=""></figure>


# Staging environments

They should be:

- quick to create
- quick to configure
- quick to remove
- cheap
- compatible with automatic deployment

<figure class="stretch"><img src="img/cups.gif" alt=""></figure>

<div class="notes">
This shoud be a **convenience**.
</div>


# About automatic deployment

We use [GitLab](https://about.gitlab.com/)

<figure class="stretch"><img src="img/gitlab.svg" alt=""></figure>

> GitLab unifies chat, issues, code review,<br>CI and CD into a single UI


# About automatic deployment

GitLab CI is very flexible:

```yaml
my_deploy:
  stage: deploy
  image: org/my_docker_image
  script:
    - echo "I can do anything here"
    - rsync ./my_projet $USER@$HOST:/var/www/
  only:
    - master
  tags:
    - docker
```


# Naive solution

We know [Debian](https://www.debian.org/). Let's use Debian!

And let's install **and** *configure*:

- Apache
- PHP
- PostgreSQL
- SSH

*by hand*!

<figure class="stretch"><img src="img/madness.gif" alt=""></figure>


# Naive solution

It works but:

- it needs a lot of manual setup<br>&rarr; admins need to be watchful and rigorous
- environments can diverge<br>&rarr; it can quickly become a mess
- security/system updates can break stuff<br>&rarr; you need to fix environments one by one
- it's soooo **boring** to use

> Boring means I will make mistakes


# Naive solution

We did it for a while.

*1 star. Would not recommend.*

<figure class="stretch"><img src="img/rain.gif" alt=""></figure>

So, can we do better?


# Dockerfile

At that time, people were talking a lot about Docker. So I took a look.

The concept of the **Dockerfile** is interesting:

- you start from an existing image
- you *declare* how to change it
- you get a new image

```dockerfile
FROM debian:jessie
RUN apt-get update \
 && apt-get install --no-install-recommends -y \
    openjdk-7-jdk
```


# Dockerfile

This is fine. But:

- you declare **how** to build an image<br>not **what** is the image you want to build
- this is too linear (layers of cache)
- it's hard to compose
- ...

<figure class="stretch"><img src="img/this-is-fine.jpg" alt=""></figure>


# Declarative vs. imperative

> **Imperative programming**: use statements that change a program's state
>
> **Declarative programming**: express the logic of a computation without describing its control flow

- imperative: shopping instructions
- declarative: shopping list


# The NixOS project

> <figure class="stretch"><img src="img/nixos.svg" alt=""></figure>
> <https://nixos.org/>

- Nix
- Nix Expression Language
- Nixpkgs
- NixOS (Linux distribution)
- ...

<div class="notes">
Created by *Eelco Dolstra* for its PhD
</div>


# Nix

> The Purely Functional Package Manager
>
> <https://nixos.org/nix/>

Packages:

- are treated like values
- are built by functions that don't have side-effects
- never change after they have been built

Builds are **reproducible**.


# Nix store

The place where packages are stored. Usually in `/nix/store/`

Example:<br><small>`/nix/store/f4gxsj6pn4ygqadwyk2m6xg1ywhfwxg1-openssl-1.0.2h/`</small>

A package's directory name contains:

- its name
- its version
- a unique identifier that captures all its dependencies

<div class="notes">
unique identifier = a cryptographic hash of the package’s build dependency graph
</div>


# User environments

<figure class="stretch"><img src="img/user-envs.png" alt=""></figure>


# The `which` package

<div class="smallcode">
```nix
{ stdenv, fetchurl }:

stdenv.mkDerivation rec {
  name = "which-2.21";

  src = fetchurl {
    url = "mirror://gnu/which/${name}.tar.gz";
    sha256 = "1bgafvy3ypbhhfznwjv1lxmd6mci3x1byilnnkc7gcr486wlb8pl";
  };

  meta = with stdenv.lib; {
    homepage = http://ftp.gnu.org/gnu/which/;
    platforms = platforms.all;
    license = licenses.gpl3;
  };
}
```
```text
result/
├── bin
│   └── which
└── share
    ├── info
    │   └── which.info
    └── man
        └── man1
            └── which.1.gz
```
</div>


# The `which` package

<div class="smallcode">
```nix
{ stdenv, fetchurl }:

stdenv.mkDerivation rec {
  name = "which-2.21";

  src = fetchurl {
    url = "mirror://gnu/which/${name}.tar.gz";
    sha256 = "1bgafvy3ypbhhfznwjv1lxmd6mci3x1byilnnkc7gcr486wlb8pl";
  };

  meta = with stdenv.lib; {
    homepage = http://ftp.gnu.org/gnu/which/;
    platforms = platforms.all;
    license = licenses.gpl3;
  };
}
```
```text
$ ldd result/bin/which
linux-gate.so.1 (0xb778e000)
libc.so.6 => /nix/store/6nqfrgw8w5xmm0zarp8f51rvs9lvgq73-glibc-2.23/lib/libc.so.6 (0xb75ea000)
/nix/store/6nqfrgw8w5xmm0zarp8f51rvs9lvgq73-glibc-2.23/lib/ld-linux.so.2 (0xb7790000)
```
</div>

<div class="notes">
ldd - print shared library dependencies
</div>


# Nix

Nice features:

- multiple versions
- complete (and explicit) dependencies
- multi-user
- atomic upgrades and rollbacks
- transparent source/binary deployment

<figure class="stretch"><img src="img/nice.gif" alt=""></figure>

<div class="notes">
build & runtime dependencies
</div>


# Nix Expression Language

> It is a pure, lazy, functional language.
>
> It is **not** a general purpose language.

Types of expressions:

- strings, integers, paths, booleans, null
- functions/lambdas
- lists
- sets
- derivations (package build actions)


# Nix Expression Language

```nix
# This is a list of strings
[ "a" "b" ''c'' "var=${var}" ]

# This is a lambda
f = x: x * x

# This is a set
{ a = 5; b = f 2; }

# This is a recursive set
rec { a = b + 1; b = 5; }
```


# Nixpkgs

> A collection of packages for the Nix package manager.
>
> <https://github.com/NixOS/nixpkgs>

- has nearly 6,500 packages
- uses a permissive MIT/X11 license
- includes a [standard library](https://github.com/NixOS/nixpkgs/tree/master/lib) for Nix Expression Language
- supports GNU/Linux (`i686-linux` and `x86_64-linux`) and Mac OS X (`x86_64-darwin`)


# The `which` package

<div class="smallcode">
```nix
{ stdenv, fetchurl }:

stdenv.mkDerivation rec {
  name = "which-2.21";

  src = fetchurl {
    url = "mirror://gnu/which/${name}.tar.gz";
    sha256 = "1bgafvy3ypbhhfznwjv1lxmd6mci3x1byilnnkc7gcr486wlb8pl";
  };

  meta = with stdenv.lib; {
    homepage = http://ftp.gnu.org/gnu/which/;
    platforms = platforms.all;
    license = licenses.gpl3;
  };
}
```


# NixOS

The Purely Functional Linux Distribution

> Nix + Nixpkgs + systemd + Linux kernel

- declarative system configuration model
- reliable upgrades
- atomic upgrades
- rollbacks
- reproducible system configurations
- safe to test changes
- Nix goodness (multi-user package management, source-based model with binaries, consistency, ...)

<div class="notes">
reliable = installing & upgrading is like from scratch
</div>


# NixOS

<div class="smallcode">
```nix
{ config, lib, pkgs, ... }:
{
  boot.loader.grub.device = "/dev/sda";
  fileSystems."/".device = "/dev/sda1";

  networking.firewall = {
    enable = true;
    allowedTCPPorts = [ 80 ];
  };

  environment.systemPackages = with pkgs; [
    wget
    git
  ];

  services = {
    sshd.enable = true;
    munin-node.enable = true;
    munin-cron = {
      enable = true;
      hosts = ''
        [${config.networking.hostName}]
        address localhost
      '';
    };
  };
}
```
</div>


# NixOS

Realise the configuration:

```text
$ nixos-rebuild switch
```

Rollback to the previous configuration:

```text
$ nixos-rebuild switch --rollback
```

Only build configuration:

```text
$ nixos-rebuild build
```

Test configuration:

```text
$ nixos-rebuild test
$ nixos-rebuild build-vm
```


# NixOS

Build your own system config using:

- the `/etc/nixos/configuration.nix` file
- a functional programming language
- a module system
- packages and generic functions from Nixpkgs

<figure class="stretch"><img src="img/lego.gif" alt=""></figure>


# Options

> A NixOS module can:
>
> - declare options
> - define options depending on other options' values

Built-in options: <https://nixos.org/nixos/options.html>

**Example**: I want a boolean option `myCustomBashAliases` that defines some Bash aliases when enabled.


# Options

In `/etc/nixos/myModule.nix`:

<div class="smallcode">
```nix
{ config, lib, pkgs, ... }:

{
  options = {
    myCustomBashAliases = lib.mkOption {
      default = false;
      description = "Enable my awesome Bash aliases.";
      types = lib.types.bool;
    };
  };

  config = lib.mkIf config.myCustomBashAliases {
    programs.bash.shellAliases = {
      ls = "ls --color=auto";
      ll = "ls --color=auto -lh";
      l = "ls --color=auto -lha";
    };
  };
}
```
</div>


# Options usage

In `/etc/nixos/configuration.nix`:

<div class="smallcode">
```nix
{ config, pkgs, ... }:

{
  imports =
    [
      # ...
      ./myModule.nix
    ];

  myCustomBashAliases = true;

  # ...
}
```
</div>

This allows to build very powerful abstractions!


# Back to the problem

I want to be able to create staging environments:

- quickly
- easily
- in a reliable way

&rarr; Let's write some NixOS modules!

<figure class="stretch"><img src="img/lets-do-this.gif" alt=""></figure>


# Knights Of Nix

An abstraction over NixOS to manage staging environments for static and PHP web apps.

<figure class="stretch"><img src="img/ni.gif" alt=""></figure>

Disclaimers:

- This is a **Proof of Concept**
- This is **not** a *Platform as a Service* (PaaS)


# Knights Of Nix

High-level definition of staging environments:

```nix
[
  # Here is a fake app
  rec {
    appType = "php";
    phpVersion = "70";
    name = "test";
    hostName = "${name}.mydomain.com";
    userSha512Password = "...";
    pgClearPassword = "...";
  }
]
```


# Knights Of Nix

Nice features:

- multi-domain support (with redirections)
- HTTPS support (automatic certification generation with [Let's Encrypt](https://letsencrypt.org/))
- per-app user, with SSH access (I can auto-deploy from GitLab \\o/)
- per-app PHP version
- per-app Cron jobs

<figure class="stretch"><img src="img/combo.gif" alt=""></figure>


# Bonus: nix-shell

Drop a Bash shell configured with the same environment that would be used to perform the build itself.

<div class="smallcode">
```text
$ nix-shell '<nixpkgs>' -A nodejs-5_x

these paths will be fetched (21.40 MiB download, 21.41 MiB unpacked):
  /nix/store/mbv89fkb9av98alqfaay8c4n3czk82q9-node-v5.9.0.tar.gz
fetching path ‘/nix/store/mbv89fkb9av98alqfaay8c4n3czk82q9-node-v5.9.0.tar.gz’...

*** Downloading ‘https://cache.nixos.org/nar/0qzzksfad1g8kk3rfpdiyhdp9h5737nn70zywakarc2lf3qhg0g1.nar.xz’ (signed by ‘cache.nixos.org-1’) to ‘/nix/store/mbv89fkb9av98alqfaay8c4n3czk82q9-node-v5.9.0.tar.gz’...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 21.4M  100 21.4M    0     0  1173k      0  0:00:18  0:00:18 --:--:-- 1374k


[nix-shell:~]$
```
</div>


# Bonus: package customization

<div class="smallcode">
```nix
nixpkgs.config.packageOverrides = pkgs: rec {
  simp_le = pkgs.simp_le.overrideDerivation (oldAttrs: {
    patches = [
      (pkgs.fetchpatch {
        url = "https://github.com/kuba/simp_le/commit/" +
              "4bc788fdd611c4118c3f86b5f546779723aca5a7.patch";
        sha256 = "0036p11qn3plydv5s5z6i28r6ihy1ipjl0y8la0izpkiq273byfc";
      })
    ];
  });
};
```
</div>

<figure class="stretch"><img src="img/mind-blown.gif" alt=""></figure>

<div class="notes">
completely override package, not only the installed instance
</div>


# Conclusion

Nix & NixOS have very interesting properties that can make server management **easier** and **safer**.

You can use software practices to do sysadmin: modularity, DRY, versioning, abstractions, ...

> The NixOS project is still a bit young, but it grows and you should definitely take a look!
>
> &rarr; <https://nixos.org>


# References

- [NixOS manual](https://nixos.org/nixos/manual/)
- [NixOS options](http://nixos.org/nixos/options.html) & [NixOS packages](http://nixos.org/nixos/packages.html)
- [Nix manual](https://nixos.org/nix/manual/)
- [Nixpkgs source code](https://github.com/NixOS/nixpkgs)
- [Nix pills](https://lethalman.blogspot.fr/search/label/nixpills)
- [Nix/NixOS papers](https://nixos.org/docs/papers.html)

<figure class="stretch"><img src="img/reading.gif" alt=""></figure>


# Questions?

<figure class="stretch"><img src="img/question.gif" alt=""></figure>

Twitter: [\@d_sferruzza](https://twitter.com/d\_sferruzza)

Slides on GitHub:

[dsferruzza/conf-immutable-infrastructure-with-nixos](http://github.com/dsferruzza/conf-immutable-infrastructure-with-nixos)
