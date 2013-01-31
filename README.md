git-http-whitelist
==================

This script is meant to be used as a passthrough to git-http-backend. It is
meant for allowing anonymous git access via http, but completely disallowing
git modifications via http.

Requirements
------------

- gitolite
- gitweb
- nginx or apache

Installation
------------

Download and put the git-http-whitelist script somewhere and make it executable.

```
$ chmod +x /path/to/git-http-whitelist
```

Modify it to point it to the correct location of git-http-backend:

```
GIT_HTTP_BACKEND="/path/to/git-http-backend"
```

Assuming that gitolite and gitweb are properly set up, modify the virtualhost
like this (example given for apache):

```
<VirtualHost *:80>
    ServerName  git.example.org
    DocumentRoot /srv/gitweb

    # The gitolite repositories directory
    SetEnv GIT_PROJECT_ROOT /srv/git/repositories
    SetEnv GIT_HTTP_EXPORT_ALL

    # Modify to point to the script
    ScriptAliasMatch \
        "(?x)^/(.*/(HEAD | \
        info/refs | \
        objects/(info/[^/]+ | \
            [0-9a-f]{2}/[0-9a-f]{38} | \
            pack/pack-[0-9a-f]{40}\.(pack|idx)) | \
        git-(upload|receive)-pack))$" \
        /path/to/git-http-whitelist/$1        

    <Directory /srv/gitweb>
        Options ExecCGI
        AllowOverride None
        AddHandler cgi-script .cgi
        DirectoryIndex gitweb.cgi
        Order allow,deny
        Allow from all
    </Directory>
</VirtualHost>
```

Restart your web server, you should now be able to access any repositories that
are visible from gitweb:

```
$ git clone http://git.example.org/repository.git
```

License
-------

```
Copyright (c) 2013 Randy Thiemann

This software is provided 'as-is', without any express or implied
warranty. In no event will the authors be held liable for any damages
arising from the use of this software.

Permission is granted to anyone to use this software for any purpose,
including commercial applications, and to alter it and redistribute it
freely, subject to the following restrictions:

   1. The origin of this software must not be misrepresented; you must not
   claim that you wrote the original software. If you use this software
   in a product, an acknowledgment in the product documentation would be
   appreciated but is not required.

   2. Altered source versions must be plainly marked as such, and must not be
   misrepresented as being the original software.

   3. This notice may not be removed or altered from any source
   distribution.
```
