% How immutable infrastructure and declarative configuration removed the pain of managing my servers
% David Sferruzza
% 16/09/2016


# About me

- [\@d_sferruzza](https://twitter.com/d\_sferruzza)
- [github.com/dsferruzza](https://github.com/dsferruzza)
- raptor trainer at [Startup Palace](http://www.startup-palace.com) *!!! FIXME !!!*
- PhD student in software engineering at *Université de Nantes*

<figure class="stretch"><img src="img/sp.gif" alt=""></figure>


# Startup Palace

We **iteratively** design and develop web applications. <small>(among other things)</small>

Frequent reviews by the customers (and by the team) are an important part of the job.

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


# About automatic deployment

We use [GitLab](https://about.gitlab.com/)

<figure class="stretch"><img src="img/gitlab.svg" alt=""></figure>

> GitLab unifies chat, issues, code review, CI and CD into a single UI


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

- it needs a lot of manual setup<br>=> admins need to be watchful and rigorous
- environments can diverge<br>=> it can become a mess
- security/system update can break stuff<br>=> you need to fix environments one by one
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
    texlive-full
```


# Dockerfile

This is fine. But:

- you declare **how** to build an image<br>not **what** is the image you want to build
- this is too linear (cache)
- it's hard to compose
- ...

<figure class="stretch"><img src="img/this-is-fine.jpg" alt=""></figure>


# Declarative vs imperative

> **Imperative programming**: use statements that change a program's state
>
> **Declarative programming**: express the logic of a computation without describing its control flow

- imperative: shopping instructions
- declarative: shopping list


# The NixOS project

> <figure class="stretch"><img src="img/nixos.svg" alt=""></figure>
> <https://nixos.org/>

- Nix (package manager)
- Nix (language)
- NixOS (distribution)
- ...
