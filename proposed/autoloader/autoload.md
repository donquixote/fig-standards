PSR-X: Autoloader
=================

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).


1. Overview
-----------

This PSR specifies the rules for an interoperable PHP autoloader which maps
namespaces to file system paths, and that can co-exist with any other SPL
registered autoloader.


2. Definitions
--------------

These definitions are presented in addition to the terms defined in PSR-T.

- **class**: The term _class_ refers to PHP classes, interfaces, and traits,
  with no regard to the file in which these may be defined.

- **fully qualified class name**: The full namespace and class name, with the
  leading namespace separator. (This is per the
  [Name Resolution Rules](http://php.net/manual/en/language.namespaces.rules.php)
  from the PHP manual.)

- **class "under" a namespace**: A class is "under" a namespace, if the
  class is either directly in that namespace, or anywhere within the namespace
  hierarchy under that namespace.

- **file "under" a directory**: A file is "under" a directory, if it is
  either directly within that directory, or anywhere in the filesystem hierarchy
  under that directory. (Symbolic links are allowed.)

- **autoloader**: A callback registered on the SPL autoload stack, OR a part
  of a larger (multiple-purpose) callback registered on the SPL autoload stack.


3. Specification
----------------

- To comply with PSR-X, a PHP class (in the library) must be in a namespace at
  least one level under the root namespace.

- The autoloader must have a way to determine for any given namespace, whether
  this namespace is associated with any base directories, and which base
  directories these are.

- If triggered with a fully-qualified class name, the autoloader may choose to
  include up to one PHP file.  
  However, it may only do so if
  * There is a namespace associated with a directory via the above mapping,
    where the class is "under" that namespace, and the file is "under"
    that directory.  
    We will refer to the namespace as "base namespace", and to the directory
    as "base directory".      
    We will refer to the remaining part of the fully qualified class name
    relative to the base namespace, without leading namespace separator, as
    "relative class name".
    We will refer to the remaining part of the file path relative to the base
    directory, without leading directory separator, as "relative file path".
  * The relative file path can be constructed from the relative class name
    by replacing every namespace separator with a directory separator, and
    appending the ".php" file suffix.

- The registered autoloader callback MUST NOT throw exceptions, MUST NOT
  raise errors of any level, and SHOULD NOT return a value.


4. Implementations
------------------

Implementations MAY contain additional features and MAY differ in how they are
implemented.

For example implemenations, see [AutoloadTest.php](AutoloadTest.php). Example
implementations MUST NOT be regarded as part of the specification; they are
examples only, and may change at any time.
