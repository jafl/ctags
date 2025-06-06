.. _ctags-lang-elm(7):

==============================================================
ctags-lang-elm
==============================================================

Random notes about tagging Elm source code with Universal Ctags

:Version: 6.2.0
:Manual group: Universal Ctags
:Manual section: 7

SYNOPSIS
--------
|	**ctags** ... --languages=+Elm ...
|	**ctags** ... --language-force=Elm ...
|	**ctags** ... --map-Elm=+.elm ...

DESCRIPTION
-----------
The Elm parser is a PEG parser using PackCC, which is part of the
ctags infrastructure. It should correctly process all top level
statements, however there is a limitation with functions embedded
in let/in blocks. They will mostly be fine, but sometimes a
function in a let/in block will be omitted.

EXAMPLES
--------

Imports
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Imported modules are tagged, and their role is "imported", not "def".
Types, functions, etc which are exposed via imported module have their
role as "exposed".

Exposed items are marked as being in the scope of their own module,
not the module that's doing the importing.

"input.elm"

.. code-block:: Elm

	module SomeMod exposing (..)

	import MyMod exposing
	  ( map
	  , Maybe
	  , Result(..)
	  , MyList(Empty)
	  )

"output.tags"
with "--options=NONE -o - --sort=no --extras=+r --fields=+r input.elm"

.. code-block:: tags

	SomeMod	input.elm	/^module SomeMod exposing (..)$/;"	m	roles:def
	MyMod	input.elm	/^import MyMod exposing$/;"	m	roles:imported
	map	input.elm	/^  ( map$/;"	f	module:MyMod	roles:exposed
	Maybe	input.elm	/^  , Maybe$/;"	t	module:MyMod	roles:exposed
	Result	input.elm	/^  , Result(..)$/;"	t	module:MyMod	roles:exposed
	MyList	input.elm	/^  , MyList(Empty)$/;"	t	module:MyMod	roles:exposed
	Empty	input.elm	/^  , MyList(Empty)$/;"	c	type:MyMod.MyList	roles:exposed

Namespaces
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Namespaces are tagged and their role is "def".

"input.elm"

.. code-block:: Elm

	module AMod exposing (..)

	import MyImport as NSpace exposing (impFunc)

"output.tags"
with "--options=NONE -o - --sort=no --extras=+r --fields=+r input.elm"

.. code-block:: tags

	AMod	input.elm	/^module AMod exposing (..)$/;"	m	roles:def
	NSpace	input.elm	/^import MyImport as NSpace exposing (impFunc)$/;"	n	module:AMod	roles:def	moduleName:MyImport
	MyImport	input.elm	/^import MyImport as NSpace exposing (impFunc)$/;"	m	roles:imported
	impFunc	input.elm	/^import MyImport as NSpace exposing (impFunc)$/;"	f	module:MyImport	roles:exposed

Type names
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Constructors top level functions will have type names.

"input.elm"

.. code-block:: Elm

	funcA : Int -> Int
	funcA a = a + 1

	type B
	    = B1Cons
	      { x : Float
	      , y : Float
	      }
	    | B2Cons String Integer
	    | B3Cons

"output.tags"
with "--options=NONE -o - --sort=no --extras=+r --fields=+r input.elm"

.. code-block:: tags

	funcA	input.elm	/^funcA a = a + 1$/;"	f	typeref:typename:Int -> Int	roles:def
	B	input.elm	/^type B$/;"	t	roles:def
	B1Cons	input.elm	/^    = B1Cons$/;"	c	type:B	typeref:typename:{ x : Float , y : Float } -> B	roles:def
	B2Cons	input.elm	/^    | B2Cons String Integer$/;"	c	type:B	typeref:typename:String -> Integer -> B	roles:def
	B3Cons	input.elm	/^    | B3Cons$/;"	c	type:B	typeref:typename:B	roles:def

Function parameter lists
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Function parameter lists can be extracted into the tags file
signature field. They are not really function signatures, but
it's the closest concept available in ctags.
Use "--fields=+S".

.. code-block:: Elm

    funcA a1 a2 =
        a1 + a2

"output.tags"
with "--sort=no --extras=+r --fields=+rS"

.. code-block:: tags

    funcA	input.elm	/^funcA a1 a2 =$/;"	f	signature:a1 a2	roles:def

KNOWN LIMITATIONS
-----------------
The ctags signature field is used for function parameter lists, even
though it's not an idea field. See above.

Elm requires all statements at the same logical level to have the
same indentation. If there is additional indentation that line is part
of the previous one. Therefore without over-complicating the
PEG parser we have the following limitations...

Sometimes functions in let/in blocks will be omitted.

Functions in let/in blocks will be marked as being in the scope of their
outer function, regardless of how deeply nested the let/in block is.

Functions in let/in blocks won't have type names.

SEE ALSO
--------
:ref:`ctags(1) <ctags(1)>`, :ref:`ctags-client-tools(7) <ctags-client-tools(7)>`
