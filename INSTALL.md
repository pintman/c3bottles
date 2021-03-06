# System Requirements

In order to install c3bottles, you will need:

*   Flask (python-flask)

*   Flask-SQLAlchemy (python-flask-sqlalchemy)

*   Flask-Login >= 0.3.0 (versions <0.3.0 will not work since a breaking
    change has been done in the API) (python-flask-login)

*   Flask-WTF (python-flaskext.wtf)

*   CairoSVG (python-cairosvg)

*   PyPDF2 (python-pypdf2)

*   Python-QRCode (python-qrcode)

*   a WSGI-capable webserver (e.g. Apache)

*   some SQL server supported by SQLAlchemy
    (the author uses PostgreSQL but others should work, too)

*   ImageMagick, GDAL and the gdal2tiles.py script to generate the map tiles
    (imagemagick, gdal-bin and python-gdal)

*   the Node.js package manager (npm)

*   on Debian, Node.js is fucked up, so the legacy symlink is needed for npm
    etc. (node-legacy)

# Installation

1.  Make sure all the dependencies are installed.
    On Debian using Apache, you can do:

        $ apt-get install python-flask python-flask-sqlalchemy \
                          python-flask-login python-flaskext.wtf \
                          python-cairosvg python-pypdf2 python-qrcode \
                          libapache2-mod-wsgi imagemagick \
                          gdal-bin python-gdal npm nodejs-legacy

2.  Copy the files into some directory readable by the web server.
    You can clone the repository from Github:

        $ git clone https://github.com/der-michik/c3bottles.git

3.  Fetch all the dependencies and build everything:

        $ npm install
        $ npm run build

4.  Create a configuriation file `config.py`. You will find a template for
    the configuration in the file `config.default.py`. c3bottles will not
    work if no config.py with the required settings is present. You have to
    configure at least a database URI and a secret key.

5.  Configure your database accordingly. The user for c3bottles needs full
    access to the database. If you use SQLite, the web server needs write
    access to the directory containing the `*.db` file and to the file itself.

6.  Initialize the database using the Python interpreter:

        $ cd /path/to/c3bottles
        $ python
        >>> from controller import db, load_config
        >>> load_config()
        >>> db.create_all()

7.  In order to use c3bottles, you need at least one admin user. You can
    create one like this:

        $ python
        >>> from controller import db, load_config
        >>> load_config()
        >>> db.session.add(User("admin", "password", True, True, True, False)
        >>> db.session.commit()

8.  Configure your webserver accordingly to run the WSGI application. Apache
    needs something like this to run c3bottles as the document root of a host:

        WSGIScriptAlias / /path/to/c3bottles/wsgi.py
        WSGIApplicationGroup %{GLOBAL}
        Alias /static /path/to/c3bottles/static

# Map

There are two main ways of using the map: For an indoor event, you can use an
appropriate building plan as the base for the map tiles. For an outdoor event
it is useful to use a map source already present, namely OpenStreetMap. These
two ways are described below in the "Internal map" and "OpenStreetMap" sections,
respectively.

*Attention:* No map source is defined by default! Please have a look in
`js/map.js`. This file contains the definition for the two map sources.
Just uncomment the one you want to use.

## Internal map

To be able to use the internal map, in addition to uncommenting it in `map.js`,
you first have to generate the map tiles. The normal build task already does
that with the map delivered with the package. If you replaced the map and just
want to regenerate the tiles using the defaults, you can do this with:

    $ npm run build:map

The standard source file for the map is `static/img/map.png`. For
tile generation, the image has to be a square whose height/width is a
power of 2. It is useful to resize and enlarge the image to the next power
of 2. This can be done with ImageMagick's `convert` utility as follows,
assuming that the image has a largest dimension between 8192 and 16384 and
a white background:

    $ cd /path/to/c3bottles/static/img
    $ convert map.png -background white -compose Copy -gravity center \
                      -resize 16384x16384 -extent 16384x16384 map_sq.png

An image size of 16384x16384 allows a maximum zoom level of 6 which is the
default in the JavaScript code of the map. If your image is larger or
smaller, the `maxZoom` setting in `static/js/map.js` has to be adapted
accordingly (i.e. to 5 for an 8192x8192 image).

Once you have a square source image with a power of 2 in size, you can
generate the tiles as follows:

    $ cd /path/to/c3bottles/static/img
    $ gdal_translate -of vrt map_sq.png map_sq.vrt
    $ gdal2tiles.py -w none -p raster map_sq.vrt tiles

## OpenStreetMap

OpenStreetMap works out of the box once you have uncommented it in `map.js`.
Although, for it to be useful, you should set the appropriate event location
coordinates and a useful zoom level as default view in `map.js`.
