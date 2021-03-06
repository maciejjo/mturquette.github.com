---
layout: post
title: Email Workflow for Champions
category: posts
---

![THIS IS EMAIL IN SPARTA!](/images/email-workflow-for-champions/this-is-sparta.jpg "THIS IS EMAIL IN SPARTA!")

I have edited this document several times while authoring it, always
trying to reduce its size. Unfortunately this is a big topic so it
merits a long article. I recommend brewing coffee before diving in.

There are four main parts: an explanation of my email work flow,
installing the requisite software, configuring the software and finally
using the software.

Finally, all of my configuration files for the software below are
available on [github](http://github.com/mturquette/ghar-email).

But what does it look like? Check out this poor quality screencast
showing what alot looks like with my configuration and theme:

<iframe width="560" height="315" src="http://www.youtube.com/embed/D0Yuc_25ySw" frameborder="0" allowfullscreen>
</iframe>

the workflow
============

-   fetch mail with [offlineimap](http://offlineimap.org/)
-   index mail with [notmuch](http://notmuchmail.org/)
-   tag mail with [afew](https://github.com/teythoon/afew/)
-   read mail with [alot](https://github.com/pazz/alot)

in human terms?
---------------

I pull email down from mail servers via offlineimap. My offlineimap
config dumps all mail in Maildir format to \$HOME/mail. Each IMAP
account gets its own directory: personal (my not-so-private gmail
account), linaro (my employer) and deferred (this site). I won't explain
how to install and configure offlineimap here since that topic is well
documented.

Notmuch indexes all new mail into a Xapian database after offlineimap
finishes syncing. This gives me fast and powerful search, similar to
Gmail. I can also tag mails which resembles Gmail's labels.

After notmuch is finished indexing the new mail afew is run. Afew
automatically tags the new mail using Bayesian magicks, similar to
Gmail's filters. The defaults work nicely without any extra
configuration and cover various things including spam filtering,
handling mailing lists and killed threads. Afew is not strictly
necessary, but I like it.

Alot is my mail reader; it was built from the ground up to work with
notmuch and it even plays nicely with afew out-of-the-box. It is not
strictly necessary to use alot, but I like it (sound familiar?). If you
want to use your existing MUA please check out the notmuch
[frontends](http://notmuchmail.org/frontends/).

installation
============

offlineimap
-----------

![Explain y in the comments.](/images/email-workflow-for-champions/y-u-no-listen.jpg "Explain y in the comments.")

notmuch
-------

### build from source

    $ git clone git://notmuchmail.org/git/notmuch
    $ cd notmuch
    $ ./setup build
    $ ./setup install --prefix=$HOME/.local

Python bindings are required by alot.

    $ cd bindings/python
    $ python setup build
    $ python setup install --prefix=$HOME/.local

If you have never installed libraries to
~/.local\\ then\\ add\\ the\\ following\\ to\\ your~/.bash\_profile.

    export LD_LIBRARY_PATH=~/.local/lib:$LD_LIBRARY_PATH

### installing from the package manager

    $ sudo apt-get install notmuch python-notmuch

afew
----

    $ git clone git://github.com/teythoon/afew.git
    $ cd afew
    $ ./setup build
    $ ./setup install --prefix $HOME/.local

alot
----

    $ sudo apt-get install python-configobj python-magic python-gpgme
    $ git clone git://github.com/pazz/alot.git
    $ cd alot
    $ ./setup build
    $ ./setup install --prefix $HOME/.local

Nice work! You have everything installed. Now to configure it for use.

configuration
=============

offlineimap
-----------

![Explain y in the comments. Again.](/images/email-workflow-for-champions/y-u-no-listen-again.jpg "Explain y in the comments. Again.")

notmuch
-------

Arguably the easiest to configure. My config can be viewed in my
[ghar-email](https://github.com/mturquette/ghar-email/blob/master/.notmuch-config)
project.

There are only four things that really need to be configured: the
location of the Maildir files, the user info (name and email addresses)
how to tag new mail and whether or not to synchronize local changes to
mail with the IMAP server. The last change makes sense for most IMAP
users. Perhaps less so if you are one of those POP luddites. There are
many different schemes for tagging new mail and some thoughts on that
can be found on the [notmuch
website](http://notmuchmail.org/initial_tagging/). If you intend to copy
my Email Workflow for Champions and use afew for tagging your mail then
I suggest you copy my scheme and simply tag new mail as "new".

afew
----

Afew's configuration is probably the most confusing due to the scarcity
of documentation (aka [Issue
\#23](https://github.com/teythoon/afew/issues/23)). For that reason I
copied the comments from the example config in the afew git repository
and merged them with the (largely unhelpful) examples from the project's
README. The result can also be found in my
[ghar-email](https://github.com/mturquette/ghar-email/blob/master/.config/afew/config)
repository.

Afew expects new mail to be tagged as "new". When run it uses notmuch to
find these mails in the Xapian database and tag them based on rules.
Everything in brackets that does *not* have punctuation or a number in
it is a **default** mail classification rule.

All of the \[Filter.x\] rules are personal rules that I set up. They are
run as part of the InboxFilter classification rule. Doubtless you might
want to customize these yourself. Notably afew's ListMailsFilter
classification rule does not catch all mails sent to lists, probably due
to some email clients populating headers incorrectly or something like
that. As such I set up redundant rules that match the style
ListMailsFilter. These rules strictly insure that mail received from a
mailing list subscription is tagged as such.

alot
----

Alot's configuration is as much a matter of personal preference as any
MUA configuration exercise. My
[config](https://github.com/mturquette/ghar-email/blob/master/.config/alot/config)
has some non-default bindings that one might find interesting. In
particular I became very accustomed to tapping 's' to star email from
the Gmail web interface, so I have bound that key to tag email as
flagged. Also of interest is the command I have bound to pipe; that
script is fully explained at the end of this post.

I use the solarized color scheme because I'm awesome and so should you
(assuming you that you too are awesome). I even [fixed
up](https://github.com/pazz/alot/pull/550) some of the colors in the
solarized\_dark scheme that ships with alot. <s>If you are interested in
using solarized\_dark I encourage you to try out my
[modifications](https://github.com/mturquette/ghar-email/blob/master/.config/alot/themes/solarized_dark.theme)
to the theme.</s> Those fixes to the solarized color scheme have been
merged into pazz's master branch. I find the theme's default layout to
be chaotic and messy, which I tried to clean up with judicious use of
whitespace and context-sensitive coloring. In particular flagged emails
show up as red, unread emails are yellow and everything else uses a nice
neutral default color.

using it
========

Using alot as your MUA is straight forward, but one must first fetch,
index and tag the mail to do so.

the cron job
------------

My cron script is for syncing, indexing and tagging new mail is on
[github](https://github.com/mturquette/ghar-email/blob/master/.local/bin/mail.cron),
but is reproduced below for the readers benefit. Put it in \~/.local/bin
and mark it executable.

bc.. \#!/bin/bash

PID=\$(pgrep offlineimap)\
LOCK=/var/lock/mail\
LOG=/home/mturquette/mail/receive.log\
export LD\_LIBRARY\_PATH=/home/mturquette/.local/lib:\$LD\_LIBRARY\_PATH

1.  offlineimap freezes up a lot. kill if previous job is still running\
    \[\[ -n "\$PID" \]\] && kill \$PID

<!-- -->

1.  sub-shell below is locked to prevent concurrency\
    (\
    \# bail early if lock is held\
    flock -n 9 || exit 1

\# timestamp the log and fetch new mail\
/bin/date &gt;&gt; \$LOG\
/usr/bin/offlineimap -o -a linaro -u basic &gt;&gt; \$LOG\
/home/mturquette/.local/bin/notmuch new &gt;&gt; \$LOG\
/home/mturquette/.local/bin/afew --tag --new --verbose 2&gt;&gt; \$LOG\
) 9&gt;/var/lock/mail

exit 0

Install the script with crontab. The below snippet will cause the script
to be run every 15 minutes.

    $ echo "`crontab -l`
    */15 * * * *  ~/.local/bin/mail.cron" | crontab -

You can check the log and view progress of the sync.

    $ less ~/mail/receive.log

If this is your initial sync from offlineimap then it will take a while.
Likewise the initial tag of your entire email archives with notmuch can
take a bit of time as well. Better brew another cup of coffee.

downloading patches
-------------------

As a Linux kernel developer I work with patches sent to mailing lists
extensively. Below is a horrible hack to download a patch from an open
email. The script scrapes the subject line and uses it to name the
patchfile which is then stored to \$HOME/src/linux/patches/incoming.

bc.. \#!/bin/bash\
PREFIX=\~/src/linux/patches/incoming

1.  save the mail to a temporary file\
    cat - | cat &gt; \$PREFIX/temp
2.  extract the subject and rename the file\
    mv \$PREFIX/temp "\$PREFIX/\`sed '/\^Subject: \*/!d; s///; s/ /-/g;
    s/\\//-/g; q' \$PREFIX/temp\`.patch"

I bound the pipe key to this function in \~/.config/alot/config. Using
pipe for this operation will fill you with joy and cause your friends to
think you are both clever and witty. I know this to be true.

    [[thread]]
        | = pipeto --format=raw ~/.local/bin/mail.print

viewing HTML emails
-------------------

If you are unlucky enough to need to view emails with embedded HTML then
you may need to install w3m for alot to display them properly. I have
found that this is unecessary for me with recent versions of alot, but
some might still benefit. For Debian-ish users:

    $ sudo apt-get install w3m

Do not forget to edit your `$HOME/.mailcap`.

Have fun with your Email Workflow for Champions™!
