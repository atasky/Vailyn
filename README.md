<h1 align="center">
  <img src='misc/logo.png' height='580'></img><br>
  Vailyn
  <br>
</h1>

<p align="center">
  <a href="https://github.com/VainlyStrain/Vailyn/blob/master/Vailyn">
    <img src="https://img.shields.io/static/v1.svg?label=Version&message=2.0&color=lightgrey&style=flat-square"><!--&logo=dev.to&logoColor=white"-->
  </a>
  <a href="https://www.python.org/">
    <img src="https://img.shields.io/static/v1.svg?label=Python&message=3.7&color=lightgrey&style=flat-square&logo=python&logoColor=white">
  </a><br>
  Phased Path Traversal Attacks
</p>

### About

Vailyn is a multi-phased vulnerability analysis and exploitation tool for path traversal/directory climbing vulnerabilities. It is built to make it as performant as possible, and to offer a wide arsenal of filter evasion techniques.

### How does it work?

Vailyn operates in 2 phases. First, it checks if the vulnerability is present. It does so by trying to access /etc/passwd (or an user-specified file), with all of its evasive payloads. Analysing the response, payloads that worked are separated from the others.

Now, the user can choose freely which payloads to use. Only these payloads will be used in the second phase.

The second phase is the exploitation phase. Now, it tries to leak all possible files from the server using a file and a directory dictionary. The search depth and the directory permutation level can be adapted via arguments. Optionally, it can download found files, and save them in its loot folder.

Right now, it supports multiple attack methods: injection via query, path, query and post data.

### Why the phase separation?

The separation in several phases is new in this version. It is done to hugely improve the performance of the tool. In previous versions, every file-directory combination was checked with every payload. This resulted in a huge overhead due to payloads being always used again, despite they are not working for the current server.

### Installation

Recommended Python version is 3.7+, but it should work fine with prior Python 3 versions, too. To install Vailyn, download the archive from the release tab, or perform

```
$ git clone https://github.com/VainlyStrain/Vailyn
```

Once on your system, you'll need to install the dependencies.

```
$ pip install -r requirements.txt   # --user
```

That's it! Fire Vailyn up by moving to its installation directory and performing

```
$ python Vailyn -h
```

### Usage

Vailyn has 3 mandatory arguments: `-v VIC, -a ACK and -l FIL PATH`. However, depending on `-a`, more arguments may be required.

```
mandatory:
  -v VIC, --victim VIC  Target to attack, part 1 [pre injection point]
  -a ACK, --attack ACK  Attack type (int)[1: query, 2: path, 3:cookie]
  -l FIL PATH, --lists FIL PATH      
                        Dictionaries to use (see templates for syntax)
additional:
  -p PAM, --param PAM   query parameter to use for --attack 1
  -s DAT, --post DAT    POST Data (set injection point with INJECT)
  -d I J, --depths I J  depths of checking (I: phase 1, J: phase 2)
  -n, --loot            Download found files into the loot folder
  -c FIL, --cookie FIL  File containing authentication cookie (if needed)
  -h, --help            show this help menu and exit
  -i FIL, --check FIL   File to check for in Phase 1 (df: /etc/passwd)
  -q VIC2, --vic2 VIC2  Attack Target, part 2 (post injection point)
  -t, --tor             Pipe attacks through the Tor anonymity network
```

Vailyn currently supports 4 attack vectors. The attack performed is identified by the `-a ACK` argument.

```
ACK        attack
----       -------
1          query-based attack  (https://site.com?file=../../../)
2          path-based attack   (https://site.com/../../../)
3          cookie-based attack (will grab the cookies for you)
4          infected post data  (ELEM1=VAL1&ELEM2=../../../)
```

You also must specify a target to attack. This is done via `-v VIC` and `-q VIC2`, where -v is the part before the injection point, and -q the rest.

Example: if the final URL should look like: `https://site.com/download.php?file=<ATTACK>&param2=necessaryvalue`, you can specify `-v https://site.com/download.php` and `-q &param2=necessaryvalue` (and `-p file`, since this is a query attack).

To perform the bruteforce attack in phase 2, you need to specify 2 dictionaries:
* FIL, containing filenames only (e.g. index.php)
* PATH, containing directory names only. Note that each directory entry MUST end with a "/". Also, Vailyn will handle directory permutation for you, so you'll need only single directories in a line.

If the attacked site is behind a login page, you can supply an authentication cookie via `-c COOKIEFILE`. If you want to attack over Tor, use `--tor`.

#### Phase 1

This is the analysis phase, where working payloads are separated from the others.

By default, `/etc/passwd` is looked up. If the server is not running Linux, you can specify a custom file by `-i FILENAME`. Note that you must include subdirectories in FILENAME.
You can modify the lookup depth with the first value of `-d` (default=8).

#### Phase 2

This is the exploitation phase, where Vailyn will try to leak as much files as possible.

The depth of lookup in phase 2 (the maximal number of layers traversed back, and the level of subdirectory recursion) is specified by the second value of the `-d` argument. In future versions, these properties can be changed independently (using 2 arguments).

By specifying `-n`, Vailyn will not only display files on the terminal, but also download and save the files into the loot folder.

If you want a verbose output (display every output, not only found files), you can use `--debug`. Note that output gets really messy, this is basically just a debug help.

### False Positive prevention

To distinguish real results from false positives, Vailyn does the following checks:
* check the status code of the response
* check if the response is identical to one taken before attack start: this is useful e.g, when the server returns 200, but ignores the payload input or returns a default page if the file is not found.
* check for empty responses
* check if common error signatures are in the response content
* check if the payload is contained in the response: this is an additional check for the case the server responds 200 for non-existing files, and reflects the payload in a message like (../../secret not found)

### Demo

[![asciicast](https://asciinema.org/a/348613.svg)](https://asciinema.org/a/348613)

### Possible Issues

Please consider, that I'm still learning. So, there may still be some undetected false positives/negatives. If you found some (or want to point out other bugs/improvements), please leave an issue.

### Code of Conduct

> Vailyn is provided as an offensive web application audit tool. It has built-in functionalities which can reveal potential vulnerabilities in web applications, which could possibly be exploited maliciously.
>
> **THEREFORE, NEITHER THE AUTHOR NOR THE CONTRIBUTORS ARE RESPONSIBLE FOR ANY MISUSE OR DAMAGE DUE TO THIS TOOLKIT.**
>
> By using this software, the user obliges to follow their local laws, to not attack someone else's system without explicit permission from the owner, or with malicious intend.
>
> In case of an infringement, only the end user who committed it is accountable for their actions.
