# PSR-4: Autoloader

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).


## 1. Overview

This PSR describes a way to lay out a directory structure with files that
define or make available PHP classes, so that they can be found by a compliant
autoloader, and so that human readers can easily find the class definitions
they are looking for.

It can apply to a full "library" (as defined below) or just a part of that. E.g.
another part of the "library" could use [PSR-0][] instead of PSR-4.

[PSR-0]: https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md


## 2. Definitions

- **library:** A combination of the following:
  * A collection of PHP files (and possibly other files) in a directory structure,
    or a subset of these files.
  * The PHP classes that are defined in or provided by this library, or those of
    these PHP classes that should be subject to this spec.
  * Optionally, documentation and meta information that belongs to the library,
    but might not be part of the directory structure.
  
  Note: A larger project composed of multiple libraries may again be regarded as
  a library by itself, and be subject to this spec.

- **class**: The term _class_ refers to PHP classes, interfaces, traits, and
  similar future resource definitions.

- **namespace**: A PHP namespace, as is syntactically valid after the
  [PHP `namespace` keyword](http://www.php.net/manual/en/language.namespaces.definition.php).
  `\` by itself is not an acceptable _namespace_ within this PSR.

- **namespace separator**: The PHP namespace separator symbol `\` (backslash).

- **fully qualified class name**: A full namespace and class name, such as
  `\Acme\Log\Writer\FileWriter` including the leading namespace
  separator.

- **autoloadable class**: Any class in the library intended for autoloading.  
  (Classes not intended for autoloading are not covered by this term.)

- **autoloadable class name**: The same as the fully qualified class name,
  but without a leading namespace separator.  
  Given a _fully qualified class name_ of `\Acme\Log\Writer\FileWriter`,
  the _autoloadable class name_ is `Acme\Log\Writer\FileWriter`.
  Typically, this is the value sent to the [__autoload()][] function
  and to callbacks registered with [spl_autoload_register()][].
  
- **namespace part**: The individual non-terminating parts of an _autoloadable
  class name_. Given an _autoloadable class name_ of
  `Acme\Log\Writer\FileWriter`, the _namespace parts_ are `Acme`, `Log`, and
  `Writer`. A _namespace part_ has no leading or trailing namespace separator.

- **class part**: The individual terminating part of an _autoloadable class
  name_. Given an _autoloadable class name_ of `Acme\Log\Writer\FileWriter`,
  the _class part_ is `FileWriter`, without a leading namespace separator.

- **namespace prefix**: One or more contiguous leading _namespace parts_ with
  namespace separators. Given an _autoloadable class name_ of
  `Acme\Log\Writer\FileWriter`, a _namespace prefix_ may be `Acme\`,
  `Acme\Log\`, or `Acme\Log\Writer\`. A _namespace prefix_ will include a
  leading namespace separator, but will not include trailing namespace separator.

- **relative class name**: The parts of the _autoloadable class name_ that
  appear after the _namespace prefix_. Given an _autoloadable class name_ of
  `Acme\Log\Writer\FileWriter` and a _namespace prefix_ of `Acme\Log\`, the
  _relative class name_ is `Writer\FileWriter`. A _relative class name_ MUST
  NOT include a leading namespace separator.

- **resource**: A class definition, typically a file in a file system.

- **scheme**: A resource storage-and-retrieval mechanism, typically a file
  system.

- **resource base**: A base path to _resources_ for a particular _namespace
  prefix_. Given a file system _scheme_ and a _namespace prefix_ of
  `Acme\Log\`, a _resource base_ could be `/path/to/acme-log/src/`. A _resource
  base_ will include a _scheme_-appropriate trailing separator, and could
  include a _scheme_-appropriate leading separator. For example, in a file 
  system _scheme_, that separator could be "\" or "/".

- **resource path**: A path in the _scheme_ representing a _resource_ defining
  an _autoloadable class name_. Given an _autoloadable class name_ of
  `Acme\Log\Writer\FileWriter`, a _namespace prefix_ of `Acme\Log\`, a
  UNIX-like file system _scheme_, a _resource base_ of
  `/path/to/acme-log/src`, and the specification described below, the
  _resource path_ will be `/path/to/acme-log/src/Writer/FileWriter.php`. The
  _resource path_ is not certain to exist in the _scheme_.

- **autoloader**: A function named `__autoload()`, or a callback registered with
  `spl_autoload_register()`.


## 3. Specification

### 3.1. Scope

The specification may apply to a "library" as defined above.

As explained above, this goes beyond what is commonly refered to as "library",
and may also refer to e.g. a subset of a library, or an "application" composed
of more than one library.


### 3.2. Requirements

1. For every autoloadable class in the library, the _autoloadable class name_
MUST begin with a _namespace part_, which MAY be followed by one or more
additional _namespace parts_, and MUST end in a _class part_.

    a. The beginning _namespace part_ of the _autoloadable class name_,
    sometimes called a "vendor name", MUST be unique to the developer or
    project. This is to prevent conflicts between different libraries,
    components, modules, etc.
    
    b. It is RECOMMENDED (but not required) that the _autoloadable class name_
    include a second _namespace part_, sometimes called a "package name", to
    identify its place within the "vendor name".

    > **Example:** \<Vendor Name>\(<Namespace>\)*<Class Name>

2. The library MUST explicitly or implicitly specify a relationship that
   associates _namespace prefixes_ with _resource bases_.

    a. A _namespace prefix_ MAY be associated with more than one
    _resource base_.

    b. To prevent conflicts, different _namespace prefixes_ SHOULD NOT
    be associated with the same _resource base_.

3. For every autoloadable class in the library, the relationship above MUST
   associate at least one _namespace prefix_ of the _autoloadable class name_
   with a _resource base_.

3. The resources in the library MUST be layed out so that an autoloader that
   performs the following steps, if called with an autoloadable class name
   for a class that is provided at least once in the library, will make
   exactly one version of this class available within the PHP script:

    1. For each _namespace prefix_ of the autoloadable class name, determine
    all _resource bases_ associated with it, if any.  
    (as by the previous points, this will yield at least one match)
    
    2. For every combination of _namespace prefix_ and _resource base_ found,
    take the _relative class name_ (relative to the _namespace prefix_),
    replace every namespace separator in it with a scheme-appropriate separator,
    append the ".php" suffix, and append the result to the _resource base_.  
    The result will be refered to as _resource path_.
    
    3. If any of the _resource paths_ obtained this way exists in the _scheme_,
    then include or require exactly one of them.


### 3.3. Example

The following example MUST NOT be regarded as part of the specification. It is
for example purposes only.

Given a UNIX-like file system _scheme_, a _fully qualified class name_ of
`\Acme\Log\Writer\FileWriter`, a _namespace prefix_ of `Acme\Log\`, and a
_resource base_ of `/path/to/acme-log/src/`, the above specification will
result in the following actions by a _conforming autoloader_:

1. `\Acme\Log\Writer\FileWriter` becomes `Acme\Log\Writer\FileWriter`.

2. The _namespace prefix_ is replaced with the _resource base_. That is,
`Acme\Log\Writer\FileWriter` is transformed into
`/path/to/acme-log/src/Writer\FileWriter`.

3. The _namespace separators_ in the _relative class name_ are replaced with
_scheme_-appropriate separators. That is,
`/path/to/acme-log/src/Writer\FileWriter` is tranformed into
`/path/to/acme-log/src/Writer/FileWriter`.

4. The result is appended with `.php` to give a _resource path_ of
`/path/to/acme-log/src/Writer/FileWriter.php`.

5. The _scheme_ is searched. If the _resource path_ exists, it is
included, required, or otherwise loaded so that it is available.


## 4. Implementations

1. A _conforming autoloader_ will not interfere with other autoloaders, and as
such it will not throw exceptions, raise errors of any level, and should not
return a value.

2. Developers who want their classes to be autoloadable by a _conforming
autoloader_ will specify how their _namespace prefixes_ correspond to
_resource bases_. The approach is left to the autoloader developer. It may
be via narrative documentation, meta-files, PHP source code, project-specific
conventions, or some other approach.

For example implementations of _conforming autoloaders_, please see the
[examples file][]. Example implementations MUST NOT be regarded as part of the
specification. They are examples only, and MAY change at any time.

[examples file]: psr-4-autoloader-examples.php


## 5. Resource Organization

The above specification implies a particular organizational structure for
class files. Developers MUST use the following process to determine where
a class file will be placed:

1. Pick a single _namespace prefix_ for the classes to be autoloaded.

2. Pick one or more _resource bases_ for the file locations.

3. Remove the _namespace prefix_ from the _autoloadable class name_.
    
4. The remaining _namespace parts_ become subdirectories under one of the
_resource bases_.

5. The remaining _class part_ becomes the file name and is suffixed with
`.php`.

For example, given:

- _autoloadable class names_ of `Acme\Log\Writer\FileWriter` and
  `Acme\Log\Writer\FileWriterTest`,

- a _namespace prefix_ of `Acme\Log`,

- a _resource base_ of `/path/to/acme-log/src/` for source files,

- a _resource base_ of `/path/to/acme-log/tests/` for test files,

... the resulting files MUST be organized like this:

    /path/to/acme-log/
        src/
            Writer/
                FileWriter.php
        tests/
            Writer/
                FileWriterTest.php
