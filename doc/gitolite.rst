Setting up git-multimail on gitolite
====================================

``git-multimail`` supports gitolite natively. Setting up
``git-multimail`` on a gitolite 3 installation can be done like this:

The explanations below show an easy way to set up ``git-multimail``,
but leave ``git-multimail`` installed and unconfigured for a while. If
you run gitolite on a production server, it is advised that you
execute the step "Set up the hook" last to avoid confusing your users
in the meantime.

Set up the hook
---------------

Log in as your gitolite user.

Create a file ``.gitolite/hooks/common/post-receive`` on your gitolite
account containing (adapt the path, obviously)::

  #!/bin/sh
  exec /path/to/git-multimail/git-multimail/git_multimail.py "$@"

Make sure it's executable (``chmod +x``). Record the hook in
gitolite::

  gitolite setup

Configuration
-------------

First, you have to allow the admin to set Git configuration variables.

As gitolite user, edit the line containing ``GIT_CONFIG_KEYS`` in file
``.gitolite.rc``, to make it look like::

  GIT_CONFIG_KEYS                 =>  'multimailhook\..*',

You can now log out and return to your normal user.

In the ``gitolite-admin`` clone, edit the file ``conf/gitolite.conf``
and add::

  repo @all
      # Not strictly needed as git_multimail.py will chose gitolite if
      # $GL_USER is set.
      config multimailhook.environment = gitolite
      config multimailhook.mailingList = # Where emails should be sent
      config multimailhook.from = # From address to use

Obviously, you can customize all parameters on a per-repository basis by
adding these ``config multimailhook.*`` lines in the section
corresponding to a repository or set of repositories.

To activate ``git-multimail`` on a per-repository basis, do not set
``multimailhook.mailingList`` in the ``@all`` section and set it only
for repositories for which you want ``git-multimail``.

Alternatively, you can set up the ``From:`` field on a per-user basis
by adding a ``BEGIN USER EMAILS``/``END USER EMAILS`` section (see
``../README``).

Troubleshooting
---------------

Warning: this will disable ``git-multimail`` during the debug, and
could confuse your users. Don't run on a production server.

To debug configuration issues with ``git-multimail``, you can add the
``--stdout`` option when calling ``git_multimail.py`` like this::

  #!/bin/sh
  exec /path/to/git-multimail/git-multimail/git_multimail.py --stdout "$@"

and try pushing from a test repository. You should see the source of
the email that would have been sent in the output of ``git push``.
