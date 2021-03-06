Title: Fixing per-user web directories on Leopard
Date: 2007-11-17 17:55
Author: gmwils
Category: apple

I do most of my web testing and development on localhost/\~gmwils.
Unfortunately, the upgrade to Leopard made this very broken.

Apple kindly updated apache from v1 to v2, and as a result the
configuration directory moves from `/etc/httpd` to `/etc/apache2`. The
log directory also moves, from `/log/httpd` to `/log/apache2`.

[Mike][] isolated the problem:

> When a user is created on your system, a small Apache configuration
> file is created that enables Apache to serve content from their
> `~/Sites/` directory. Under Tiger, these files were stored in
> `/private/etc/httpd/users/`. From what I can tell, if you've done an
> upgrade from Tiger to Leopard, those files do not get migrated over to
> the new `/private/etc/apache2/users/` folder.
>
> So, in order to make your sites work again, make sure to copy your
> Apache per-user configuration files from `/private/etc/httpd/users` to
> `/private/etc/apache2/users`.

  [Mike]: http://www.workerswithoutcubicles.net/node/7
