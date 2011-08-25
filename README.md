---
title: shelltestrunner
---

shelltestrunner is a handy cross-platform tool for testing command-line
programs or arbitrary shell commands.  It reads simple declarative tests
specifying a command, some input, and the expected output, error output
and exit status.  Tests can be run selectively, in parallel, with a
timeout, in color, and/or with differences highlighted. Projects using it
include hledger, yesod, and berp. shelltestrunner is free software released under GPLv3+.

Simon Michael wrote and maintains shelltestrunner; I was inspired by John
Wiegley's ledger tests.  John Macfarlane, Bernie Pope and Trygve Laugstøl
have contributed code. The
[hackage page](http://hackage.haskell.org/package/shelltestrunner) shows
the libraries it relies on - most notably, Max Bolingbroke's
test-framework.

## Usage

### Getting started

 Your machine's packaging system may provide shelltestrunner; if not, get
 yourself a working [cabal](http://www.haskell.org/haskellwiki/Cabal-Install) and

    $ cabal install shelltestrunner

 shelltestrunner should build with ghc 6.10 or greater; unicode support
 requires ghc 6.12 or greater.  It has been tested on gnu/linux, mac and
 windows.

### Defining tests

 Tests are defined in test files (typically `tests/*.test`), one or more per file.
 A test requires at least a command and an expected exit status, so it can be as simple as:

    commandthatshouldsucceed
    >>>= 0

 Here's the full test format:

    # optional comments
    one-line shell command (required; indent to disable --with substitution)
    <<<
    0 or more lines of stdin input
    >>>
    0 or more lines of expected stdout output (or /regexp/ on the previous line)
    >>>2
    0 or more lines of expected stderr output (or /regexp/ on the previous line)
    >>>= 0 (or other expected numeric exit status, or /regexp/) (required)

 If a `/regexp/` pattern is used, a match anywhere in the output allows
 the test to pass. The regular expression syntax supported is that of
 [regex-tdfa](http://hackage.haskell.org/package/regex-tdfa).  You can put
 `!` before a /regexp/ or numeric exit status to negate the match.

 Here
 [are](http://joyful.com/repos/shelltestrunner/tests)
 [more](http://joyful.com/repos/hledger/tests)
 [real-world](https://github.com/yesodweb/yesod/blob/master/yesod/tests/scaffold.shelltest)
 [examples](https://github.com/bjpop/berp/tree/master/test/regression).

### Running tests

 Run `shelltest` with one or more test file or directory paths. A
 directory means "all files below it with the test suffix (default:
 .test)". Eg:

    $ shelltest example.test
    :example.test: [OK]
    
             Test Cases  Total      
     Passed  1           1          
     Failed  0           0          
     Total   1           1          

 Here are the options, and some notes:

    $ shelltest --help
    shelltest 1.1
    
    shelltest [OPTIONS] [TESTFILES|TESTDIRS]
    
    Common flags:
      -a --all              Show all output on failures, even if large
      -c --color            Show colored output if your terminal supports it
      -d --diff             Show expected vs. actual in diff format (implies -a)
      -x --exclude=STR      Exclude test files whose path contains STR
         --execdir          Run tests from within the test file's directory
         --extension=EXT    Filename suffix of test files (default: .test)
      -w --with=EXECUTABLE  Replace the first word of (unindented) test commands
         --debug            Show debug info, for troubleshooting
         --debug-parse      Show test file parsing info and stop
      -h --help-format      Display test format help
      -? --help             Display help message
      -V --version          Print version information
    
         -- TFOPTIONS       Set extra test-framework options like -j/--threads,
                            -t/--select-tests, -o/--timeout, --hide-successes.
                            Use -- --help for a list. Avoid spaces.

 Test commands normally run within your current directory; the `--execdir`
 option makes them run within the directory where they are defined.

 The `-w/--with` option replaces the first word of all commands with
 something else, which can be useful for testing alternate versions of a
 program. Commands which have been indented by one or more spaces will not
 be affected.

 The test-framework library provides additional options which you can
 specify after `--`. Run `shelltest -- --help` for a list. Here are some
 useful ones:

      -j NUMBER        --threads=NUMBER             number of threads to use to run tests
      -o NUMBER        --timeout=NUMBER             how many seconds a test should be run for before giving up, by default
      -t TEST-PATTERN  --select-tests=TEST-PATTERN  only tests that match at least one glob pattern given by an instance of this argument will be run
                       --hide-successes             hide sucessful tests, and only show failures

 Example: run all tests defined in or below the tests directory with
 "args" in their name, up to 8 in parallel, allowing no more than a second
 for each, showing only failures and the final summary.  Avoid spaces
 between flags and values here (use <span
 style="white-space:nowrap;">`-targs`</span> not <span
 style="white-space:nowrap;">`-t args`</span> ):

    $ shelltest tests -- -targs -j8 -o1 --hide
    
             Test Cases   Total
     Passed  2            2
     Failed  0            0
     Total   2            2

## Contributing

 You can get the latest code here:

    $ darcs get http://joyful.com/repos/shelltestrunner

 [browse code](http://joyful.com/darcsweb/darcsweb.cgi?r=shelltestrunner;a=headblob;f=/shelltest.hs) -
 [recent changes](http://joyful.com/darcsweb/darcsweb.cgi?r=shelltestrunner)

 <a href="https://www.wepay.com/donate/39988?ref=widget&utm_medium=widget&utm_campaign=donation"
    target="_blank" style="float:right;margin:0 1em;"
    ><img src="https://www.wepay.com/img/widgets/donate_with_wepay.png" alt="Donate with WePay" /></a>
 Patches and feedback are welcome:
 [chat me](irc://irc.freenode.net/#haskell) (`sm` on irc.freenode.net) or
 [email me](mailto:simon@joyful.com?subject=shelltestrunner).
 Support/enhancement requests are handled on a best-effort basis, or you can [hire me](http://joyful.com/) or
 [donate](https://www.wepay.com/donate/39988?utm_campaign=donation)!

## Release history

**1.1** (2011/8/25)

  * bump process dependency to allow building with GHC 7.2.1
  * new `-a/--all` flag shows all failure output without truncating

**1.0** (2011/7/23)

  * New home page/docs
  * The `>>>=` field is now required; you may need to add it to your existing tests
  * Input and expected output can now contain lines beginning with `#`
  * Multiple tests in a file may now have whitespace between them
  * The `-i/--implicit` option has been dropped
  * New `-d/--diff` option shows test failures as a unified diff when possible, including line numbers to help locate the problem
  * New `-x/--exclude` option skips certain test files (eg platform-specific ones)
  * Passing arguments through to test-framework is now more robust
  * Fixed: parsing could fail when input contained left angle brackets
  * Fixed: some test files generated an extra blank test at the end

**0.9** (2010/9/3)

  * show plain non-ansi output by default, add --color option
  * better handling of non-ascii test data. We assume that non-ascii file
    paths, command-line arguments etc. are UTF-8 encoded on unix systems
    (cf http://www.dwheeler.com/essays/fixing-unix-linux-filenames.html),
    and that GHC 6.12 or greater is used. Then:
    - non-ascii test file paths should render correctly, eg in failure messages
    - non-ascii test commands should run correctly
    - non-ascii expected output should match correctly
    - non-ascii regular expressions should match correctly. (Caveat: not
      thoroughly tested, this may break certain regexps, )
  * use regex-tdfa instead of pcre-light for better windows compatibility
    To avoid a memory leak in current regex-tdfa, only regular expressions
    up to 300 characters in size are supported. Also, DOTALL is no longer
    enabled and probably fewer regexp constructs are supported.  There are
    still issues on windows/wine but in theory this will help.
  * tighten up dependencies

**0.8** (2010/4/9)

  * rename executable to shelltest. The package might also be renamed at some point.
  * better built-in help
  * shell tests now include a full command line, making them more readable
    and self-contained. The --with option can be used to replace the first
    word with something else, unless the test command line begins with a
    space.
  * we also accept directory arguments, searching for test files below
    them, with two new options:
      --execdir        execute tested command in same directory as test file
      --extension=EXT  file extension of test files (default=.test)

**0.7** (2010/3/5)

  * more robust parsing
    - --debug-parse parses test files and stops
    - regexps now support escaped forward slash (`\/`)
    - bad regexps now fail at startup
    - command-line arguments are required in a test, and may be blank
    - a >>>= is no longer required to separate multiple tests in a file
    - comments can be appended to delimiter lines
    - comments can appear at end of file
    - files need not have a final newline
    - files containing nothing, all comments, or valid tests are allowed; anything else is rejected
    - somewhat better errors
    - allow indented input
  * support negative (-) and negatively-matched (!) numeric exit codes
  * let . in regexps match newline
  * warn but continue when a test file fails to parse
  * output cleanups, trim large output
  * more flexible --implicit flag
  * switch to the more robust and faster pcre-light regexp lib

**0.6** (2009/7/15)

  * allow multiple tests per file, handle bad executable better

**0.5** (2009/7/14)

  * show failure output in proper order

**0.4** (2009/7/14)

  * run commands in a more robust way to avoid hangs
    This fixes hanging when a command generates large output, and hopefully
    all other deadlocks. The output is consumed strictly. Thanks to Ganesh
    Sittampalam for his help with this.
  * --implicit-tests flag providing implicit tests for omitted fields
  * --debug flag
  * regular expression matching
  * disallow interspersed foreign options which confused parseargs
  * change comment character to #

**0.3** (2009/7/11)

  * misc. bugfixes/improvements

**0.2** (2009/7/10)

  * bugfix, build with -threaded

**0.1** (2009/7/10)

  * shelltestrunner, a generic shell command stdout/stderr/exit status tester
