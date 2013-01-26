python-libmount
===============

A library for reading and manipulating filesystem tables, such as ``/etc/fstab``.

It uses ``ctypes`` to wrap ``libmount``, part of
`util-linux <http://userweb.kernel.org/~kzak/util-linux/>`_.


Usage
-----

Every reading and manipulation of the filesystem table should take place in a
``with`` block to take the lock::

    from libmount import FilesystemTable
    
    with FilesystemTable() as fstab:
        print fstab[0].target

A ``FilesystemTable`` acts like a list, so you can slice and iterate::

    with FilesystemTable() as fstab:
        print [fs.source for fs in fstab]
        print fs[2:5]

``FilesystemTable`` objects contain ``Filesystem`` objects, that each have
``source``, ``target``, ``fstab`` and ``options`` attributes. The first three
are strings, whereas the latter is ``set``-like::

    with FilesystemTable() as fstab:
        fs = fstab[0]
        
        # Will print e.g. "/dev/sda1 on / type ext4 (user_xattr)"
        print fs
        
        fs.source = '/dev/sda2'
        print "Options: %s" % ", ".join(fs.options)
        fs.options -= set(['user_xattr'])

To update the on-disk filesystem table, call ``save()``::

    with FilesystemTable() as fstab:
        for fs in fstab:
            if fs.fstype in ('ext3', 'ext4'):
                fs.options.add('user_xattr')
        fstab.save()

You can get UUID of drive by using ``get_uuid()`` method, like this::

    with FilesystemTable() as fstab:
        for fs in fstab:
            print "%s = %s" % (fs.source, fs.get_uuid())

Getting proper results from get_uuid() requires root access on some systems. If drive does
not have UUID, or you don't have permissions to read it, get_uuid() will return empty string.
UUID detection requires libblkid installed in your system.


Limitations
-----------

It is not yet possible to add or remove entries. This has not been thoroughly
tested when run by non-privileged users.


Feedback
--------

Feedback is gratefully received to `infodev@oucs.ox.ac.uk <mailto:infodev@oucs.ox.ac.uk>`_,
or as an issue in `the issue tracker <https://github.com/oucs/python-libmount/issues>`_.

