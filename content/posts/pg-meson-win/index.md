---
title: "Step-by-Step Guide to Compiling PostgreSQL on Windows using Chocolatey, Meson and Ninja"
date: 2025-01-12T21:12:02+05:00
draft: false

description: "Learn how to set up dependencies and compile PostgreSQL on Windows using Chocolatey, Strawberry Perl, and Meson. This step-by-step guide simplifies the process, ensuring smooth compilation with additional support for libraries like OpenSSL and libxml."

resources:
  - name: "featured-image"
    src: "featured-image.webp"

tags: ["PostgreSQL Compilation", "Chocolatey Setup", "Meson Ninja Build", "Open Source Databases", "PostgreSQL Windows Guide", "PostgreSQL Development"]
categories: ["Documentation"]
theme: "full"
images: ["https://theundersurfers.com/pg-meson-win/featured-image.webp"]
seo:
  images: ["https://theundersurfers.com/pg-meson-win/featured-image.webp"]
---

<!--more-->

## Dependencies

Use chocolatey for install dependencies for the postgres windows compilation.  Use strawberry perl instead of the active state perl as it is easy to setup. Also for know we will prefer the pkgconflite from the choco which is the lighter version of pkg-config and also simple for the setup. The pkg-config which comes with strawberry-perl doesn't work and get ignored.

```jsx
choco install winflexbison
choco install sed
choco install gzip
choco install strawberryperl
choco install diffutils
choco install pkgconflite

```

Now install meson and ninja. If you have python install you can use the pip manager otherwise just use the msi installer for the windows. https://mesonbuild.com/Getting-meson.html#installing-meson-and-ninja-with-the-msi-installer

```jsx
pip install meson ninja
```

Now you need some additional dependencies if you wanna compile the postgres with some additional  support like openssl, gsspapi etc. These dependencies is hrd to setup on perl but luckily Dave Page has organized a workflow for these dependencies build. You can just simply download the artifacts `all-deps-win64` from the repo. https://github.com/dpage/winpgbuild/actions/

## Compilation

Now open the visual studio native x64 developer command line. Set the variable for your installed dependencies first. And then use meson fro the compilation.

```jsx
SET PG_DEP=C:\Users\Imran\Desktop\work\pg\postgres\dep\all-deps-win64

-- use (--wipe) if you wanna reconfigure from scratch
meson setup --prefix=path\to\installation\dir -Dextra_include_dirs=%PG_DEP%\include -Dextra_lib_dirs=%PG_DEP%\lib\amd64 --cmake-prefix=%PG_DEP% --pkg-config-path=%PG_DEP%\lib\pkgconfig -Dgssapi=disabled

```
I just disabled the gssapi because there is a bug on postgres windows where gssapi and openssl cannot be compiled together. This will be fixed soon. For now just disable it.

{{< admonition type=tip title="This is a tip" open=false >}}
If you got stuck some where and want to reconfigure the meson setup from scratch use `—wipe` argument with the build dir as input.

```jsx
meson setup --wipe build
```

For advance setup related to assigning specific compilers to the meson you can use the native `ini` file. https://mesonbuild.com/Machine-files.html#
{{< /admonition >}}

## Installation

Now proceed towards the installation.

```jsx
cd build
ninja 
ninja install
```

Now as we have installed postgres with some additional libraries like openssl. We need to bundle the dependencies to the final installation.

```
SET PG_DEP=C:\Users\Imran\Desktop\work\pg\postgres\dep\all-deps-win64
SET PG_INSTALL=C:\Users\Imran\Desktop\work\pg\postgres\installs\pgmaster

copy %PG_DEP%\bin64\icuuc*.dll %PG_INSTALL%\bin
copy %PG_DEP%\bin64\icudt*.dll %PG_INSTALL%\bin
copy %PG_DEP%\bin64\icuin*.dll %PG_INSTALL%\bin
copy %PG_DEP%\bin\libiconv-2.dll %PG_INSTALL%\bin
copy %PG_DEP%\bin\libintl-8.dll %PG_INSTALL%\bin
copy %PG_DEP%\bin\libxml2.dll %PG_INSTALL%\bin
copy %PG_DEP%\bin\libxslt.dll %PG_INSTALL%\bin
copy %PG_DEP%\bin\libssl-*-x64.dll %PG_INSTALL%\bin
copy %PG_DEP%\bin\libcrypto-*-x64.dll %PG_INSTALL%\bin
copy %PG_DEP%\bin\liblz4.dll %PG_INSTALL%\bin
copy %PG_DEP%\bin\libzstd.dll %PG_INSTALL%\bin
copy %PG_DEP%\bin\zlib1.dll %PG_INSTALL%\bin
```

## Cluster Init

Now Initialize the cluster and enjoy the postgres.

```
C:\Users\Imran\Desktop\work\pg\postgres\installs\pgmaster>bin\initdb.exe data
The files belonging to this database system will be owned by user "Imran".
This user must also own the server process.

The database cluster will be initialized with locale "English_United States.1252".
The default database encoding has accordingly been set to "WIN1252".
The default text search configuration will be set to "english".

Data page checksums are enabled.

creating directory data ... ok
creating subdirectories ... ok
selecting dynamic shared memory implementation ... windows
selecting default "max_connections" ... 100
selecting default "autovacuum_worker_slots" ... 16
selecting default "shared_buffers" ... 128MB
selecting default time zone ... Asia/Karachi
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
syncing data to disk ... ok

initdb: warning: enabling "trust" authentication for local connections
initdb: hint: You can change this by editing pg_hba.conf or using the option -A, or --auth-local and --auth-host, the next time you run initdb.

Success. You can now start the database server using:

    ^"bin^\pg^_ctl^" -D data -l logfile start

C:\Users\Imran\Desktop\work\pg\postgres\installs\pgmaster>bin\pg_ctl -D data start
waiting for server to start....2025-01-12 21:53:05.764 PKT [19844] LOG:  starting PostgreSQL 18devel on x86_64-windows, compiled by msvc-19.42.34435, 64-bit
2025-01-12 21:53:05.774 PKT [19844] LOG:  listening on IPv6 address "::1", port 5432
2025-01-12 21:53:05.778 PKT [19844] LOG:  listening on IPv4 address "127.0.0.1", port 5432
2025-01-12 21:53:05.827 PKT [15960] LOG:  database system was shut down at 2025-01-12 21:50:46 PKT
2025-01-12 21:53:05.835 PKT [19844] LOG:  database system is ready to accept connections
 done
server started

C:\Users\Imran\Desktop\work\pg\postgres\installs\pgmaster>bin\psql postgres
psql (18devel)
WARNING: Console code page (437) differs from Windows code page (1252)
         8-bit characters might not work correctly. See psql reference
         page "Notes for Windows users" for details.
Type "help" for help.

postgres=# \conninfo
You are connected to database "postgres" as user "Imran" on host "localhost" (address "::1") at port "5432".
postgres=#
```

## References

- https://wiki.postgresql.org/wiki/Meson
- https://www.postgresql.org/docs/current/install-meson.html