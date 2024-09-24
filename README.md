# A C3 PCRE2 Library.

## Introduction

This [C3](https://c3-lang.org) library is a wrapper around the [PCRE2](https://github.com/PCRE2Project/pcre2) regular expression library.

## Status

Currently only the simple 4 function POSIX API is implemented. Ideally the library will be expanded to encompass all the functions that PCRE2 provides.

There are 64-bit PCRE2 DLL files in the `windows-x64` directory. These were built using [vcpkg](https://vcpkg.io/). The DLL files need to be placed where `PATH` picks it up, or in the same directory as the executable, etc., like is the case for any other program and DLL.

The `manifest.json` file currently only enables the linking of `pcre2-posix` for `windows-x64`, `linux-x64`, and `linux-aarch64` (because that is what I have access to at the moment and can verify works). More will be added as needed or requested (of course anyone can add what they need in a local copy).

## Usage

* Ensure the PCRE2 library is installed on your system (DLLs is included for x64 Windows in this wrapper library)
* Clone the repository into a directory mentioned in the `dependency-search-path` array in `project.json` (the default `"lib"` directory in the project for instance)
* Add `"pcre2"` to the `dependencies` array in `project.json`

## Example Code

```cpp
module playground;
import std::io;
import pcre2::posix;

fn void match(String s)
{
    io::printfn("Input [%s]", s);

    char[1024] errbuf;

    Regex_t re;
    char *pattern = "([a-z]+) ([a-z]+)? ([a-z]+)";
    CInt result = posix::regcomp(&re, pattern, posix::REG_ICASE);
    defer posix::regfree(&re);
    if (result != 0) {
        posix::regerror(result, &re, &errbuf, 1024);
        io::printfn("regcomp error: %s", (ZString)&errbuf);
        return;
    }

    Regmatch_t[4] matches;
    result = posix::regexec(&re, s, matches.len, &matches, 0);
    if (result != 0) {
        posix::regerror(result, &re, &errbuf, 1024);
        io::printfn("regexec error: %s", (ZString)&errbuf);
        return;
    }

    io::printfn("Matched string [%s]", s[matches[0].rm_so..matches[0].rm_eo-1]);
    foreach (i, match : matches[1..]) {
        if (match.rm_so == -1) continue;
        io::printfn("Match group %s [%s]", i+1, s[match.rm_so..match.rm_eo-1]);
    }
}

fn void main(String[] args)
{
    match("123 abc DEF ghi 456");
    io::printn();
    match("abc 123 ghi");
    io::printn();
    match("abc  ghi");
}
```
The output of this example code is
```
Input [123 abc DEF ghi 456]
Matched string [abc DEF ghi]
Match group 1 [abc]
Match group 2 [DEF]
Match group 3 [ghi]

Input [abc 123 ghi]
regexec error: match failed

Input [abc  ghi]
Matched string [abc  ghi]
Match group 1 [abc]
Match group 3 [ghi]
```

## PCRE2 POSIX API

### pcre2::regcomp

```
fn CInt regcomp(
    Regex_t *preg,
    char *pattern,
    CInt flags);
```
Compiles a regular expression into a `Regex_t` variable. Returns 0 on success, otherwise an error code that can be used with `pcre2::regerror`.

### pcre2::regexec

```
fn CInt regexec(
    Regex_t *preg,
    char *string,
    usz nmatch,
    Regmatch_t *pmatch,
    CInt eflags);
```
Matches a compiled regular expression with a string, and populates the array of potential match groups. Returns 0 on success, otherwise an error code that can be used with `pcre2::regerror`.

### pcre2::regerror

```
fn usz regerror(
    CInt errcode,
    Regex_t *preg,
    char *errbuf,
    usz errbuf_size);
```
Stores a string version of the PCRE2 error code in the provided buffer. The return value is the required size of a buffer to hold the whole message including the terminating zero. This value is greated than the passed `errbuf_size` argument if the buffer contains a truncated message.

### pcre2::regfree

```
fn void regfree(Regex_t *preg);
```
Frees a compiled regular expression.

## Constants

The constants below are used to influence the behavior of the regular expression matching and error handling. See the [PCRE2](https://pcre2project.github.io/pcre2/doc/html/pcre2posix.html) documentation for the details and implications of them.

### Compile and Execution Flags

```
const CInt REG_ICASE    = 0x0001;  /* Maps to PCRE2_CASELESS */
const CInt REG_NEWLINE  = 0x0002;  /* Maps to PCRE2_MULTILINE */
const CInt REG_NOTBOL   = 0x0004;  /* Maps to PCRE2_NOTBOL */
const CInt REG_NOTEOL   = 0x0008;  /* Maps to PCRE2_NOTEOL */
const CInt REG_DOTALL   = 0x0010;  /* NOT defined by POSIX; maps to PCRE2_DOTALL */
const CInt REG_NOSUB    = 0x0020;  /* Do not report what was matched */
const CInt REG_UTF      = 0x0040;  /* NOT defined by POSIX; maps to PCRE2_UTF */
const CInt REG_STARTEND = 0x0080;  /* BSD feature: pass subject string by so,eo */
const CInt REG_NOTEMPTY = 0x0100;  /* NOT defined by POSIX; maps to PCRE2_NOTEMPTY */
const CInt REG_UNGREEDY = 0x0200;  /* NOT defined by POSIX; maps to PCRE2_UNGREEDY */
const CInt REG_UCP      = 0x0400;  /* NOT defined by POSIX; maps to PCRE2_UCP */
const CInt REG_PEND     = 0x0800;  /* GNU feature: pass end pattern by re_endp */
const CInt REG_NOSPEC   = 0x1000;  /* Maps to PCRE2_LITERAL */
```

### Error Codes
```
const CInt REG_ASSERT   =  1;  /* internal error ? */
const CInt REG_BADBR    =  2;  /* invalid repeat counts in {} */
const CInt REG_BADPAT   =  3;  /* pattern error */
const CInt REG_BADRPT   =  4;  /* ? * + invalid */
const CInt REG_EBRACE   =  5;  /* unbalanced {} */
const CInt REG_EBRACK   =  6;  /* unbalanced [] */
const CInt REG_ECOLLATE =  7;  /* collation error - not relevant */
const CInt REG_ECTYPE   =  8;  /* bad class */
const CInt REG_EESCAPE  =  9;  /* bad escape sequence */
const CInt REG_EMPTY    = 10;  /* empty expression */
const CInt REG_EPAREN   = 11;  /* unbalanced () */
const CInt REG_ERANGE   = 12;  /* bad range inside [] */
const CInt REG_ESIZE    = 13;  /* expression too big */
const CInt REG_ESPACE   = 14;  /* failed to get memory */
const CInt REG_ESUBREG  = 15;  /* bad back reference */
const CInt REG_INVARG   = 16;  /* bad argument */
const CInt REG_NOMATCH  = 17;  /* match failed */
```
## License

This library is released under the MIT license.