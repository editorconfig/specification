..  Copyright (c) 2019 EditorConfig Team
    All rights reserved.

    Redistribution and use in source and binary forms, with or without
    modification, are permitted provided that the following conditions are met:

    1. Redistributions of source code must retain the above copyright notice,
        this list of conditions and the following disclaimer.
    2. Redistributions in binary form must reproduce the above copyright notice,
        this list of conditions and the following disclaimer in the documentation
        and/or other materials provided with the distribution.

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


EditorConfig Specification
^^^^^^^^^^^^^^^^^^^^^^^^^^

EditorConfig helps maintain consistent coding styles for multiple developers
working on the same project across various editors and IDEs. The EditorConfig
project consists of a file format for defining coding styles and a collection
of text editor plugins that enable editors to read the file format and adhere
to defined styles. EditorConfig files are easily readable and they work nicely
with version control systems.

.. contents:: Table of Contents

EditorConfig File Format
========================

EditorConfig files use an INI format that is compatible with the format used
by Python ConfigParser Library, but ``[`` and ``]`` are allowed in the section
names. The section names are filepath globs, similar to the format accepted by
gitignore. Forward slashes (``/``) are used as path separators and semicolons
(``;``) or octothorpes (``#``) are used for comments. Comments should go
individual lines. EditorConfig files should be UTF-8 encoded, with either CRLF
or LF line separators.

Filename globs containing path separators (``/``) match filepaths in the same
way as the filename globs used by ``.gitignore`` files. Backslashes (``\\``) are
not allowed as path separators.

A semicolon character (``;``) starts a line comment that terminates at the end
of the line. Line comments and blank lines are ignored when parsing. Comments
may be added to the ends of non-empty lines. An octothorpe character (``#``) may
be used instead of a semicolon to denote the start of a comment.

Filename and Location
=====================

When a filename is given to EditorConfig a search is performed in the
directory of the given file and all parent directories for an EditorConfig
file (named ".editorconfig" by default).  All found EditorConfig files are
searched for sections with section names matching the given filename. The
search will stop if an EditorConfig file is found with the root property set
to true or when reaching the root filesystem directory.

Files are read top to bottom and the most recent rules found take
precedence. If multiple EditorConfig files have matching sections, the rules
from the closer EditorConfig file are read last, so properties in closer
files take precedence.

Wildcard Patterns
=================

Section names in EditorConfig files are filename globs that support pattern
matching through Unix shell-style wildcards. These filename globs recognize
the following as special characters for wildcard matching:

.. list-table::
   :header-rows: 1

   * - Special Characters
     - Matching
   * - ``*``
     - any string of characters, except path separators (``/``)
   * - ``**``
     - any string of characters
   * - ``?``
     - any single character
   * - ``[seq]``
     - any single character in seq
   * - ``[!seq]``
     - any single character not in seq
   * - ``{s1,s2,s3}``
     - any of the strings given (separated by commas, can be nested)
   * - ``{num1..num2}``
     - any integer numbers between ``num1`` and ``num2``, where ``num1`` and ``num2``
       can be either positive or negative

The backslash character (``\\``) can be used to escape a character so it is
not interpreted as a special character.

The maximum length of a section name is 4096 characters. All sections
exceeding this limit are ignored.

Supported Properties
====================

EditorConfig file sections contain properties, which are name-value pairs
separated by an equal sign (``=``). EditorConfig plugins will ignore
unrecognized property names and properties with invalid values.

Here is the list of all property names understood by EditorConfig and all
valid values for these properties:

.. list-table::
   :header-rows: 0

   * - ``indent_style``
     - Set to ``tab`` or ``space`` to use hard tabs or soft tabs respectively. The
       values are case insensitive.
   * - ``indent_size``
     - Set to a whole number defining the number of columns used for each
       indentation level and the width of soft tabs (when supported). If this
       equals to ``tab``, the ``indent_size`` will be set to the tab size, which
       should be ``tab_width`` if ``tab_width`` is specified, or the tab size set
       by editor if ``tab_width`` is not specified. The values are case
       insensitive.
   * - ``tab_width``
     - Set to a whole number defining the number of columns used to represent
       a tab character. This defaults to the value of ``indent_size`` and should
       not usually need to be specified.
   * - ``end_of_line``
     - Set to ``lf``, ``cr``, or ``crlf`` to control how line breaks are
       represented. The values are case insensitive.
   * - ``charset``
     - Set to ``latin1``, ``utf-8``, ``utf-8-bom``, ``utf-16be`` or ``utf-16le`` to
       control the character set. Use of ``utf-8-bom`` is discouraged.
   * - ``trim_trailing_whitespace``
     - Set to ``true`` to remove any whitespace characters preceeding newline
       characters and ``false`` to ensure it doesn't.
   * - ``insert_final_newline``
     - Set to ``true`` ensure file ends with a newline when saving and ``false``
       to ensure it doesn't.
   * - ``root``
     - Must be specified at the top of the file outside of any sections. Set
       to ``true`` to stop ``.editorconfig`` files search on current file. The
       value is case insensitive.

For any property, a value of ``unset`` is to remove the effect of that
property, even if it has been set before. For example, add ``indent_size =
unset`` to undefine indent_size property (and use editor default).

Property names are case insensitive and all property names are lowercased when
parsing. The maximum length of a property name is 50 characters and the
maximum length of a property value is 255 characters. Any property beyond
these limits would be ignored.
