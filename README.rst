aiomysql
========
.. image:: https://travis-ci.org/aio-libs/aiomysql.svg?branch=master
    :target: https://travis-ci.org/aio-libs/aiomysql
.. image:: https://codecov.io/gh/aio-libs/aiomysql/branch/master/graph/badge.svg
    :target: https://codecov.io/gh/aio-libs/aiomysql
    :alt: Code coverage
.. image:: https://badge.fury.io/py/aiomysql.svg
    :target: https://badge.fury.io/py/aiomysql
    :alt: Latest Version
.. image:: https://readthedocs.org/projects/aiomysql/badge/?version=latest
    :target: https://aiomysql.readthedocs.io/
    :alt: Documentation Status

**aiomysql** is a "driver" for accessing a `MySQL` database
from the asyncio_ (PEP-3156/tulip) framework. It depends on and reuses most
parts of PyMySQL_ . *aiomysql* tries to be like awesome aiopg_ library and
preserve same api, look and feel.

Internally **aiomysql** is copy of PyMySQL, underlying io calls switched
to async, basically ``yield from`` and ``asyncio.coroutine`` added in
proper places)). `sqlalchemy` support ported from aiopg_.


Documentation
-------------
https://aiomysql.readthedocs.io/


Mailing List
------------
https://groups.google.com/forum/#!forum/aio-libs


Basic Example
-------------

**aiomysql** based on PyMySQL_ , and provides same api, you just need
to use  ``await conn.f()`` or ``yield from conn.f()`` instead of calling
``conn.f()`` for every method.

Properties are unchanged, so ``conn.prop`` is correct as well as
``conn.prop = val``.


.. code:: python

    import asyncio
    from aiomysql import create_pool


    loop = asyncio.get_event_loop()

    async def go():
        async with create_pool(host='127.0.0.1', port=3306,
                               user='root', password='',
                               db='mysql', loop=loop) as pool:
            async with pool.get() as conn:
                async with conn.cursor() as cur:
                    await cur.execute("SELECT 42;")
                    value = await cur.fetchone()
                    print(value)


    loop.run_until_complete(go())


Connection Pool
---------------
Connection pooling ported from aiopg_ :

.. code:: python

    import asyncio
    import aiomysql


    async def test_example(loop):
        pool = await aiomysql.create_pool(host='127.0.0.1', port=3306,
                                          user='root', password='',
                                          db='mysql', loop=loop)
        async with pool.acquire() as conn:
            async with conn.cursor() as cur:
                await cur.execute("SELECT 42;")
                print(cur.description)
                (r,) = await cur.fetchone()
                assert r == 42
        pool.close()
        await pool.wait_closed()


    loop = asyncio.get_event_loop()
    loop.run_until_complete(test_example(loop))


Example of SQLAlchemy optional integration
------------------------------------------
Sqlalchemy support has been ported from aiopg_ so api should be very familiar
for aiopg_ user.:

.. code:: python

    import asyncio
    import sqlalchemy as sa

    from aiomysql.sa import create_engine


    metadata = sa.MetaData()

    tbl = sa.Table('tbl', metadata,
                   sa.Column('id', sa.Integer, primary_key=True),
                   sa.Column('val', sa.String(255)))


    async def go(loop):
        engine = await create_engine(user='root', db='test_pymysql',
                                     host='127.0.0.1', password='', loop=loop)
        async with engine.acquire() as conn:
            await conn.execute(tbl.insert().values(val='abc'))
            await conn.execute(tbl.insert().values(val='xyz'))

            async for row in conn.execute(tbl.select()):
                print(row.id, row.val)

        engine.close()
        await engine.wait_closed()


    loop = asyncio.get_event_loop()
    loop.run_until_complete(go(loop))


Requirements
------------

* Python_ 3.3+
* asyncio_ or Python_ 3.4+
* PyMySQL_


.. _Python: https://www.python.org
.. _asyncio: http://docs.python.org/3.4/library/asyncio.html
.. _aiopg: https://github.com/aio-libs/aiopg
.. _PyMySQL: https://github.com/PyMySQL/PyMySQL
.. _Tornado-MySQL: https://github.com/PyMySQL/Tornado-MySQL
