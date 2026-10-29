================
launch-editor-py
================

.. image:: https://img.shields.io/github/actions/workflow/status/adamchainz/launch-editor-py/main.yml.svg?branch=main&style=for-the-badge
   :target: https://github.com/adamchainz/launch-editor-py/actions?workflow=CI

.. image:: https://img.shields.io/badge/Coverage-100%25-success?style=for-the-badge
   :target: https://github.com/adamchainz/launch-editor-py/actions?workflow=CI

.. image:: https://img.shields.io/pypi/v/launch-editor-py.svg?style=for-the-badge
   :target: https://pypi.org/project/launch-editor-py/

.. image:: https://img.shields.io/badge/code%20style-black-000000.svg?style=for-the-badge
   :target: https://github.com/psf/black

.. image:: https://img.shields.io/badge/pre--commit-enabled-brightgreen?logo=pre-commit&logoColor=white&style=for-the-badge
   :target: https://github.com/pre-commit/pre-commit
   :alt: pre-commit

----

Open files in the user’s editor, optionally at a given line and column.

A Python port of the `launch-editor <https://github.com/vitejs/launch-editor>`__ npm package, as used by Vite to open files from the browser.
It detects which editor is running, and knows how to pass a line and column to many editors.

----

**Get better at command line Git** with my book `Boost Your Git DX <https://adamchainz.gumroad.com/l/bygdx>`__.

----

Requirements
------------

Python 3.11 to 3.15 supported.

Installation
------------

1. Install with **pip**:

   .. code-block:: sh

       python -m pip install launch-editor-py

   The package imports as ``launch_editor``.

Usage
-----

Call ``launch_editor()`` with a filename, and optionally a line and column:

.. code-block:: python

    from launch_editor import LaunchEditorError, launch_editor

    try:
        launch_editor("src/example.py", line=12, column=4)
    except LaunchEditorError as exc:
        print(f"Could not open editor: {exc}")

API
---

``launch_editor(filename, line=None, column=None, *, editor=None)``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Open ``filename`` in the user’s editor, at ``line`` and ``column`` if given, and return the editor’s ``subprocess.Popen`` object.

The editor is picked by ``guess_editor()``, passing it ``editor``.

GUI editors start in the background, and the function returns immediately.
If a GUI editor exits with an error, a warning is logged to the ``launch_editor`` logger.

Terminal editors, like Vim, take over the current terminal until they exit.
While one is open, further attempts to open a terminal editor raise ``LaunchEditorError``.

Raises ``LaunchEditorError`` if the file does not exist, no editor is found, or the editor cannot be run.

``guess_editor(editor=None)``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Return the command for the user’s editor, as a list of strings, or an empty list if none is found.
The editor is chosen from, in order:

1. The ``editor`` argument, as a command string like ``"code --new-window"`` or a list of arguments.
2. The ``LAUNCH_EDITOR`` environment variable.
3. A supported editor that is currently running, on macOS, Linux, and Windows.
4. The ``VISUAL`` environment variable.
5. The ``EDITOR`` environment variable.

``LAUNCH_EDITOR`` is shared with the launch-editor npm package, so you can set it once for both.
Like there, it’s treated as a single command, not split into arguments.
It may name a script, which is passed the file, line, and column as separate arguments, unless it’s a supported editor.

``position_args(editor, filename, line, column=None)``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Return the arguments that open ``filename`` at ``line`` and ``column`` in the given ``editor`` command.
Unsupported editors get only the filename, since passing a position incorrectly can cause errors or confusing behaviour.

``LaunchEditorError``
~~~~~~~~~~~~~~~~~~~~~

A basic exception subclass, raised when the editor cannot be launched.

Supported editors
-----------------

Opening at a line and column is supported for:

* VS Code and its forks: Antigravity, Cursor, Trae, VSCodium
* JetBrains IDEs: AppCode, CLion, GoLand, IntelliJ IDEA, PhpStorm, PyCharm, Rider, RubyMine, WebStorm
* Atom, Emacs, Helix, joe, micro, nano, Neovim, Notepad++, Sublime Text, TextMate, Vim, Zed

Command line
------------

Run ``python -m launch_editor`` to open a file from the command line, or check which editor would be used:

.. code-block:: console

    $ python -m launch_editor --guess
    code
    $ python -m launch_editor src/example.py:12:4

Full help:

.. [[[cog
.. import cog
.. import os
.. import subprocess
.. import sys
.. result = subprocess.run(
..     [sys.executable, "-m", "launch_editor", "--help"],
..     capture_output=True,
..     text=True,
..     env={**os.environ, "COLUMNS": "80"},
.. )
.. cog.outl("")
.. cog.outl(".. code-block:: console")
.. cog.outl("")
.. for line in result.stdout.splitlines():
..     if line.strip() == "":
..         cog.outl("")
..     else:
..         cog.outl("   " + line.rstrip())
.. cog.outl("")
.. ]]]

.. code-block:: console

   usage: python -m launch_editor [-h] [--editor EDITOR] [--guess] [FILE]

   Open a file in your editor.

   positional arguments:
     FILE             The file to open, optionally followed by :LINE or
                      :LINE:COLUMN, like example.py:12:4.

   options:
     -h, --help       show this help message and exit
     --editor EDITOR  The editor command to use, instead of guessing.
     --guess          Print the editor command that would be used, and exit.

.. [[[end]]]

Security
--------

Web applications often use this package to open files from a browser, via an endpoint on a local development server.
Such an endpoint runs a program, so protect it:

* Accept an identifier for a file the application already knows about, rather than an arbitrary path.

* Only accept POST requests, and check the ``Sec-Fetch-Site`` and ``Origin`` headers to reject cross-origin requests.
  Any website the user visits can send requests to ``localhost``.

* Check the ``Host`` header is a local name, like ``localhost`` or ``127.0.0.1``, to prevent DNS rebinding attacks.

Filenames are passed to editors as absolute paths, so a name starting with ``-`` cannot be interpreted as an option.

On Windows, editors launched through batch files, such as VS Code’s ``code.cmd``, run through ``cmd.exe``, which has its own command line parsing.
To prevent command injection, launch-editor-py quotes every argument and refuses to pass arguments containing ``"``, ``%``, ``!``, or line breaks to them.
UNC paths are also refused on Windows, like the npm package, to avoid leaking NTLM credentials to remote hosts—see `its explanation <https://github.com/vitejs/launch-editor/tree/main/packages/launch-editor#unc-paths-on-windows>`__.

Differences from the launch-editor npm package
----------------------------------------------

This package was last compared against the npm package at version `2.14.2 <https://github.com/vitejs/launch-editor/tree/v2.14.2>`__ (commit ``b533489``), on 2026-10-28.
Upstream ``main`` had no further changes to the package at that time.

It also includes the fix from upstream `PR #163 <https://github.com/vitejs/launch-editor/pull/163>`__, unmerged at that time, so Cursor, Trae, and Antigravity open at the given line and column when detected running on Windows.

Additions:

* Opening at a line and column for Helix (``hx``), micro, nano, Neovim (``nvim``), and vi.

* Helix, micro, Neovim, and vi are treated as terminal editors, alongside Emacs, nano, and Vim.

* A command line interface, ``python -m launch_editor``.

Changes:

* Errors raise ``LaunchEditorError``, rather than calling an error callback and printing to the console.
  A missing file is an error, rather than being silently ignored.
  A GUI editor that exits with an error logs a warning to the ``launch_editor`` logger.

* ``launch_editor()`` takes the line and column as separate arguments, rather than parsing them from ``file:line:column``.
  The command line interface still accepts ``file:line:column``, but not ``file://`` URLs.

* Opening a terminal editor while one is already open raises an error, rather than killing the open editor.

* ``VISUAL`` and ``EDITOR`` are split into arguments, so values like ``code --wait`` work.
  ``LAUNCH_EDITOR`` is still treated as a single command, like the npm package.

* On Linux, running editors are detected by exact process name, rather than by substring, so for example ``nvim`` isn’t detected as ``vim``.

* On Windows, editor executables are run directly, rather than through ``cmd.exe``.
  Batch file editors, like VS Code’s ``code.cmd``, are still run through ``cmd.exe``, with every argument quoted, rather than escaped with ``^``.
  Arguments containing ``"``, ``%``, ``!``, or line breaks are refused for them, where the npm package refuses only ``%`` and line breaks.

* There’s no special handling for StackBlitz WebContainers.

License
-------

MIT.
This package is a port of the launch-editor npm package, Copyright (c) 2017-present Yuxi (Evan) You, which in turn derives from code in create-react-app, Copyright (c) 2015-present Facebook, Inc., both MIT licensed.
See the ``LICENSE`` file for details.
