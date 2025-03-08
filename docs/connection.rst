.. _connection:

Connection configuration
========================

ClickHouse SQLAlchemy uses the following syntax for the connection URL:

    .. code-block::

     clickhouse<+driver>://<user>:<password>@<host>:<port>/<database>[?key=value..]

Where:

- **driver** is driver to use. Possible choices: ``http``, ``native``, ``asynch``.
  ``http`` is default. When you omit driver http is used.
- **database** is database connect to. Default is ``default``.
- **user** is database user. Defaults to ``'default'``.
- **password** of the user. Defaults to ``''`` (no password).
- **port** can be customized if ClickHouse server is listening on non-standard
  port.

Additional parameters are passed to driver.

Common options
--------------

- **engine_reflection** controls table engine reflection during table reflection.
  Engine reflection can be very slow if you have thousand of tables. You can
  disable reflection by setting this parameter to ``false``. Possible choices:
  ``true``/``false``. Default is ``true``.
- **server_version** can be used for eliminating initialization
  ``select version()`` query. Generally you shouldn't set this parameter and
  server version will be detected automatically.


Driver options
--------------

There are several options can be specified in query string.

HTTP
~~~~

- **port** is port ClickHouse server is bound to. Default is ``8123``.
- **timeout** in seconds. There is no timeout by default.
- **protocol** to use. Possible choices: ``http``, ``https``. ``http`` is default.
- **verify** controls certificate verification in ``https`` protocol.
  Possible choices: ``true``/``false``. Default is ``true``.

Simple URL example using SQLAlchemy 2.0 style:

    .. code-block:: python

        from sqlalchemy import create_engine
        from sqlalchemy.engine import URL

        url = URL.create(
            drivername="clickhouse+http",
            host="localhost",
            database="db"
        )
        engine = create_engine(url)

URL example for ClickHouse https port:

    .. code-block:: python

        url = URL.create(
            drivername="clickhouse+http",
            username="user",
            password="password",
            host="host",
            port=8443,
            database="db",
            query={"protocol": "https"}
        )
        engine = create_engine(url)

When you are using `nginx` as proxy server for ClickHouse server, the URL creation might look like:

    .. code-block:: python

        url = URL.create(
            drivername="clickhouse+http",
            username="user",
            password="password",
            host="host",
            port=8124,
            database="test",
            query={"protocol": "https"}
        )
        engine = create_engine(url)

If you need control over the underlying HTTP connection, pass a `requests.Session
<https://requests.readthedocs.io/en/master/user/advanced/#session-objects>`_ instance
to ``create_engine()``, like so:

    .. code-block:: python

        from sqlalchemy import create_engine
        from sqlalchemy.engine import URL
        from requests import Session

        url = URL.create(
            drivername="clickhouse+http",
            host="localhost",
            database="test"
        )
        engine = create_engine(
            url,
            connect_args={'http_session': Session()}
        )


Native
~~~~~~

Please note that native connection **is not encrypted**. All data including
user/password is transferred in plain text. You should use this connection over
SSH or VPN (for example) while communicating over untrusted network.

Simple URL example using SQLAlchemy 2.0 style:

    .. code-block:: python

        from sqlalchemy import create_engine
        from sqlalchemy.engine import URL

        url = URL.create(
            drivername="clickhouse+native",
            host="localhost",
            database="db"
        )
        engine = create_engine(url)

All connection string parameters are proxied to ``clickhouse-driver``.
See it's `parameters <https://clickhouse-driver.readthedocs.io/en/latest/api.html#clickhouse_driver.connection.Connection>`__.

Example with LZ4 compression secured with Let's Encrypt certificate on server side:

    .. code-block:: python

        import certify
        from sqlalchemy import create_engine
        from sqlalchemy.engine import URL

        url = URL.create(
            drivername="clickhouse+native",
            username="user",
            password="pass",
            host="host",
            database="db",
            query={
                "compression": "lz4",
                "secure": "True",
                "ca_certs": certify.where()
            }
        )
        engine = create_engine(url)

Example with multiple hosts:

    .. code-block:: python

        url = URL.create(
            drivername="clickhouse+native",
            host="wronghost",
            database="default",
            query={"alt_hosts": "localhost:9000"}
        )
        engine = create_engine(url)


Asynch
~~~~~~

Same as Native.

Simple URL example using SQLAlchemy 2.0 style:

    .. code-block:: python

        from sqlalchemy import create_engine
        from sqlalchemy.engine import URL

        url = URL.create(
            drivername="clickhouse+asynch",
            host="localhost",
            database="db"
        )
        engine = create_engine(url)

All connection string parameters are proxied to ``asynch``.
See it's `parameters <https://github.com/long2ice/asynch/blob/dev/asynch/connection.py>`__.
