..  Copyright (c) 2019--2025 EditorConfig Team
    All rights reserved.

    Redistribution and use in source and binary forms, with or without
    modification, are permitted provided that the following conditions are met:

    1. Redistributions of source code must retain the above copyright notice,
       this list of conditions and the following disclaimer.
    2. Redistributions in binary form must reproduce the above copyright
       notice, this list of conditions and the following disclaimer in the
       documentation and/or other materials provided with the distribution.

    THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
    AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
    IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
    ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
    LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
    CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
    SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
    INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
    CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
    ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
    POSSIBILITY OF SUCH DAMAGE.

.. highlight:: text

EditorConfig Specification
^^^^^^^^^^^^^^^^^^^^^^^^^^

This is version |version| of this specification.

.. contents:: Table of Contents

Introduction (informative)
==========================

*All content in this document is normative unless marked "(informative)".*

EditorConfig helps maintain consistent coding styles for multiple developers
working on the same project across various editors and IDEs. The EditorConfig
project consists of a file format for defining coding styles and a collection
of text editor plugins that enable editors to read the file format and adhere
to defined styles. EditorConfig files are easily readable and they work nicely
with version control systems.


Terminology
===========

.. versionchanged:: 0.15.1

In EditorConfig:

- "EditorConfig files" (named ``.editorconfig`` in lowercase) include
  section(s) storing key-value pairs. EditorConfig files must conform to this
  specification.
- "Cores" parse files conforming to this specification, and provide
  key-value pairs to plugins.
- "Plugins" receive key-value pairs from cores and update an editor's
  settings based on the key-value pairs.
- "Editors" permit editing files, and use plugins to update settings for
  files being edited.
- The words "tab" and "hard tab" are interchangable and represent the 
  character defined by the Unicode HT/TAB symbol (U+0009). 
- The word "space" is the Unicode character defined by the Unicode Space/SP symbol (U+0020).
- "Column" is an abstract atomic unit of indentation. A single space, as defined above, is expected
  to contribute exactly one column to the indentation of a given line. The amount of columns
  contributed by the hard tab depends on the configuration pairs defined in the :ref:`supported-pairs` section.
- The term "soft tab" represents the numerous amount (1..N) of sequential space characters, which,
  when considered together, form an indentation level that has a length equal to the length of a single 
  hard tab. The length of both the soft tab and a hard tab is measured in columns.

A conforming core or plugin must pass the tests in the
`core-tests repository`_ or `plugin-tests repository`_, respectively.

*(informative)* Some plugins include or bundle their own cores, and some rely
on external cores.  Some editors include or bundle plugin or core
functionality.  Editors, plugins, and cores may all come from different
people.  Those people may or may not have any direct interaction with the
EditorConfig organization.

File Format
===========

.. versionchanged:: 0.17.2

EditorConfig files are in an INI-like file format.
To read an EditorConfig file, take one line at a time, from beginning to end.
For each line:

#. Remove all leading and trailing whitespace.
#. Process the remaining text as specified for its type below.

The types of lines are:

- Blank: Contains nothing.  Blank lines are ignored.

- Comment: starts with a ``;`` or a ``#``.  Comment lines are ignored.

- Section Header: starts with a ``[`` and ends with a ``]``.
  These lines define globs; see :ref:`glob-expressions`.

   - May contain any characters between the square brackets (e.g.,
     ``[`` and ``]`` and even spaces and tabs are allowed).
   - Forward slashes (``/``) are used as path separators.
   - Backslashes (``\\``) are not allowed as path separators (even on Windows).

- Key-Value Pair (or Pair): contains a key and a value, separated by an ``=``.
  See :ref:`supported-pairs`.

   - Key: The part before the first ``=`` on the line.
   - Value: The part, if any, after the first ``=`` on the line.
   - Keys and values are trimmed of leading and trailing whitespace, but
     include any whitespace that is between non-whitespace characters.
   - If a value is not provided, then the value is an empty string
     (i.e., ``""`` in C or Python).

Any line that is not one of the above is invalid.

EditorConfig files must be UTF-8 encoded, with LF or CRLF line separators.

No inline comments
------------------

.. versionchanged:: 0.15.0

A ``;`` or ``#`` anywhere other than at the beginning of a line does *not*
start a comment, but is part of the text of that line.  For example::

  [*.txt]
  foo = editorconfig ;)

gives variable ``foo`` the value ``editorconfig ;)`` in ``*.txt`` files,
*not* the value ``editorconfig``.

This specification does not define any "escaping" mechanism for
``;`` or ``#`` characters.

.. admonition :: Compatibility

  The EditorConfig file format formerly allowed the use of ``;`` and ``#`` after the
  beginning of the line to mark the rest of a line as comment. This led to
  confusion how to parse values containing those characters. Old EditorConfig
  parsers may still allow inline comments.

Parts of an EditorConfig file
-----------------------------

The parts of an EditorConfig file are:

- Preamble: the lines that precedes the first section. The preamble is optional
  and may contain key-value pairs, comments and blank lines.
- Section Name: the string between the beginning ``[`` and the ending ``]``.
- Section: the lines starting from a Section Header until the beginning of
  the next Section Header or the end of the file.

.. _glob-expressions:

Glob Expressions
================

Section names in EditorConfig files are filepath globs, similar to the format
accepted by ``.gitignore``. They support pattern matching through Unix
shell-style wildcards. These filepath globs recognize the following as
special characters for wildcard matching:

.. list-table::
   :header-rows: 1

   * - Special Characters
     - Matching
   * - ``*``
     - any string of characters, except path separators (``/``)
   * - ``**``
     - any string of characters
   * - ``?``
     - any single character, except path separators (``/``)
   * - ``[seq]``
     - any single character in seq. Any character inside those brackets is
       considered literally. It means, that pattern ``[ab*c{1..2}]`` is considered literally:
       either 'a', or 'b', or '*', or 'c', or '{', or '1', or '.', or '2', or '}'.
   * - ``[!seq]``
     - any single character not in seq. Any character inside those brackets is
       considered literally as well (see example above).
   * - ``{s1,s2,s3}``
     - any of the strings given (separated by commas, can be nested) (But ``{s1}`` only matches ``{s1}`` literally.)
   * - ``{num1..num2}``
     - any integer numbers between ``num1`` and ``num2``, where ``num1`` and ``num2``
       can be either positive or negative. ``num1`` is required to be
       less than ``num2``. For instance, ``{1..3}``, ``{-1..4}`` are valid, but ``{-4..-5}``,
       ``{3..1}`` are not.

If the glob contains a path separator (a ``/`` not inside square brackets), then the glob is relative
to the directory level of the particular `.editorconfig` file itself.
Otherwise the pattern may also match at any level below the `.editorconfig`
level. For example, ``*.c`` matches any file that ends with ``.c`` in the
directory of ``.editorconfig`` or any other directory below one that stores this ``.editorconfig``.
However, the glob ``subdir/*.c`` only matches files that end
with ``.c`` in the ``subdir`` directory in the directory of ``.editorconfig``.

Therefore, a leading slash is not relevant if there is already a slash in the middle of the pattern.
Thus, the globs `/subdir/*.c` and `subdir/*.c` must yield the same result.

As a corollary, a section name ending with ``/`` does not match any file.

The backslash character (``\\``) can be used to escape a character so it is
not interpreted as a special character.

Cores must accept section names with length up to and including 1024 characters.
Beyond that, each implementation may choose to define its own upper limit or no explicit upper limit at all.

File Processing
===============

When a filename is given to EditorConfig a search is performed in the
directory of the given file and all parent directories for an EditorConfig
file. An EditorConfig file is named ".editorconfig", all lowercased.
Non-existing directories are treated as if they exist and are empty. All found
EditorConfig files are searched for sections with section names matching the
given filename. The search shall stop if an EditorConfig file is found with
the ``root`` key set to ``true`` in the preamble or when reaching the root
filesystem directory.

Files are read top to bottom and the most recent pairs found take
precedence. Thus, in case a given file matches multiple sections
within a single ``.editorconfig`` file, the pairs defined in the section that
comes later in the ``.editorconfig`` file take precedence over pairs defined
in the section that comes earlier in the same ``.editorconfig`` file. 
If multiple EditorConfig files have matching sections, the pairs
from the closer EditorConfig file are read last, so pairs in closer
files take precedence.

Capitalization of the File Name
-------------------------------

As noted above, the ``.editorconfig`` filename should be lowercased. On some
platforms, opening a file with a different capitalization results in opening
the same file with lowercased file names. On such a platform, in addition to
the all lowercased ``.editorconfig`` file name, a Core may choose to also
accept files with a different capitalization as if it were all lowercased.

*(informative)* Such platform is common with a case-insensitive filesystem.
For example, a file named ``.editorConfig`` exists in the filesystem, but
opening a file named ``.editorconfig`` via a file-opening API still opens the
differently capitalized ``.editorConfig`` file. The behavior of the Core as
described in the previous paragraph is to prevent the need of the additional
operation of specifically retrieving the filename, which can be relatively
expensive in the context of EditorConfig.

.. _supported-pairs:

Supported Pairs
===============

.. versionchanged:: 0.17.1

EditorConfig file sections contain key-value pairs separated by an
equal sign (``=``). With the exception of the ``root`` key, all pairs MUST be
located under a section to take effect.

- EditorConfig cores shall accept and report all syntactically valid
  key-value pairs, even if the key is not defined in this specification.
- EditorConfig plugins shall ignore unrecognized keys and invalid/unsupported
  values.

Here is the list of all keys defined by this version of this specification,
and the supported values associated with them:

.. list-table::
   :header-rows: 1

   * - Key
     - Supported values
   * - ``indent_style``
     - Set to ``tab`` or ``space`` to use tabs or spaces for indentation, respectively. Option ``tab`` 
       implies that an indentation is to be filled by as many hard tabs as possible, with the rest of the
       indentation filled by spaces. A non-normative explanation can be found in the indentation_ section. 
       The values are case-insensitive.
   * - ``indent_size``
     - Set to a whole number defining the number of columns used for each
       indentation level and the width of soft tabs (when supported). If this
       equals ``tab``, the ``indent_size`` shall be set to the tab size, which
       should be ``tab_width`` (if specified); else, the tab size set by the
       editor. The values are case-insensitive.
   * - ``tab_width``
     - Set to a whole number defining the number of columns used to represent
       a tab character. This defaults to the value of ``indent_size`` and should
       not usually need to be specified.
   * - ``end_of_line``
     - Set to ``lf``, ``cr``, or ``crlf`` to control how line breaks are
       represented. The values are case-insensitive.
   * - ``charset``
     - Set to ``latin1``, ``utf-8``, ``utf-8-bom``, ``utf-16be`` or ``utf-16le`` to
       control the character set. Use of ``utf-8-bom`` is discouraged.
   * - ``spelling_language``
     - Sets the natural language that should be used for spell checking.
       Only one language can be specified.  There is no default value.

       The format is ``ss`` or ``ss-TT``, where ``ss`` is an `ISO 639`_
       two-letter language code and ``TT`` is an `ISO 3166`_ two-letter
       territory identifier.  (Therefore ``spelling_language`` must be
       either two or five characters long.)

       **Note:** This property does **not** specify the charset to be used.
       The charset is in separate property ``charset``.
   * - ``trim_trailing_whitespace``
     - Set to ``true`` to remove all whitespace characters preceding newline
       characters in the file and ``false`` to ensure it doesn't.
   * - ``insert_final_newline``
     - Set to ``true`` ensure file ends with a newline when saving and ``false``
       to ensure it doesn't.  Editors must not insert newlines in empty files
       when saving those files, even if ``insert_final_newline = true``.

   * - ``root``
     - Must be specified in the preamble.  Set to ``true`` to tell the core
       not to check any higher directory for EditorConfig settings for on the
       current filename.  The value is case-insensitive.

For any pair, a value of ``unset`` removes the effect of that
pair, even if it has been set before. For example, add ``indent_size =
unset`` to undefine the ``indent_size`` pair (and use editor defaults).

Pair keys are case-insensitive. All keys are lowercased after parsing.

Cores must accept keys and values with lengths up to and including 1024 and 4096 characters respectively.
Beyond that, each implementation may choose to define its own upper limits or no explicit upper limits at all.

.. indentation:

Indentation (Non-Normative)
===========================
The indentation related options (``indent_style``, ``indent_size`` and ``tab_width``) require a special documentation
section to specify their behavior. Consider the following code snippet:

.. code-block:: python

    def execute():
        source = "indentation is important"
        for i in source.split(" "):
            print(i)

The ``indent_size`` setting for this code snippet equals 4, because ``indent_size`` means how many columns are required
to indent the next line in relation to previous (if indentation, of course, is applicable for this line). Then the next question
is *how* this indentation of 4 columns is achieved. It may be 4 consequent spaces,
a single tab with width equal to 4, or two tabs with width equal to 2.

This is when ``indent_style`` comes into picture. It specifies what character should be used **whenever possible** in order to
achieve the indentation size specified in ``indent_size``. To fully understand what "whenever possible" actually means, let's
consider the following EditorConfig file living in the same directory as the file above:

.. code-block:: ini

    root = true
    [example_file.py]
    indent_style = tab
    indent_size = 4
    tab_width = 3

The ``indent_size`` of 4 is not achievable by placing 1 or 2 consequent tabs, because ``tab_width = 3``. Therefore,
in order to comply with this EditorConfig configuration, the new lines (where indentation is applicable) **must be precisely
indented with one tab, and one space**. That is because by placing one tab we're not achieving the ``indent_size`` required, but by
placing the 2 consequent tabs we're overreaching. Therefore, although ``indent_style`` is ``tab``, we still have to supplement
with one space character to fulfill the requirement.

For another example, if we have the following EditorConfig file:

.. code-block:: ini

    root = true
    [another_file.py]
    indent_style = tab
    indent_size = 8
    tab_width = 4

One **MUST** expect that spaces will not be used at all for indentation, since all the indentation can be achieved via tabs only.

Additionally, it is possible to have ``indent_size`` less than the ``tab_width``.

.. code-block:: ini

    root = true
    [another_file.py]
    indent_style = tab
    indent_size = 4
    tab_width = 8

To understand the way it works, let's look at the following example:

.. code-block:: python

    def func():
        if True:
            return True

In this case, the line where the ``if`` statement condition is specified is indented with 4 spaces, because the ``indent_size = 4``
and the tab cannot fit in. On the other hand, the line with ``return`` statement must be indented with one tab, because the
indentation level for this line is 8 columns, and a tab can fit in.

Suggestions for Plugin Developers
=================================

TODO. For now please read the `Plugin Guidelines`_ on GitHub wiki.

Versioning
==========

*This section applies beginning with version 0.14.0 of this specification.*

This specification has a version, tagged in the `specification repository`_.
Each specification version corresponds to the same version in the
`core-tests repository`_.

The version numbering of the specification follows
`Semantic Versioning 2.0.0`_ ("SemVer").  The version numbering of
the `core-tests repository`_ also follows SemVer.

Each EditorConfig core, to pass the core tests, must process version
numbers given with the ``-b`` switch, and must report version numbers when
given ``-v`` or ``--version``.  The version numbers used for ``-b``, ``-v``,
and ``--version`` are versions of this specification.  For example, the
Vimscript core might respond to ``-v`` with:

::

  EditorConfig Vimscript core v1.0.0 - Specification Version 0.14.0

Cores, plugins, or editors supporting EditorConfig have their own version
numbers.  Those version numbers are independent of the version number of
this specification.

.. _core-tests repository: https://github.com/editorconfig/editorconfig-core-test
.. _ISO 639: https://en.wikipedia.org/wiki/ISO_639
.. _ISO 3166: https://en.wikipedia.org/wiki/ISO_3166
.. _Python configparser Library: https://docs.python.org/3/library/configparser.html
.. _Plugin Guidelines: https://github.com/editorconfig/editorconfig/wiki/Plugin-Guidelines
.. _plugin-tests repository: https://github.com/editorconfig/editorconfig-plugin-tests
.. _Semantic Versioning 2.0.0: https://semver.org/spec/v2.0.0.html
.. _specification repository: https://github.com/editorconfig/specification

