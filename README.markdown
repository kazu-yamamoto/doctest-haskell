# Doctest: Test interactive Haskell examples

`doctest` is a small program, that checks [examples in Haddock comments]
(http://www.haskell.org/haddock/doc/html/ch03s08.html#id566093).  It is similar
to the [popular Python module with the same name]
(http://docs.python.org/library/doctest.html).


## Installation

`doctest` is available from
[Hackage](http://hackage.haskell.org/cgi-bin/hackage-scripts/package/doctest).
Install it, by typing:

    cabal install doctest

Make sure that Cabal's `bindir` is on your `PATH`.

On Linux:

    export PATH="$HOME/.cabal/bin:$PATH"

On Mac OS X:

    export PATH="$HOME/Library/Haskell/bin:$PATH"

On Windows it's `C:\Documents And Settings\user\Application Data\cabal\bin`.

For more information, see the [section on paths in the Cabal User Guide]
(http://www.haskell.org/cabal/users-guide/installing-packages.html#paths-in-the-simple-build-system).

## Usage

Below is a small Haskell module.
The module contains a Haddock comment with some examples of interaction.
The examples demonstrate how the module is supposed to be used.

```haskell
module Fib where

-- | Compute Fibonacci numbers
--
-- Examples:
--
-- >>> fib 10
-- 55
--
-- >>> fib 5
-- 5
fib :: Int -> Int
fib 0 = 0
fib 1 = 1
fib n = fib (n - 1) + fib (n - 2)
```

(A comment line starting with `>>>` denotes an _expression_.
All comment lines following an expression denote the _result_ of that expression.
Result is defined by what an
[REPL](http://en.wikipedia.org/wiki/Read-eval-print_loop) (e.g. ghci)
prints to `stdout` and `stderr` when evaluating that expression.)

With `doctest` you may check whether the implementation satisfies the given examples, by typing:

    doctest Fib.hs

You may produce Haddock documentation for that module with:

    haddock -h Fib.hs -o doc/

### Example groups

Examples from a single Haddock comment are grouped together and share the same
scope.  E.g. the following works:

```haskell
-- |
-- >>> let x = 23
-- >>> x + 42
-- 65
```

If an example fails, subsequent examples from the same group are skipped.  E.g.
for

```haskell
-- |
-- >>> let x = 23
-- >>> let n = x + y
-- >>> print n
```

`print n` is not tried, because `let n = x + y` fails (`y` is not in scope!).

### Setup code

You can put setup code in a [named chunk][named-chunks] with the name `$setup`.
The setup code is run before each example group.  If the setup code produces
any errors/failures, all tests from that module are skipped.

Here is an example:

```haskell
module Foo where
-- $setup
-- >>> let x = 23 :: Int

-- |
-- >>> foo + x
-- 65
foo :: Int
foo = 42
```


### Multi-line input
GHCi supports commands which span multiple lines, and the same syntax works for doctest:

```haskell
-- |
-- >>> :{
--  let
--    x = 1
--    y = 2
--  in x + y + multiline
-- :}
-- 6
multiline = 3
```

Note that `>>>` can be left of for the lines following the first: this so that
haddock does not strip leading whitespace. The expected output has whitespace
stripped relative to the :}.

Some peculiarities on the ghci side mean that whitespace at the very start is lost.
This breaks the example `broken`, since the the x and y are aligned from ghci's
perspective.  A workaround is to avoid leading space, or add a newline such
that the indentation does not matter:

```haskell
{- | >>> :{
let x = 1
    y = 2
  in x + y + works
:}
6
-}
works = 3

{- | >>> :{
 let x = 1
     y = 2
  in x + y + broken
:}
3
-}
broken = 3
```

### Multi-line output
If there are no blank lines in the output, multiple lines are handled
automatically.

```haskell
-- | >>> putStr "Hello\nWorld!"
-- Hello
-- World!
```

If however the output contains blank lines, they must be noted
explicitly with `<BLANKLINE>`. For example,

```haskell
import Data.List ( intercalate )

-- | Double-space a paragraph.
--
--   Examples:
--
--   >>> let s1 = "\"Every one of whom?\""
--   >>> let s2 = "\"Every one of whom do you think?\""
--   >>> let s3 = "\"I haven't any idea.\""
--   >>> let paragraph = unlines [s1,s2,s3]
--   >>> putStrLn $ doubleSpace paragraph
--   "Every one of whom?"
--   <BLANKLINE>
--   "Every one of whom do you think?"
--   <BLANKLINE>
--   "I haven't any idea."
--
doubleSpace :: String -> String
doubleSpace = (intercalate "\n\n") . lines
```

### QuickCheck properties

Haddock (since version 2.13.0) has markup support for properties.  Doctest can
verify properties with QuickCheck.  A simple property looks like this:

```haskell
-- |
-- prop> \xs -> sort xs == (sort . sort) (xs :: [Int])
```

The lambda abstraction is optional and can be omitted:

```haskell
-- |
-- prop> sort xs == (sort . sort) (xs :: [Int])
```

A complete example that uses setup code is below:

```haskell
module Fib where

-- $setup
-- >>> import Control.Applicative
-- >>> import Test.QuickCheck
-- >>> newtype Small = Small Int deriving Show
-- >>> instance Arbitrary Small where arbitrary = Small . (`mod` 10) <$> arbitrary

-- | Compute Fibonacci numbers
--
-- The following property holds:
--
-- prop> \(Small n) -> fib n == fib (n + 2) - fib (n + 1)
fib :: Int -> Int
fib 0 = 0
fib 1 = 1
fib n = fib (n - 1) + fib (n - 2)
```

### Hiding examples from Haddock

You can put examples into [named chunks] [named-chunks], and not refer to them
in the export list.  That way they will not be part of the generated Haddock
documentation, but Doctest will still find them.

[named-chunks]: http://www.haskell.org/haddock/doc/html/ch03s05.html

### Using GHC extensions

The easiest way to tell Doctest about GHC extensions is to use [LANGUAGE
pragmas] [language-pragma] in your source files.

Alternatively you can pass any GHC options to Doctest, e.g.:

    doctest -XCPP Foo.hs

[language-pragma]: http://www.haskell.org/ghc/docs/latest/html/users_guide/pragmas.html#language-pragma

#### OverloadedStrings example

It should be noted that Doctest behaves like ghci.
Let's say you want to use the [OverloadedStrings] [overloaded-strings]
[LANGUAGE pragma] [language-pragma]. In this case, the [LANGUAGE pragmas] [language-pragma]
allows you to use [OverloadedStrings] [overloaded-strings] in the source file.
If you want to use them in examples, too, you have to explicitly say so.

There are three ways to deal with this:

1.  Pass ```-XOverloadedStrings``` to doctest

2.  Make it part of your example

    ```Haskell
    -- | Xpto function
    --
    -- This is meant to be used with GHC's `OverloadedStrings` extension:
    --
    -- >>> :set -XOverloadedStrings
    --
    -- >>> xpto "what?"
    -- "what?:xpto!"
    xpto :: Text -> Text
    xpto = (<> ":xpto!")
    ```

3.  Putting it into a ```$setup``` hook

    ```Haskell
    -- $setup
    -- The code examples in this module require GHC's `OverloadedStrings`
    -- extension:
    --
    -- >>> :set -XOverloadedStrings

    -- | Xpto function
    -- >>> xpto "what?"
    -- "what?:xpto!"
    xpto :: Text -> Text
    xpto = (<> ":xpto!")
    ```

Note that a ```$setup``` hook is also a named chunk,
so you can refer to it in the module header
(that way making it part of the module documentation).

The third (3) option may be preferable, as it puts you in the
flexible position to show/hide the dependency of your code examples
on OverloadedStrings (as you deem fit).

[overloaded-strings]: http://www.haskell.org/ghc/docs/7.8.2/html/users_guide/type-class-extensions.html#overloaded-strings

### Cabal integration

Doctest provides both, an executable and a library.  The library exposes a
function `doctest` of type:

```haskell
doctest :: [String] -> IO ()
```

Doctest's own `main` is simply:

```haskell
main = getArgs >>= doctest
```

Consequently, it is possible to create a custom executable for a project, by
passing all command-line arguments that are required for that project to
`doctest`.  A simple example looks like this:

```haskell
-- file doctests.hs
import Test.DocTest
main = doctest ["-isrc", "src/Main.hs"]
```

And a corresponding Cabal test suite section like this:

    test-suite doctests
      type:          exitcode-stdio-1.0
      ghc-options:   -threaded
      main-is:       doctests.hs
      build-depends: base, doctest >= 0.8

## Development [![Build Status](https://secure.travis-ci.org/sol/doctest.png)](http://travis-ci.org/sol/doctest)

Join in at `#hspec` on freenode.

Discuss your ideas first, ideally by opening an issue on GitHub.

Add tests for new features, and make sure that the test suite passes with your
changes.

    cabal configure --enable-tests && cabal build && cabal test


## Contributors

 * Adam Vogt
 * Anders Persson
 * Ankit Ahuja
 * Edward Kmett
 * Hiroki Hattori
 * Joachim Breitner
 * João Cristóvão
 * Kazu Yamamoto
 * Levent Erkok
 * Matvey Aksenov
 * Michael Orlitzky
 * Michael Snoyman
 * Nick Smallbone
 * Sakari Jokinen
 * Simon Hengel
