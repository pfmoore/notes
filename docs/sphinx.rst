Setting up a Sphinx Project
===========================

Introduction
------------

This is my first Sphinx project, so it's a good chance to use it
to document the process I went through to set it up. It's purely
documentation, no source code or anything else, so should be
fairly minimal.

Creating the Project
--------------------

To create the project, initialise a new repository. I used git,
and hosted it on github, but there's also bitbucket (and
Mercurial).

I created an empty project and added a ``README.md`` file to keep
github happy.  I then created a ``docs`` subdirectory, and ran
``sphinx-quickstart`` in it. I took most of the defaults, gave the
project a name of "Notes" and a version of 0.1.

Adding Content
--------------

First of all, I edited the ``index.rst`` page. The default title
is "Welcome to ``<project>``'s documentation!", which isn't really
appropriate for a set of notes, so I amended that. I also added an
entry to the toctree, as follows::

    .. toctree::
       :maxdepth: 2

       sphinx

This refers to a document ``sphinx.rst``, which is this one.

I then created ``sphinx.rst``, with basically whatever I wanted in
it.

Building the First Version
--------------------------

To build the documentation, I used the ``make.bat`` script that
was created by ``sphinx-quickstart``::

    ./make.bat html

Initial Issues
--------------

I made a typo in the name of the ``sphinx.rst`` file. There were a
couple of warnings about a file not in a toctree and a toctree
with a nonexistent reference, but I missed them. But it was fairly
obvious when I previewed the output, and easy to fix.

Initially, I had "Building the First Version" above as a level 1
heading. By doing so, it became a top-level entry in the TOC. It
looks like the TOC is based purely on heading levels, and the
split between different source files is not visible, unless you
make it so with your heading structure. That's probably a good
thing, but it came as a bit of a surprise. As a good practice,
therefore, I'll probably have each ``.rst`` file with a single
top-level heading and everything else level 2 or lower.

Things To Look Into
-------------------

The HTML pages have a header and footer with links to "Notes 0.1
documentation". I'm not sure how to change that.

Committing the Changes
----------------------

To commit the new files, I did the following:

1. Add a ``.gitignore`` file containing the line ``docs/_build``.
2. Run ``git add .gitignore docs`` to add everything else.

You get a warning about CRLF conversion in ``docs/Makefile``. I'm going to
assume this is benign, as I've never really got to grips with git's line ending
handling.
