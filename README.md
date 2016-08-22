# Clair on Barge with Vagrant

https://github.com/coreos/clair

> Clair is an open source project for the static analysis of vulnerabilities in appc and docker containers.

This builds a Clair environment with [Vagrant](https://www.vagrantup.com/) instantly.

## Requirements

- [VirtualBox](https://www.virtualbox.org/)
- [Vagrant](https://www.vagrantup.com/)

## Boot up

```bash
$ vagrant up
```

That's it.

https://github.com/coreos/clair#hello-heartbleed

> During the first run, Clair will bootstrap its database with vulnerability data from its data sources. It can take several minutes before the database has been fully populated.

You can check the status by using `docker logs clair` and you will see `updater: update finished`.

```bash
$ docker logs clair
2016-08-21 23:40:28.804803 I | pgsql: running database migrations
goose: migrating db environment '', current version: 0, target: 20151222113213
OK    20151222113213_Initial.sql
2016-08-21 23:40:29.101777 I | pgsql: database migration ran successfully
2016-08-21 23:40:29.114150 I | notifier: notifier service is disabled
2016-08-21 23:40:29.114338 I | api: starting main API on port 6060.
2016-08-21 23:40:29.115062 I | api: starting health API on port 6061.
2016-08-21 23:40:29.115157 I | updater: updater service started. lock identifier: 11bf7927-b6f2-4b4e-a5ac-62c292f7b9c7
2016-08-21 23:40:29.237012 I | updater: updating vulnerabilities
2016-08-21 23:40:29.237058 I | updater: fetching vulnerability updates
2016-08-21 23:40:29.237089 I | updater/fetchers/ubuntu: fetching Ubuntu vulnerabilities
2016-08-21 23:40:29.238441 I | updater/fetchers/debian: fetching Debian vulnerabilities
2016-08-21 23:40:29.239648 I | updater/fetchers/rhel: fetching Red Hat vulnerabilities
2016-08-21 23:54:17.457960 I | updater: adding metadata to vulnerabilities
2016-08-22 00:18:51.876149 I | updater: update finished
```

## Analyse vulnerabilities in a Docker image with `analyze-local-images` command

```bash
$ vagrant ssh
[bargee@barge ~]$ docker pull ailispaw/ubuntu-essential:14.04
14.04: Pulling from ailispaw/ubuntu-essential
c298559fc8ae: Pull complete
1c93d6585dd1: Pull complete
Digest: sha256:8bed724d571307e245a27ac50ba0b0ee2119b5ba7b57109fbddfbb5466679241
Status: Downloaded newer image for ailispaw/ubuntu-essential:14.04
[bargee@barge ~]$ analyze-local-images -minimum-severity Medium ailispaw/ubuntu-essential:14.04
2016-08-22 02:27:10.048230 I | Saving ailispaw/ubuntu-essential:14.04 to local disk (this may take some time)
2016-08-22 02:27:14.070162 I | Retrieving image history
2016-08-22 02:27:14.103607 I | Analyzing 2 layers...
2016-08-22 02:27:14.103791 I | Analyzing c298559fc8ae67275ae2d0d36cfbc3a2960b15440d6dcedbec13d0174ace4e1d
2016-08-22 02:27:14.114818 I | Analyzing 1c93d6585dd188283a8706f1df22d662cdf7a49700286a582f453e2a24715eba
2016-08-22 02:27:14.117480 I | Retrieving image's vulnerabilities
Clair report for image ailispaw/ubuntu-essential:14.04 (2016-08-22 02:27:14.14364884 +0000 UTC)
CVE-2016-2781 (Medium)
        nonpriv session can escape to the parent session by using the TIOCSTI ioctl

        Package:       coreutils @ 8.21-1ubuntu5.4
        Link:          http://people.ubuntu.com/~ubuntu-security/cve/CVE-2016-2781
        Layer:         c298559fc8ae67275ae2d0d36cfbc3a2960b15440d6dcedbec13d0174ace4e1d

CVE-2016-1238 (Medium)
        (1) cpan/Archive-Tar/bin/ptar, (2) cpan/Archive-Tar/bin/ptardiff,
        (3) cpan/Archive-Tar/bin/ptargrep, (4) cpan/CPAN/scripts/cpan,
        (5) cpan/Digest-SHA/shasum, (6) cpan/Encode/bin/enc2xs, (7)
        cpan/Encode/bin/encguess, (8) cpan/Encode/bin/piconv, (9)
        cpan/Encode/bin/ucmlint, (10) cpan/Encode/bin/unidump, (11)
        cpan/ExtUtils-MakeMaker/bin/instmodsh, (12) cpan/IO-Compress/bin/zipdetails,
        (13) cpan/JSON-PP/bin/json_pp, (14) cpan/Test-Harness/bin/prove, (15)
        dist/ExtUtils-ParseXS/lib/ExtUtils/xsubpp, (16) dist/Module-CoreList/corelist,
        (17) ext/Pod-Html/bin/pod2html, (18) utils/c2ph.PL, (19) utils/h2ph.PL,
        (20) utils/h2xs.PL, (21) utils/libnetcfg.PL, (22) utils/perlbug.PL, (23)
        utils/perldoc.PL, (24) utils/perlivp.PL, and (25) utils/splain.PL in Perl 5.x
        before 5.22.3-RC2 and 5.24 before 5.24.1-RC2 do not properly remove . (period)
        characters from the end of the includes directory array, which might allow local
        users to gain privileges via a Trojan horse module under the current working
        directory.

        Package:       perl @ 5.18.2-2ubuntu1.1
        Link:          http://people.ubuntu.com/~ubuntu-security/cve/CVE-2016-1238
        Layer:         c298559fc8ae67275ae2d0d36cfbc3a2960b15440d6dcedbec13d0174ace4e1d

```
