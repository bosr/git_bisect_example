# Simple example of `git bisect`
Having the following files and history:

    * 0fb3944 - (27 minutes ago) add file6 - Romain Bossart (HEAD, master)
    * 45e33af - (28 minutes ago) add file5 - Romain Bossart
    * a22e57a - (49 minutes ago) add file4 - Romain Bossart
    * 7c034d3 - (47 minutes ago) add file3 - Romain Bossart
    * 035c49b - (48 minutes ago) add file2 - Romain Bossart
    * 5e67772 - (51 minutes ago) add file - Romain Bossart

The contents of the files are simply: 1 in `file`, 2 in `file2`, 3 in `file3`, and so on.

We want to find in which commit the bug was introduced. Here we simply assume that having a `3` is a bug.

## Add a test script
We must write an executable test script: `test-error.sh`:

    #!/usr/bin/env zsh
    if [[ $(ack 3 *.txt | xargs echo) != "" ]]; then echo 'found'; exit 1; fi

As stated in the [git-scm Book](http://git-scm.com/book/en/Git-Tools-Debugging-with-Git), the test script must return 0 on success and 1 to 127 (except 125) for failure.

## Run bisected bug hunt
We will look for the bug between two commits (which is the entire history in our pet case). Simply run:

    git bisect start HEAD 5e67

which means `HEAD` is the marked as bad and the first commit is marked as good. Then run the recursive bisection:

    git bisect run ./test-error.sh
    
Git bisects and runs the `test-error.sh` script at each step and stops when he spotted the last good commit and the last bad commit.

    ~/D/t/tutu git:bisect/bad~3:bisect ❯❯❯ git bisect run ./test-error.sh
    running ./test-error.sh
    found
    Bisecting: 0 revisions left to test after this (roughly 0 steps)
    [035c49b9e718dcf6550ea8a8129b522943063cac] add file2
    running ./test-error.sh
    7c034d34c895066db3489421bd1b9258a202cf97 is the first bad commit
    commit 7c034d34c895066db3489421bd1b9258a202cf97
    Author: Romain Bossart <romain.bossart@gmail.com>
    Date:   Thu May 1 14:25:09 2014 +0200

        add file3

    :000000 100644 0000000000000000000000000000000000000000 74c2610569349e12f2a4ceef7128190f73f653f0 A      file3.txt
    bisect run success
    ~/D/t/tutu git:bisect/good-035c49b9e718dcf6550ea8a8129b522943063cac:bisect ❯❯❯

Great.
