# FastCGI Fano Web Framework Skeleton Application

FastCGI web application skeleton using Fano Framework, Pascal web application framework. It uses Apache mod_fcgid module.

This project is generated using [Fano CLI](https://github.com/fanoframework/fano-cli)
command line tools to help scaffolding web application using Fano Framework.

## Requirement

- Linux or FreeBSD
- [Free Pascal](https://www.freepascal.org/) >= 3.0
- Apache with [mod_fcgid](https://httpd.apache.org/mod_fcgid/mod/mod_fcgid.html)
- [Fano CLI](https://github.com/fanoframework/fano-cli)
- Administrative privilege for virtual host setup

## Installation

### TL;DR
Make sure all requirements above are met. Run

```
$ git clone https://github.com:fanofamework/fano-fcgid.git --recursive
$ cd fano-fcgid
$ ./tools/config.setup.sh
$ ./build.sh
$ sudo fanocli --deploy-fcgid=hello.fano
```
Open browser and go to `http://hello.fano`, you should see application.

### Build

Make sure [Free Pascal](https://www.freepascal.org/) is installed. Run

    $ fpc -i

If you see something like `Free Pascal Compiler version 3.0.4`,  you are good to go.

Clone this repository

    $ git clone https://github.com:fanofamework/fano-fcgid.git --recursive

`--recursive` is needed so git also pull [Fano](https://github.com/fanoframework/fano) repository.

If you are missing `--recursive` when you clone, you may find that `vendor/fano` directory is empty. In this case run

    $ git submodule update --init

To update Fano to its latest commit, run

    $ git checkout master && git submodule foreach --recursive git pull origin master

Above command will checkout to `master` branch of this repository and pull latest update from `master` branch of [Fano](https://github.com/fanoframework/fano) repository.

Copy `*.cfg.sample` to `*.cfg`.
Make adjustment as you need in `build.cfg`, `build.prod.cfg`, `build.dev.cfg`
and run `build.sh` shell script (if you are on Windows, then `build.cmd`).

These `*.cfg` files contain some Free Pascal compiler switches that you can turn on/off to change how executable is compiled and generated. For complete
explanation on available compiler switches, consult Free Pascal documentation.

Also copy `src/config/config.json.sample` to `src/config/config.json` and edit
configuration as needed. For example, you may need to change `baseUrl` to match your own base url so JavaScript or CSS stylesheets point to correct URL.

    $ cp config/config.json.sample config/config.json
    $ cp build.prod.cfg.sample build.prod.cfg
    $ cp build.dev.cfg.sample build.dev.cfg
    $ cp build.cfg.sample build.cfg
    $ ./build.sh

`tools/config.setup.sh` shell script is provided to simplify copying those
configuration files. Following shell command is similar to command above.

    $ ./tools/config.setup.sh
    $ ./build.sh

By default, it will output binary executable in `public` directory.

### Build for different environment

To build for different environment, set `BUILD_TYPE` environment variable.

#### Build for production environment

    $ BUILD_TYPE=prod ./build.sh

Build process will use compiler configuration defined in `vendor/fano/fano.cfg`, `build.cfg` and `build.prod.cfg`. By default, `build.prod.cfg` contains some compiler switches that will aggressively optimize executable both in speed and size.

#### Build for development environment

    $ BUILD_TYPE=dev ./build.sh

Build process will use compiler configuration defined in `vendor/fano/fano.cfg`, `build.cfg` and `build.dev.cfg`.

If `BUILD_TYPE` environment variable is not set, production environment will be assumed.

## Change executable output directory

Compilation will output executable to directory defined in `EXEC_OUTPUT_DIR`
environment variable. By default is `public` directory.

    $ EXEC_OUTPUT_DIR=/path/to/public/dir ./build.sh

## Change executable name

Compilation will use executable filename as defined in `EXEC_OUTPUT_NAME`
environment variable. By default is `app.cgi` filename.

    $ EXEC_OUTPUT_NAME=index.cgi ./build.sh

## Run

### Run with a webserver

Setup a virtual host. Please consult documentation of web server you use.

For example on Apache,

```
<VirtualHost *:80>
     ServerName www.example.com
     DocumentRoot /home/example/public

     <Directory "/home/example/public">
         Options +ExecCGI
         AllowOverride FileInfo
         Require all granted
         DirectoryIndex app.cgi
         AddHandler fcgid-script .cgi
     </Directory>
</VirtualHost>
```
On Apache, you will need to enable `mod_fcgid`. If correct module not loaded, above virtual host will cause `app.cgi` is downloaded instead of executed.

For example, on Debian, this will install and enable `mod_fcgid` module.

```
$ sudo apt install libapache2-mod-fcgid
$ sudo a2enmod fcgid
$ sudo systemctl restart apache2
```

Depending on your server setup, for example, if  you use `.htaccess`, add following code:

```
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^(.*)$ app.cgi [L]
</IfModule>
```
and put `.htaccess` file in same directory as `app.cgi` file (i.e., in `public` directory).

Content of `.htaccess` basically tells Apache to serve existing files/directories directly. For any non-existing files/directories, pass them to our application.

## Deployment

You need to deploy only executable binary and any supporting files such as HTML templates, images, css stylesheets, application config.
Any `pas` or `inc` files or shell scripts is not needed in deployment machine in order application to run.

So for this repository, you will need to copy `public`, `Templates`, `config`
and `storages` directories to your deployment machine. make sure that
`storages` directory is writable by web server.

For more information [Deploy as FastCGI application](https://fanoframework.github.io/deployment/fastcgi/)

## Known Issues

### Issue with GNU Linker

When running `build.sh` script, you may encounter following warning:

```
/usr/bin/ld: warning: public/link.res contains output sections; did you forget -T?
```

This is known issue between Free Pascal and GNU Linker. See
[FAQ: link.res syntax error, or "did you forget -T?"](https://www.freepascal.org/faq.var#unix-ld219)

However, this warning is minor and can be ignored. It does not affect output executable.

### Issue with unsynchronized compiled unit with unit source

Sometime Free Pascal can not compile your code because, for example, you deleted a
unit source code (.pas) but old generated unit (.ppu, .o, .a files) still there
or when you switch between git branches. Solution is to remove those files.

By default, generated compiled units are in `bin/unit` directory.
But do not delete `README.md` file inside this directory, as it is not being ignored by git.

```
$ rm bin/unit/*.ppu
$ rm bin/unit/*.o
$ rm bin/unit/*.rsj
$ rm bin/unit/*.a
```

Following shell command will remove all files inside `bin/unit` directory except
`README.md` file.

    $ find bin/unit ! -name 'README.md' -type f -exec rm -f {} +

`tools/clean.sh` script is provided to simplify this task.

### Windows user

Free Pascal supports Windows as target operating system, however, this repository is not yet tested on Windows. To target Windows, in `build.cfg` replace
compiler switch `-Tlinux` with `-Twin64` and uncomment line `#-WC` to
become `-WC`.

### Lazarus user

While you can use Lazarus IDE, it is not mandatory tool. Any text editor for code editing (Atom, Visual Studio Code, Sublime, Vim etc) should suffice.
