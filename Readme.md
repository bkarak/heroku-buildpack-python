Heroku buildpack: Python
========================

> **Note (October 2026):** this fork is kept for the record. It added two build steps to the 2013 Python buildpack; LuaJIT, for lupa (Lua scripting inside a Python service), and libffi, for pyOpenSSL (validating Apple in-app purchase receipts).
>
> Both steps were fixed in 2026. They now download from Fedora's source archive, which keeps tarballs under their checksum, verify each download's SHA-256, build into `.heroku/vendor` (cached, and on the library paths at run time) and run only when `requirements.txt` asks for lupa, or for cffi, cryptography or pyOpenSSL. The libffi step used to unpack the source and build nothing. The `PKG_CONFIG_PATH` typo is fixed, and an unused copy of the lupa source is gone. The rest of the buildpack is still the 2013 one, and it does not build on any current Heroku stack (Ubuntu 24.04 has no `python2`, which its build tooling needs, and its Python runtimes are no longer downloadable).
>
> Neither step is needed today. `lupa`, `cffi` and `cryptography` install as manylinux wheels that already carry LuaJIT, libffi and OpenSSL, so the stock [heroku/python](https://github.com/heroku/heroku-buildpack-python) buildpack handles a `requirements.txt` with `lupa` and `pyOpenSSL` as it is.
>
> The story of this fork: [Blast from the past: the Heroku buildpacks I forked](https://bkarak.wizhut.tech/blog/2026/29102026).

This is a [Heroku buildpack](http://devcenter.heroku.com/articles/buildpacks) for Python apps, powered by [pip](http://www.pip-installer.org/).

This is a modified version of the buildpack that includes:

- LuaJIT: required by the [lupa](https://pypi.org/project/lupa/) module
- libffi: required by [pyOpenSSL](https://pypi.org/project/pyOpenSSL/), through cryptography and cffi

Usage
-----

Example usage:

    $ ls
    Procfile  requirements.txt  web.py

    $ heroku create --stack cedar --buildpack git://github.com/heroku/heroku-buildpack-python.git

    $ git push heroku master
    ...
    -----> Fetching custom git buildpack... done
    -----> Python app detected
    -----> No runtime.txt provided; assuming python-2.7.3.
    -----> Preparing Python runtime (python-2.7.3)
    -----> Installing Distribute (0.6.34)
    -----> Installing Pip (1.2.1)
    -----> Installing dependencies using Pip (1.2.1)
           Downloading/unpacking Flask==0.7.2 (from -r requirements.txt (line 1))
           Downloading/unpacking Werkzeug>=0.6.1 (from Flask==0.7.2->-r requirements.txt (line 1))
           Downloading/unpacking Jinja2>=2.4 (from Flask==0.7.2->-r requirements.txt (line 1))
           Installing collected packages: Flask, Werkzeug, Jinja2
           Successfully installed Flask Werkzeug Jinja2
           Cleaning up...

You can also add it to upcoming builds of an existing application:

    $ heroku config:add BUILDPACK_URL=git://github.com/heroku/heroku-buildpack-python.git

The buildpack will detect your app as Python if it has the file `requirements.txt` in the root. 

It will use Pip to install your dependencies, vendoring a copy of the Python runtime into your slug. 

Specify a Runtime
-----------------

You can also provide arbitrary releases Python with a `runtime.txt` file.

    $ cat runtime.txt
    python-3.3.2
    
Runtime options include:

- python-2.7.4
- python-3.3.2
- pypy-1.9 (experimental)
