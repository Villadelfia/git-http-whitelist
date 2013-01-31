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
Copyright (c) 2013, Randy Thiemann
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:
    * Redistributions of source code must retain the above copyright
      notice, this list of conditions and the following disclaimer.
    * Redistributions in binary form must reproduce the above copyright
      notice, this list of conditions and the following disclaimer in the
      documentation and/or other materials provided with the distribution.
    * Neither the name of the git-http-whitelist project nor the
      names of its contributors may be used to endorse or promote products
      derived from this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL RANDY THIEMANN BE LIABLE FOR ANY
DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
(INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
(INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
```
