Mailfilt is a perl script that filters incoming mail based upon a set
of rules defined in a /.rules/ file.  The location of that file and
the mail directory in which to put messages are specified at the
beginning of Mailfilt itself.  The default directory is "$HOME/mail".

The /.rules/ file has a very rich syntax but is formatted like a makefile.
The following shows a sample /.rules/ file that illustrates many of the
features:

PGP
---

Mailfilt has built-in support for PGP signatures.  When it encounters a
PGP signed message, it will attempt to run /pgp/ to authenticate
the message.  For this to work properly, pgp must be in one of the
directories in the PATH statement in the configuration section of the
Mailfilt code and your public keyring must be in the directory pointed
to by the PGPPATH statement.  

The signature block will appear in the message as it was sent, but
following the END SIGNATURE marker, a couple lines will be inserted
showing whether /pgp/ could authenticate the message.

Users without pgp can use mailfilt without problem, except that an
error message will appear in the logfile each time a signed message is
received.

Example .rules file:
====================
    /dev/null:
        Subject: this is junk

    jimbob@the.farm:        #Forward this stuff to Jim Bob
        Subject: Farm report
        &!From: jimbob

    |/usr/bin/handle-cronmail:
        From: root
        &Subject: ^cron                #Subject must start with cron

    bulk:
        Rcpt: biglist@alma.matter #biglist must be in To: or Cc:

    personal:         #Stuff addressed to either of my mail aliases
        To: myname@big.fast.computer
        
        To: myothername@big.fast.computer

    primary:        #Anything that hasn't already matched will go to this file

In this example, mail with "this is junk" in the subject will go to
"/dev/null".  Mail with "Farm report" in the subject that doesn't come
from /jimbob/ will be mailed to /"jimbob@the.farm"/.  Mail from /root/
with /cron/ in the subject line will be put in the file /bulk/ in the
mail directory.  Mail that has /biglist@alma.matter/  in the To: or Cc:
lines will also be put in /bulk/.  Anything that hasn't met the above
criteria and that has /myname@big.fast.computer/ in the To: line will be
put in the file /personal/ in the mail directory.  Finally, everything
else will be put in the file /primary/.  This means that nothing will
end up in "/var/spool/mail".

Some general rules to remember
==============================

* The first rule that matches is the one used.
* Mail will only be forwarded once (to prevent looping)
* Searches are in regex format (substring unless special operators given)
* All matches are case-insensitive
* Rcpt: is the combination of To: and Cc:
* Filenames are in the mail directory unless they begin with a slash
* If a message doesn't match any rules, it will be placed in /var/spool/mail
* Logical ORs are accomplished using multiple sets of criteria under a
  single destination---see the "personal" block in the example above.

Grammer for .rules file:
========================
    FILE       ::=        {SET}

    SET        ::=        TARGET: {COMMENT} \n
                          {RULESET}
                          {SET}

    TARGET     ::=        FOLDER | ADDRESS | PIPE

    RULESET    ::=        WS RULE
                          WS ANDRULE
                          {SKIPLINE RULESET}

    ANDRULE    ::=        &RULE {ANDRULE}

    RULE       ::=        [!]HEADER: WS regex {COMMENT} \n

    HEADER     ::=        From | Subject | To | Rcpt | Date | ....

    ADDRESS    ::=        user@machine

    FOLDER     ::=        filename
               |          /path/to/file

    PIPE       ::=        |/path/to/program

    SKIPLINE   ::=        \n{COMMENT}\n

    WS         ::=         \t  {WS} 
               |          ` ' {WS}

    COMMENT    ::=        {WS}#string


Verision History
================

New Features in Version 1.3
---------------------------
* Bug fix: Pipe headers couldn't have whitespace

New Features in Version 1.21
----------------------------
* Bug fix: Headers spanning multiple lines were truncated
* Bug fix: <TT>cc</TT> header not included in <TT>Rcpt</TT>

New Features in Version 1.20
----------------------------
* PGP support
* Header matches are now case-insensitive

New Features in Version 1.11
----------------------------
* Better file locking for use with sendmail and mail agents
* Can save messages to pipes
* This page updated slightly
 
