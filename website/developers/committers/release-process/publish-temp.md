---
layout: website-normal
title: Publish to the staging area
navgroup: developers
---

Update the canonical Git repository
-----------------------------------

Make a signed tag for this release candidate:

{% highlight bash %}
for m in ${MODULES}; do ( cd $m && git tag -s -m "Tag release ${VERSION_NAME} release candidate ${RC_NUMBER}" rel/apache-brooklyn-${VERSION_NAME}-rc${RC_NUMBER} ); done
{% endhighlight %}

Now push the release branch and release candidate tag:

{% highlight bash %}
for m in ${MODULES}; do ( cd $m && git push apache-git $VERSION_NAME && git push apache-git rel/apache-brooklyn-${VERSION_NAME}-rc${RC_NUMBER} ); done
{% endhighlight %}


Publish the source and binary distributions to the pre-release area
-------------------------------------------------------------------

You will need to have checked out the Apache distribution Subversion repository located at
https://dist.apache.org/repos/dist/dev/brooklyn. Please refer to [Prerequisites](prerequisites.md) for
information.

In your workspace for the `dist.apache.org` repo, create a directory with the artifact name and version:

{% highlight bash %}
mkdir apache-brooklyn-${VERSION_NAME}-rc${RC_NUMBER}
{% endhighlight %}

Copy into this directory all of the artifacts from the previous step - `-src` and `-bin`, `.tar.gz`, `.zip` and `.rpm`,
and all associated `.md5`, `.sha1`, `.sha256` and `.asc` signatures. Then commit:

{% highlight bash %}
svn add apache-brooklyn-${VERSION_NAME}-rc${RC_NUMBER}
svn commit --username $SVN_USERNAME --no-auth-cache --message "Add apache-brooklyn-${VERSION_NAME}-rc${RC_NUMBER} to dist/dev/brooklyn"
{% endhighlight %}

These steps can be performed as part of the `make-release-artifacts.sh` script used earlier
if `${APACHE_DIST_SVN_DIR}` points to the appropriate subversion directory.


Close the staging repository on Apache's Nexus server
-----------------------------------------------------

*Closing* the staging repository locks it from further changes, and provides a public URL for the repository that can
be used for downloading the artifacts.

Go to the Apache Nexus server at [https://repository.apache.org/](https://repository.apache.org/) and log in using the
link in the top right (the credentials are the same as your Git and Jenkins credentials). Go to the "Staging
Repositories" page, and tick the repository with the name starting `orgapachebrooklyn`. Click the **Close** button.
Provide a description which includes the version and release candidate, e.g. `Apache Brooklyn 0.10.0-rc1`.
