Making Executable Commands on Windows
=====================================

Introduction
------------

The user interface for any program starts with the command you use to
execute it. For example, to start Python, you run the command ``python``.
But it's not as simple as that - the operating system needs to work out
what code to run, based on that command, and that's where the complications
start.

Because things are complicated, I'll give the punchline here. If you don't
want to read the gory details, feel free to just follow this advice. But if
you feel yourself thinking "but what about..." or "surely...", then please
*read the rest of this article*. You're probably wrong! There are a *lot* of
solutions out there that work 99% of the time - but that 1% can make life
really awkward for your users.

So, the advice:

.. note::
   Always create an ``exe`` file to run your main program.


How to create an exe
--------------------

The biggest reason people don't want to create an ``exe`` file for their
program is that it's hard. You really need a C compiler to build exes, and
most Windows users don't have that. Also, editing ``exe`` files means
editing the source and rebuilding. Not simple, especially compared to
a quick edit of your Python script (or whatever).

But typically, programming environments offer a means of building an
executable to "wrap" scripts. For Python, you can define setuptools
`entry points`_ in your ``setup.py`` script, and these will be converted
into ``exe`` files when you install your application.

For a more general solution, I'm in the process of developing `Shimmy`_, an
application to build executable "Shims" which can run any command you want.
It's a work in progress, but it is usable as it stands.

.. _entry points: https://pythonhosted.org/setuptools/setuptools.html#automatic-script-creation
.. _Shimmy: https://github.com/pfmoore/shimmy

Why must I use an exe?
----------------------

OK, so onto the meat of this article. Why do you *have* to use an executable?
The simple answer is that the Windows kernel treats ``exe`` files specially.
No other format of file is handled as a first-class citizen. There are lots
of APIs that deal with other forms of "executable", but they all end up needing
an ``exe`` in the end.

The fundamental Windows API for creating a new process is ``CreateProcess``. It
takes the filename of an executable, a command line[1]_, and a few other
parameters that don't really matter here. The filename can be omitted, if so
the command line is parsed for the first token, which is treated as the
filename.

The `documentation`_ for ``CreateProcess`` explains the details, but in essence
Windows only looks for files with the extension ``.exe`` when searching for a
command. You can specify the exact filename of the command, in which case you
can put any extension you like. But if you do, Windows will tell you the file
is not a valid Windows executable. So don't do that.

There's one exception - if you specify a Windows Batch file (``.bat`` or
``.cmd`` extension), Windows appears to run it successfully. But this isn't
documented - indeed, the documentation explicitly says

    To run a batch file, you must start the command interpreter;
    set lpApplicationName to cmd.exe and set lpCommandLine to the
    following arguments: /c plus the name of the batch file.

So not only is specifying a ``.bat`` file undocumented, it's actually
documented that you should use a *different* method of handling that case.
It works, but it's probably going to bite you at some stage. And anyway,
the search process doesn't try a ``.bat`` extension, so your users have to
explicitly include the extension when running your command, which probably
isn't what you want (and definitely isn't a portable approach).

.. _documentation: https://msdn.microsoft.com/en-us/library/windows/desktop/ms682425%28v=vs.85%29.aspx

Are there any other issues?
---------------------------

If that wasn't enough to make the point, here are some other issues commonly
seen when using alternatives to ``exe`` wrappers:

1. If you use a ``.bat`` file, you cannot call the command from another batch
   file. Calling one batch file from another transfers control (a "goto" rather
   than a subroutine call), which causes your original batch file to appear
   to terminate unexpectedly with no error. You have to use the "CALL" batch
   file command to call one batch file from another. And you (or one of your
   users) will probably forget...
2. If you interrupt a batch file with Ctrl-C you get the dreadful "Terminate
   batch job (Y/N)?" prompt. This doesn't always happen, but do you really
   want your users to see that?
3. If you use a ``.py`` file (or similar) and rely on ``PATHEXT`` to make the
   shell know your file is executable, it will work fine in a command shell
   (cmd or powershell) but will fail when someone tries to call your command
   from ``subprocess.Popen``, or task scheduler, or any other means that uses
   ``CreateProcess`` directly.

But I still don't want to go to the trouble of using an exe!
------------------------------------------------------------

Well, honestly, that's fine. If it's a script for your own use, or maybe for
your project, team, or company, then go for it. As long as you understand the
limitations, using ``PATHEXT`` to make ``.py`` files executable is a fine
solution. And some people even think ``.bat`` files are a pretty neat thing ;-)

But be very careful before distributing your code to the wider world. Someone
out there will need to use it in a context where nothing but an ``exe``
suffices. And they'll blame you. You can point them to workarounds like Shimmy
(mentioned above) but why not just provide something robust in the first place?

Some final notes
----------------

This article has been pretty hard-line. Sorry for that, but I've spent a lot
of time debating this issue on various mailing lists, and I started as one
of the people arguing against needing ``exe`` files. I was persuaded by many
people coming up with examples of issues with basically every solution I
proposed. So the advice here is pretty well battle-tested.

In the Python world, the issue of executable scripts was a huge topic of debate
in the early days of disturtils. Even when setuptools introduced the idea of
console entry points, it wasn't universally accepted at first. So plenty of
older projects still use other solutions (the ``scripts`` argument in
distutils' ``setup()`` function). Don't hate them for it - often such projects
were spending time maintaining Windows batch files when they didn't use Windows
themselves, for which they should be commended - but feel free to suggest to
them that it's probably time that they moved over to setuptools entry points.

.. rubric:: Footnotes

.. [1] Unix programmers take note, ``CreateProcess`` does *not* take a list of
       arguments, and much of the fun in cross-platform handling of
       subprocesses comes from hiding this fundamental difference between the
       two operating systems.
