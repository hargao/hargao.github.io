---
title: Sqlalchemy Session
tags:
---

- session.remove()


- https://github.com/Vankeservice/Gemini/blob/master/tests/views/conftest.py
```python
# -*- coding: utf-8 -*-

import os
import urlparse

import pytest
from flask_migrate import upgrade
from sqlalchemy import event
from sqlalchemy.engine import create_engine
from sqlalchemy.exc import OperationalError, ProgrammingError

from gemini.corelibs.store import db as _db
from gemini.gateway.app import create_app


def _extract_database_name(uri):
    if uri is None:
        raise ValueError("SQLALCHEMY_DATABASE_URI settings error!")

    return urlparse.urlparse(uri).path[1:]


def _create_database(raw_uri, db_name):
    """ Drop testing database if exists, then recreate a new one! """

    template_engine = create_engine(raw_uri.replace(db_name, ""), echo=False)

    conn = template_engine.connect()
    conn = conn.execution_options(autocommit=False)
    conn.execute("ROLLBACK")
    try:
        conn.execute("DROP DATABASE %s" % "unittest_{}".format(db_name))
    except ProgrammingError:
        # Could not drop the database, probably does not exist
        conn.execute("ROLLBACK")
        # may need extra info for debug
        # traceback.print_exc(file=sys.stderr)
    except OperationalError:
        # Could not drop database because it's being accessed by other users (psql prompt open?)
        conn.execute("ROLLBACK")
        # may need extra info for debug
        # traceback.print_exc(file=sys.stderr)

    conn.execute("CREATE DATABASE %s DEFAULT CHARACTER SET utf8;" % "unittest_{}".format(db_name))
    conn.close()

    template_engine.dispose()


@pytest.fixture(scope="session")
def app():
    """
    Returns session-wide application.
    """
    app = create_app(
        _config={
            'DEBUG': True,
            'TESTING': True
        }
    )

    raw_uri = app.config["SQLALCHEMY_DATABASE_URI"]
    db_name = _extract_database_name(raw_uri)

    _create_database(raw_uri, db_name)

    app.config["SQLALCHEMY_DATABASE_URI"] = raw_uri.replace(db_name, "unittest_{}".format(db_name))
    return app


@pytest.fixture(scope="session")
def db(app, request):
    """
    Returns session-wide initialised database.
    """
    with app.app_context():
        # 1. Already drop database before, no need.
        # 2. drop_all cannot drop the alembic table.
        # _db.drop_all()

        work_dir = os.getcwd()
        try:
            # change to root path
            os.chdir(os.path.abspath(os.path.join(app.instance_path, os.path.pardir)))
            upgrade()
            # create_all not perform migrations, so change to upgrade
            # _db.create_all()
        finally:
            os.chdir(work_dir)
    return _db


@pytest.fixture
def session(app, db, request):
    """
    Returns function-scoped session.
    """
    with app.app_context():
        conn = _db.engine.connect()
        txn = conn.begin()

        options = dict(bind=conn, binds={})
        sess = _db.create_scoped_session(options=options)

        # establish  a SAVEPOINT just before beginning the test
        # (http://docs.sqlalchemy.org/en/latest/orm/session_transaction.html#using-savepoint)
        sess.begin_nested()

        @event.listens_for(sess, 'after_transaction_end')
        def restart_savepoint(sess2, trans):
            # Detecting whether this is indeed the nested transaction of the test

            if trans.nested and not trans._parent.nested:
                # The test should have normally called session.commit(),
                # but to be safe we explicitly expire the session
                sess2.expire_all()
                sess2.begin_nested()

        _db.session = sess
        yield sess

        # Cleanup
        sess.remove()
        # This instruction rollback any commit that were executed in the tests.
        txn.rollback()
        conn.close()
```