CLI-Scanner
===========

Small helper tool to scan documents via the CLI. It wraps several CLI-based
tools which should be available in the Linux distribution.

Supports:

* Manual multi-page documents
* ADF multi-page documents
* Single-Page documents


Requirements
============

* Python 3
* ``scanimage``
* Ghostscript (for the ``gs`` command)
* ``imagemagick``


Installation
============

For easy acces, simply drop ``cliscan`` somewhere in your path, for example::

    ${HOME}/.local/bin

You can also simply run it as-is. There are no third-party Python packages
needed. As long as the requirements above are met you are good to go.


Synopsis
========

See::

    cliscan --help


Details
=======

The format is currently hard-coded as DIN-A4, 150dpi grayscale but can easily
be modified in the code.
