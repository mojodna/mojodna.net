---
layout: post
title: "The OS X Spatial Stack"
---

# The OS X Spatial Stack

I've been doing a bunch of spatial analysis and exploration recently. Where I
previously would have used a Debian or Ubuntu server (or VM) to experiment, I'm
now using Snow Leopard exclusively.

However, it's not all roses. As with most things, there are many paths to
installing and configuring software, not all of them complementary. What
follows is **my** experience installing and configuring various tools and their
dependencies. I've attempted to install as few duplicate dependencies as
possible (preferring Apple-provided versions) and to simplify the overall
experience in order that indvidual elements may be upgraded independently.

<!--
## Virtualizing Snow Leopard

Writing instructions for system configuration is difficult when you're not
starting from a fresh slate, as you may inadvertently omit something that was
installed in another way. Virtualization is often a good solution for this, but
virtualizing OS X isn't as easy (or as legal) as it could be.

I did some searching around and found a few tricks for getting Leopard running
under VMware Fusion. These instructions were condensed from
http://blog.rectalogic.com/2008/08/virtualizing-mac-os-x-leopard-client.html
and were sufficient to get a running Snow Leopard VM under Fusion 2.0.5.

{% highlight bash %}
# cd /Library/Application\ Support/VMware\ Fusion/isoimages/
# mkdir original
# mv darwin.iso tools-key.pub *.sig  original/
# sed "s/ServerVersion.plist/System.plist/g " < original/darwin.iso > darwin.iso
# openssl genrsa -out tools-priv.pem  2048
  Generating RSA private key, 2048 bit long modulus
  .........................+++
  .................................................................+++
  e is 65537 (0x10001)

# openssl rsa -in tools-priv.pem -pubout -pubout -out tools-key.pub
writing RSA key

# for A in *.iso ; do openssl dgst -sha1 -sign tools-priv.pem < $A > $A.sig ; done
{% endhighlight %}

Unfortunately, upgrading to 10.6.2 consistently causes the VM to crash during
boot. I was hopeful that Fusion 3 would address this problem (since it claims
to have better support for Snow Leopard (Server) guests), but the 10.6.0 VM
won't boot at all (nor will it install using the voodoo above).
-->

## Xcode

First, [download and install Xcode
3.2.1](https://connect.apple.com/cgi-bin/WebObjects/MemberSite.woa/wa/getSoftware?bundleID=20505).
The defaults are fine.

## Homebrew

Next, install [homebrew](http://github.com/mxcl/homebrew). I've found that
homebrew is significantly lighter and easier than
[MacPorts](http://macports.org/) for installing additional open source
software, as it re-uses system versions of libraries wherever possible.

Part of the homebrew philosophy is:

> OS X is designed to minimise sudo use, you only need it for real root-level
> stuff. You know your /System and /usr are as clean and pure as the day you
> bought your Mac because you didn't sudo. Sleep better at night! ... Lets face
> it; Homebrew is not installing anything system-critical. Apple already did
> that.

Homebrew is typically installed in `/usr/local`, so:

{% highlight bash %}
$ sudo mkdir -p /usr/local
$ sudo chown -R `whoami` /usr/local
{% endhighlight %}

(If you already had MySQL installed in `/usr/local`, fix it: `sudo chown -R
mysql:mysql /usr/local/mysql`.)

Now, install homebrew:

{% highlight bash %}
$ cd /usr/local
$ curl -L http://github.com/mxcl/homebrew/tarball/master | tar xz --strip 1 -C .
{% endhighlight %}

## Homebrew-provided Libraries

While we're working with homebrew, let's install some dependencies that we'll
need later.

First, [`boost`](http://www.boost.org/), a set of C++ libraries which are
necessary to compile Mapnik:

{% highlight bash %}
$ brew install boost
{% endhighlight %}

This step will probably take the longest, so skip ahead and start downloading
frameworks and software from the [KyngChaos Wiki](http://www.kyngchaos.com/).

Next, [`icu`](http://site.icu-project.org/), a set of Unicode libraries for C++
and Mapnik's other homebrew-provided dependency:

{% highlight bash %}
$ brew install icu4c
{% endhighlight %}

Finally, [`pkg-config`](http://pkg-config.freedesktop.org/) is necessary to
cleanly compile PIL:

{% highlight bash %}
$ brew install pkg-config
{% endhighlight %}

You'll want to make sure that `/usr/local/bin` is in your `$PATH`.

## KyngChaos Frameworks and Binaries

The [KyngChaos Wiki](http://www.kyngchaos.com/) is the holy grail for OS X
spatial downloads, as everything is cleanly packaged, up-to-date, and very well
organized.

From the [Unix Compatibility
Frameworks](http://www.kyngchaos.com/software:frameworks) page, you'll want to
download and install:

* **FreeType** 
* **GDAL Complete**--(this includes UnixImageIO (notably `jpeg` and `libpng`), PROJ,
  GEOS, SQLite3 (including Spatialite), and GDAL)

These will install to `/Library/Frameworks` and will set up pointers to their
corresponding Python packages in `/Library/Python/2.6/site-packages/`.

From the [Qgis](http://www.kyngchaos.com/software:qgis) page, you'll want the
most up-to-date Qgis installer (non-standalone).

Note: Qgis will run in 32-bit mode on Snow Leopard, as the QT framework is not
yet 64-bit. This means that Python Qgis plugins will call 32-bit Python
libraries (which may not exist, depending how they were installed). This bit me
when trying to use
[Quantumnik](http://bitbucket.org/springmeyer/quantumnik/wiki/Home), as I
failed to build a 32-bit version of Mapnik. If you get this working, please
post a comment.

From the [PostgreSQL](http://www.kyngchaos.com/software:postgres) page,
download and install:

* **PostgreSQL 8.4** (server + client)
* **PostGIS 1.4**

## Mapnik

By now, `boost` should have finished compiling, so you should install the
remaining homebrew-provided libraries.

From [mapnik.org](http://mapnik.org/), download the current version of the
Mapnik source and extract it to `/usr/local/src`:

{% highlight bash %}
$ mkdir -p /usr/local/src
$ cd /usr/local/src
$ tar zxf ~/Downloads/mapnik-0.6.1.tar.bz2
$ cd mapnik-0.6.1
{% endhighlight %}

Paste the following into `config.py`:

{% highlight python %}
OPTIMIZATION = '3'
INPUT_PLUGINS = 'all'
BOOST_INCLUDES = '/usr/local/include/boost'
BOOST_LIBS = '/usr/local/lib'
FREETYPE_CONFIG = '/Library/Frameworks/FreeType.framework/unix/bin/freetype-config'
PNG_INCLUDES = '/Library/Frameworks/UnixImageIO.framework/unix/include'
PNG_LIBS = '/Library/Frameworks/UnixImageIO.framework/unix/lib'
JPEG_INCLUDES = '/Library/Frameworks/UnixImageIO.framework/unix/include'
JPEG_LIBS = '/Library/Frameworks/UnixImageIO.framework/unix/lib'
TIFF_INCLUDES = '/Library/Frameworks/UnixImageIO.framework/unix/include'
TIFF_LIBS = '/Library/Frameworks/UnixImageIO.framework/unix/lib'
PROJ_INCLUDES = '/Library/Frameworks/PROJ.framework/unix/include'
PROJ_LIBS = '/Library/Frameworks/PROJ.framework/unix/lib'
GDAL_CONFIG = '/Library/Frameworks/GDAL.framework/unix/bin/gdal-config'
PG_CONFIG = '/usr/local/pgsql/bin/pg_config'
SQLITE_INCLUDES = '/Library/Frameworks/SQLite3.framework/unix/include'
SQLITE_LIBS = '/Library/Frameworks/SQLite3.framework/unix/lib'
FRAMEWORK_SEARCH_PATH = '/System/Library/Frameworks/Python.framework/Versions/2.6/'
BINDINGS = 'all'
JOBS = 8
{% endhighlight %}

Then, configure, build, and install Mapnik:

{% highlight bash %}
$ python scons/scons.py configure
$ python scons/scons.py install
{% endhighlight %}

## Python

We'll start with the easy stuff and `easy_install` the following:

* Flickr.API - Flickr has good geo data (I'm biased)
* [IPython](http://ipython.scipy.org/) - Interactive Python
* [nik2img](http://code.google.com/p/mapnik-utils/wiki/Nik2Img) - Mapnik
  command-line utility.
* [Nikweb](http://code.google.com/p/mapnik-utils/wiki/Nikweb) - Mapnik-based
  "static maps API" (though it requires GeoJSON input to determine appropriate
  bounding boxes and additional features)
* [nose](http://code.google.com/p/python-nose/) - Unit testing framework
* numscons - scons for numpy/scipy--required for scipy installation
* readline - causes IPython to use readline instead of libedit, solving all
  sorts of problems
* [TileCache](http://tilecache.org/) - WMS-C compliant tile server

{% highlight bash %}
$ easy_install Flickr.API
$ easy_install ipython
$ easy_install nik2img
$ easy_install Nikweb
$ easy_install nose
$ easy_install numscons
$ easy_install readline
$ easy_install TileCache
{% endhighlight %}

### PIL

[PIL](http://www.pythonware.com/products/pil/) is the Python Imaging Library.
Download the current source kit and extract it to `/usr/local/src`:

{% highlight bash %}
$ cd /usr/local/src
$ tar zxf ~/Downloads/Imaging-1.1.6.tar.gz
$ cd Imaging-1.1.6/
{% endhighlight %}

Since you previously installed `pkg-config`, building it is straightforward:

{% highlight bash %}
$ python setup.py install
{% endhighlight %}

### matplotlib

[matplotlib](http://matplotlib.sourceforge.net/) is a plotting library for
Python. It can be used both programmatically and interactively.

Download the source distribution from SourceForge and extract it to
`/usr/local/src`:

{% highlight bash %}
$ cd /usr/local/src
$ tar zxf ~/Downloads/matplotlib-0.99.1.2.tar.gz
$ cd matplotlib-0.99.1.1/ # why yes, this is a mistake in the pkg
{% endhighlight %}

Again, this is straightforward:

{% highlight bash %}
$ python setup.py install
{% endhighlight %}

matplotlib includes the [Basemap
Toolkit](http://matplotlib.sourceforge.net/basemap/doc/html/), which makes it
easier to work with cartographic data.

Download the source distribution from
[SourceForge](http://sourceforge.net/projects/matplotlib/files/matplotlib-toolkits/)
and extract it to `/usr/local/src`. While you're at it, download `natgrid` from
the same place.

{% highlight bash %}
$ cd /usr/local/src
$ tar zxf ~/Downloads/basemap-0.99.4.tar.gz
$ tar zxf ~/Downloads/natgrid-0.1.tar.gz
{% endhighlight %}

Basemap needs to know where GEOS was installed:

{% highlight bash %}
$ cd basemap-0.99.4/
$ GEOS_DIR=/Library/Frameworks/GEOS.framework/unix python setup.py install
{% endhighlight %}

`natgrid` is straightforward:

{% highlight bash %}
$ cd natgrid-0.1/
$ python setup.py install
{% endhighlight %}

### numpy / scipy

[numpy and scipy](http://www.scipy.org/) are the Pythonic chainsaws for
scientific computing. numpy will mostly install out-of-the-box, but we'll
install it from source.

Before we can build them, we need to install a Fortran compiler from [AT&T
Research](http://r.research.att.com/tools/):
http://r.research.att.com/gfortran-4.2.3.dmg

Download source distributions of numpy and scipy from
SourceForge([numpy](http://sourceforge.net/projects/numpy/files/),
[scipy](http://sourceforge.net/projects/scipy/files/)) and extract them to
`/usr/local/src`:

{% highlight bash %}
$ cd /usr/local/src
$ tar zxf ~/Downloads/numpy-1.3.0.tar.gz
$ tar zxf ~/Downloads/scipy-0.7.1.tar.gz
{% endhighlight %}

Build and install numpy:

{% highlight bash %}
$ cd numpy-1.3.0/
$ LDFLAGS="-lgfortran -arch x86_64" FFLAGS="-arch x86_64" \
    python setup.py install
{% endhighlight %}

Build and install scipy:

{% highlight bash %}
$ cd scipy-0.7.1/
$ LDFLAGS="-lgfortran -arch x86_64" FFLAGS="-arch x86_64" \
    python setup.py install
{% endhighlight %}

Note: the GDAL installer from KyngChaos bundles a version of numpy in
`/Library/Frameworks/GDAL.framework/Versions/1.6/Python/site-packages/numpy/`,
so if you run into weirdness, it's probably safe to delete that one.

Note: scipy recommends installing `fftw` for fast Fourier transformations; I was
unable to get scipy to build when it was installed (using homebrew), so I'm
omitting that step.

### Cascadenik

[Cascadenik](http://code.google.com/p/mapnik-utils/wiki/Cascadenik) is a
pre-processor that converts CSS-like syntax into Mapnik XML configuration
files. It supports handy shorthand such as "`.thing[zoom > 12] { ... }`" for
targeting standard slippy zoom levels and generally makes it easier to maintain
map styles. `nik2img` supports Cascadenik styling natively, making it easier to
create and tweak styles (otherwise you'll need to run `cascadenik-compile.py`
manually).

First, check out Cascadenik from Google Code:

{% highlight bash %}
$ cd /usr/local/src
$ svn co http://mapnik-utils.googlecode.com/svn/trunk/serverside/cascadenik
{% endhighlight %}

Next, build and install it:

{% highlight bash %}
$ cd cascadenik
$ python setup.py install
{% endhighlight %}

It will automatically download and install `cssutils` for you before making
`cascadenik-compile.py` and `cascadenik-style.py` available in your `$PATH`.

## OpenStreetMap

When working with data from [OpenStreetMap](http://openstreetmap.org/),
[`osm2pgsql`](http://wiki.openstreetmap.org/wiki/Osm2pgsql) and
[Osmosis](http://wiki.openstreetmap.org/wiki/Osmosis) are critical.

To install `osm2pgsql`, check it out from OSM's subversion repository and use
`make` to build and install it, linking against the KyngChaos framework:

{% highlight bash %}
$ cd /usr/local/src
$ svn co http://svn.openstreetmap.org/applications/utils/export/osm2pgsql/ 
$ cd osm2pgsql
$ PATH=$PATH:/Library/Frameworks/GEOS.framework/unix/bin/ \
    CFLAGS="-I/Library/Frameworks/PROJ.framework/unix/include" \
    LDFLAGS="-L/Library/Frameworks/PROJ.framework/unix/lib/" \
    make
{% endhighlight %}

You'll have to install it by hand to `/usr/local`:

{% highlight bash %}
$ install -m 0755 osm2pgsql /usr/local/bin
$ install default.style /usr/local/share/osm2pgsql
{% endhighlight %}

`default.style` is the standard map of OSM tags to database columns; you'll
need it when you run an import, even if the defaults are fine.

[Osmosis](http://wiki.openstreetmap.org/wiki/Osmosis) is a filter-driven tool
for processing large volumes of OSM XML.

To begin, download the [latest version of
Osmosis](http://dev.openstreetmap.org/~bretth/osmosis-build/osmosis-latest-bin.tar.gz).

Extract it to `/usr/local`:

{% highlight bash %}
$ cd /usr/local
$ tar zxf ~/Downloads/osmosis-latest-bin.tar.gz
{% endhighlight %}

Now, create a symlink to the binary:

{% highlight bash %}
$ ln -s /usr/local/osmosis-0.32/bin/osmosis /usr/local/bin/osmosis
{% endhighlight %}

## PostgreSQL Additions

At present, the KyngChaos distribution of PostgreSQL does not include the
[intarray](http://www.postgresql.org/docs/current/static/intarray.html) module
(you can check `/usr/local/pgsql/share/contrib` to see if it's been added).
`osm2pgsql` uses this module to create updatable Postgres OSM mirrors (meaning
that you can stay up-to-date by applying [planet
diffs](http://planet.openstreetmap.org/)).

This means that we need to build it ourselves. First, [download the source
tarball corresponding to the version you already have
installed](http://www.postgresql.org/ftp/source/v8.4.1/). Next, extract it to
`/usr/local/src`:

{% highlight bash %}
$ cd /usr/local/src
$ tar zxf ~/Downloads/postgresql-8.4.1.tar.bz2
{% endhighlight %}

Do a simplified build (referring to the [build notes for
PostgreSQL](http://www.kyngchaos.com/macosx:build:postgresql) for partial
consistency):

{% highlight bash %}
$ cd postgresql-8.4.1/
$ CFLAGS="-Os -D_FILE_OFFSET_BITS=64" \
    ./configure --with-openssl --with-pam --with-krb5 --with-ldap \
    --enable-thread-safety --with-bonjour --with-python --without-perl
$ make -j8
{% endhighlight %}

Note: Few of the build instructions are actually followed here; we just need
enough of the core compiled that `intarray` will build.

Now that `intarray` can be linked in a build environment, build it:

{% highlight bash %}
$ cd contrib/intarray
$ make
$ sudo make install
{% endhighlight %}

With `intarray` now installed, you can enable the database of your choosing
(`osm` in this case):

{% highlight bash %}
$ sudo -u postgres psql -d osm -f /usr/local/pgsql/share/contrib/_int.sql
{% endhighlight %}

## GDAL-based DEM Utilities

Until GDAL 1.7 is released, if you want to generate hillshades, slope and
aspect maps, or color reliefs, you'll need to install
[demtools](http://www.perrygeo.net/wordpress/?p=7). `demtools_osx.patch` is a
patch that makes it build on OS X:

{% highlight bash %}
$ cd /usr/local/src
$ svn co http://perrygeo.googlecode.com/svn/trunk/demtools/
$ cd demtools
$ curl -Ls http://bit.ly/6O2TIf | patch -p0
$ make
$ make install
{% endhighlight %}

## GUI Utilities

For graphical management of spatial data beyond what Qgis can handle, I usually
use [Base](http://menial.co.uk/software/base/) (for SQLite databases) and
[pgAdmin](http://www.pgadmin.org/) (for PostgreSQL databases).

## Congratulations!

You now have a working spatial stack on OS X! There are certainly things missed
here, but I'll try to keep this up-to-date as I discover new tools and simpler
methods (and in response to your comments, dear reader).
