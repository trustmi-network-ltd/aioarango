AQL
----

**ArangoDB Query Language (AQL)** is used to read and write data. It is similar
to SQL for relational databases, but without the support for data definition
operations such as creating or deleting :doc:`databases <database>`,
:doc:`collections <collection>` or :doc:`indexes <indexes>`. For more
information, refer to `ArangoDB manual`_.

.. _ArangoDB manual: https://docs.arangodb.com

AQL Queries
===========

AQL queries are invoked from AQL API wrapper. Executing queries returns
:doc:`result cursors <cursor>`.

**Example:**

.. testcode::

    from aioarango import ArangoClient, AQLQueryKillError

    # Initialize the ArangoDB client.
    client = ArangoClient()

    # Connect to "test" database as root user.
    db = await client.db('test', username='root', password='passwd')

    # Insert some test documents into "students" collection.
    await db.collection('students').insert_many([
        {'_key': 'Abby', 'age': 22},
        {'_key': 'John', 'age': 18},
        {'_key': 'Mary', 'age': 21}
    ])

    # Get the AQL API wrapper.
    aql = db.aql

    # Retrieve the execution plan without running the query.
    await aql.explain('FOR doc IN students RETURN doc')

    # Validate the query without executing it.
    await aql.validate('FOR doc IN students RETURN doc')

    # Execute the query
    cursor = await db.aql.execute(
      'FOR doc IN students FILTER doc.age < @value RETURN doc',
      bind_vars={'value': 19}
    )
    # Iterate through the result cursor
    student_keys = [doc['_key'] async for doc in cursor]

    # List currently running queries.
    await aql.queries()

    # List any slow queries.
    await aql.slow_queries()

    # Clear slow AQL queries if any.
    await aql.clear_slow_queries()

    # Retrieve AQL query tracking properties.
    await aql.tracking()

    # Configure AQL query tracking properties.
    await aql.set_tracking(
        max_slow_queries=10,
        track_bind_vars=True,
        track_slow_queries=True
    )

    # Kill a running query (this should fail due to invalid ID).
    try:
        await aql.kill('some_query_id')
    except AQLQueryKillError as err:
        assert err.http_code == 404
        assert err.error_code == 1591

See :ref:`AQL` for API specification.


AQL User Functions
==================

**AQL User Functions** are custom functions you define in Javascript to extend
AQL functionality. They are somewhat similar to SQL procedures.

**Example:**

.. testcode::

    from aioarango import ArangoClient

    # Initialize the ArangoDB client.
    client = ArangoClient()

    # Connect to "test" database as root user.
    db = await client.db('test', username='root', password='passwd')

    # Get the AQL API wrapper.
    aql = db.aql

    # Create a new AQL user function.
    await aql.create_function(
        # Grouping by name prefix is supported.
        name='functions::temperature::converter',
        code='function (celsius) { return celsius * 1.8 + 32; }'
    )
    # List AQL user functions.
    await aql.functions()

    # Delete an existing AQL user function.
    await aql.delete_function('functions::temperature::converter')

See :ref:`AQL` for API specification.


AQL Query Cache
===============

**AQL Query Cache** is used to minimize redundant calculation of the same query
results. It is useful when read queries are issued frequently and write queries
are not.

**Example:**

.. testcode::

    from aioarango import ArangoClient

    # Initialize the ArangoDB client.
    client = ArangoClient()

    # Connect to "test" database as root user.
    db = await client.db('test', username='root', password='passwd')

    # Get the AQL API wrapper.
    aql = db.aql

    # Retrieve AQL query cache properties.
    await aql.cache.properties()

    # Configure AQL query cache properties
    await aql.cache.configure(mode='demand', max_results=10000)

    # Clear results in AQL query cache.
    await aql.cache.clear()

See :ref:`AQLQueryCache` for API specification.
