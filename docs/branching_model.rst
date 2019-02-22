Python SDK Branching Model
==========================

This page aims to give users and developers a basic overview of the
branching system we’re using as we continue development on the Splunk
SDK for Python. While we will attempt to follow it as closely as
possible, this document serves as a description of **guidelines** rather
than a description of **rules**.

Git Flow
--------

By and large, we are using the **Git Flow** model of branching. Rather
than repeating it here, you can read an excellent (and well worthwhile)
description on the the original author’s page:

-  `Original
   Description <http://nvie.com/posts/a-successful-git-branching-model/>`__
-  `Git Plugin <https://github.com/nvie/gitflow>`__

Branches in the ``splunk-sdk-python`` Repository
------------------------------------------------

The following list is a description of the main types of branches we
will have, and what it means to those working with the SDK.

-  ``master``: this is the main branch of the repository, and is the one
   that vistors will see when they visit the repository page on GitHub
   by default. In general, it will **always** reflect the most up to
   date release. Our goal is that cloning from master should always be
   possible and *stable*.

-  ``develop``: this is the branch where active work for the next
   release is. It should be generally stable, and if you’re actively
   working on developing the Splunk SDK for Python or want the latest
   and greatest, this is where you want to be.

-  ``feature/*``: these are “topic” branches, where larger features or
   experimental ones might be staged. There is no guarantee on when they
   will be merged back into ``develop`` (if ever)

-  ``release/*``: this is the release staging branch. Once the work in
   ``develop`` is done for a certain release, we will move it to a
   release staging branch for more testing and polishing. These are
   nearly stable branches.

We might have a variety of other branches, but these are the main ones.
If you have any questions, please :doc:`contact_us`.
