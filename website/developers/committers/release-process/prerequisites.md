---
layout: website-normal
title: Release Prerequisites
navgroup: developers
---

Subversion repositories for release artifacts
---------------------------------------------

Apache releases are posted to dist.apache.org, which is a Subversion repository.

We have two directories here:

- https://dist.apache.org/repos/dist/release/brooklyn - this is where PMC approved releases go. Do not upload
  here until we have a vote passed on dev@brooklyn. Check out this folder and name it
  `apache-dist-release-brooklyn`
- https://dist.apache.org/repos/dist/dev/brooklyn - this is where releases to be voted on go. Make the release
  artifact, and post it here, then post the [VOTE] thread with links here. Check out this folder and name it
  `apache-dist-dev-brooklyn`.

Example:

{% highlight bash %}
svn co https://dist.apache.org/repos/dist/release/brooklyn apache-dist-release-brooklyn
svn co https://dist.apache.org/repos/dist/dev/brooklyn apache-dist-dev-brooklyn
{% endhighlight %}

When working with these folders, **make sure you are working with the correct one**, otherwise you may be publishing
pre-release software to the global release mirror network!


Software packages
-----------------

The following software packages are required during the build. Make sure you have them installed.

- A Java Development Kit, version 1.8
- `maven` and `git`
- Go Language 1.6 - usually provided by the `golang` package on popular distributions
- The `rpmbuild` command - usually provided by the `rpm` package on popular distributions
- `xmlstarlet` is required by the release script to process version numbers in `pom.xml` files;
  on mac, `port install xmlstarlet` should do the trick.
- `zip` and `unzip`
- `gnupg2`, and `gnupg-agent` if it is packaged separately (it is on Ubuntu Linux)
- `pinentry` for secure entry of GPG passphrases. If you are building remotely on a Linux machine, `pinentry-curses` is
  recommended; building on a mac, `port install pinentry-mac` is recommended.
- `md5sum` and `sha1sum` - these are often present by default on Linux, but not on Mac;
  `port install md5sha1sum` should remedy that.
- if `gpg` does not resolve (it is needed for maven), create an alias or script pointing at `gpg2 "$@"`
- the `mmv` command (usually in a package named `mmv`) will help with the final steps of the release process


GPG keys
--------

The release manager must have a GPG key to be used to sign the release. See below to install `gpg2`
(with a `gpg` alias).  The steps here also assume you have the following set
(not using `whoami` if that's not appropriate):

{% highlight bash %}
ASF_USERNAME=`whoami`
GPG_KEY=$ASF_USERNAME@apache.org
SVN_USERNAME=$ASF_USERNAME
{% endhighlight %}

If you have an existing GPG key, but it does not include your Apache email address, you can add your email address as
described [in this Superuser.com posting](https://superuser.com/a/293283). Otherwise, create a new GPG key giving your
Apache email address, using `gpg2 --gen-key` then `gpg2 --export-key $GPG_KEY > my-apache.key` and 
`gpg2 --export-secret-key -a $GPG_KEY > my-apache.private.key` in the right directory (`~/.ssh` is a good one).

Upload your GPG public key (complete with your Apache email address on it) to a public keyserver - e.g. run
`gpg2 --export --armor $GPG_KEY` and paste it into the ???submit??? box on http://pgp.mit.edu/

Look up your key fingerprint with `gpg2 --fingerprint $GPG_KEY` - it???s the long sequence of hex numbers
separated by spaces. Log in to [https://id.apache.org/](https://id.apache.org/) then copy-and-paste the fingerprint into
???OpenPGP Public Key Primary Fingerprint???. Submit.

Now add your key to the `apache-dist-release-brooklyn/KEYS` file:

{% highlight bash %}
cd apache-dist-release-brooklyn
(gpg2 --list-sigs $ASF_USERNAME@apache.org && gpg2 --armor --export $ASF_USERNAME@apache.org) >> KEYS
svn --username $SVN_USERNAME --no-auth-cache commit -m "Update brooklyn/KEYS for $GPG_KEY"
{% endhighlight %}

References:

* [Post on the general@incubator list](https://mail-archives.apache.org/mod_mbox/incubator-general/201410.mbox/%3CCAOGo0VawupMYRWJKm%2Bi%2ByMBqDQQtbv-nQkfRud5%2BV9PusZ2wnQ%40mail.gmail.com%3E)
* [GPG cheatsheet](http://irtfweb.ifa.hawaii.edu/~lockhart/gpg/gpg-cs.html)

We recommend the use of the `gpg-agent`, as the release process invokes gpg to sign a large number of artifacts, one at
a time. The agent stores its configuration in `~/.gnupg/gpg-agent.conf`. A sample configuration is shown below; it uses
the Mac OSX `pinentry-mac` program which can be obtained through MacPorts or other sources. For other platforms you will
need to change this; sometimes you can omit it completely and your OS will pick a suitable alternative. The following
two lines cause your passphrase to be cached in memory for a limited period; it will expire from the cache 30 minutes
after it was most recently accessed, or 4 hours after it was first cached.  

~~~
pinentry-program /Applications/MacPorts/pinentry-mac.app/Contents/MacOS/pinentry-mac
default-cache-ttl 1800
max-cache-ttl 14400
~~~

If you experience trouble with PGP subsequently (when running maven):

* See [GnuPG/Pinentry Enigmail debugging](https://www.enigmail.net/support/gnupg2_issues.php) for tips on diagnosing gpg-agent communication (from the process to this agent and from this agent to the pinentry program)
* See [GnuPG Agent Options](https://www.gnupg.org/documentation/manuals/gnupg/Agent-Options.html) for extended gpg-agent debug


Maven configuration
-------------------

The release will involve uploading artifacts to Apache's Nexus instance - therefore you will need to configure your
Maven install with the necessary credentials.

You will need to add something like this to your `~/.m2/settings.xml` file:

{% highlight xml %}
<?xml version="1.0"?>
<settings xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.1.0 http://maven.apache.org/xsd/settings-1.1.0.xsd"
          xmlns="http://maven.apache.org/SETTINGS/1.1.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

    <!-- ... -->

    <servers>

        <!-- ... -->

        <!-- Required for uploads to Apache's Nexus instance. These are LDAP credentials - the same credentials you
           - would use to log in to Git and Jenkins (but not JIRA) -->
        <server>
            <id>apache.snapshots.https</id>
            <username>xxx</username>
            <password>xxx</password>
        </server>
        <server>
            <id>apache.releases.https</id>
            <username>xxx</username>
            <password>xxx</password>
        </server>

        <!-- ... -->

    </servers>

    <!-- ... -->

</settings>
{% endhighlight %}
