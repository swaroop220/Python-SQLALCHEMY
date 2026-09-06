# Python-SQLALCHEMY

<img width="469" height="333" alt="image" src="https://github.com/user-attachments/assets/65898559-6e3b-477b-aacd-9aea9d4f05dc" />


## SQL Expression Language:
  SQL Expression Language is a toolkit on its own, independent of the ORM package, which provides a system of constructing SQL expressions represented by composable objects, which can then be “executed” against a target database within the scope of a specific transaction, returning a result set. Inserts, updates and deletes (i.e. DML) are achieved by passing SQL expression objects representing these statements along with dictionaries that represent parameters to be used with each statement.


## SQLAlchemy ORM:
  SQLAlchemy ORM builds upon the Core to provide optional object relational mapping capabilities. The ORM provides an additional configuration layer allowing user-defined Python classes to be mapped to database tables and other constructs, as well as an object persistence mechanism known as the Session. It then extends the Core-level SQL Expression Language to allow SQL queries to be composed and invoked in terms of user-defined objects.


## Engine:
  The Engine is the starting point for any SQLAlchemy application. It’s “home base” for the actual database and its DBAPI, delivered to the SQLAlchemy application through a connection pool and a Dialect, which describes how to talk to a specific kind of database/DBAPI combination.
The purpose of the Engine is to connect to the database by providing a Connection object.
connect() <- Engine <- pool, Dialect <- DBBAPI <- Database

```python
from sqlalchemy import create_engine
engine = create_engine("postgresql+psycopg2://scott:tiger@localhost:5432/mydatabase", echo=True)

```

This echo instruct the Engine to log all of the SQL it emits to a Python logger that will write to standard out

#### engine.connect()
	 creates a database connection and executes the operation in a transaction. The default behavior of the Python DBAPI is that a transaction is always in progress; when the connection is released, a ROLLBACK is emitted to end the transaction. The transaction is not committed automatically; if we want to commit data we need to call Connection.commit() 
	 SQLAlchemy refers to this style as commit as you go.

### engine.begin() 
	this method to get the connection, rather than the Engine.connect() method. This method will manage the scope of the Connection and also enclose everything inside of a transaction with either a COMMIT at the end if the block was successful, or a ROLLBACK if an exception was raised. This style is known as begin once:

## DBAPI
  DBAPI is shorthand for the phrase “Python Database API Specification”. This is a widely used specification within Python to define common usage patterns for all database connection packages. The DBAPI is a “low level” API which is typically the lowest level system used in a Python application to talk to a database. SQLAlchemy’s dialect system is constructed around the operation of the DBAPI, providing individual dialect classes which service a specific DBAPI on top of a specific database engine; for example, the create_engine() URL postgresql+psycopg2://@localhost/test refers to the psycopg2 DBAPI/dialect combination, whereas the URL mysql+mysqldb://@localhost/test refers to the MySQL for Python DBAPI/dialect combination.

## Dialect:
  In SQLAlchemy, the “dialect” is a Python object that represents information and methods that allow database operations to proceed on a particular kind of database backend and a particular kind of Python driver (or DBAPI) for that database. SQLAlchemy dialects are subclasses of the Dialect class (Define the behavior of a specific database and DB-API combination).
The Dialect acts as a factory for other database-specific object implementations including ExecutionContext, Compiled, DefaultGenerator, and TypeEngine.

## Connection Pooling:
  A connection pool is a standard technique used to maintain long running connections in memory for efficient reuse, as well as to provide management for the total number of connections an application might use simultaneously.

Particularly for server-side web applications, a connection pool is the standard way to maintain a “pool” of active database connections in memory which are reused across requests.

SQLAlchemy includes several connection pool implementations which integrate with the Engine. They can also be used directly for applications that want to add pooling to an otherwise plain DBAPI approach.

The Engine returned by the create_engine() function in most cases has a QueuePool integrated, pre-configured with reasonable pooling defaults.

The most common QueuePool tuning parameters can be passed directly to create_engine() as keyword arguments: pool_size, max_overflow, pool_recycle and pool_timeout.
For example:
```python

engine = create_engine(
    "postgresql+psycopg2://me@localhost/mydb", pool_size=20, max_overflow=0
)
```

## Session:
  The fundamental transactional / database interactive object when using the ORM is called the Session. It refers to a Connection internally which it uses to emit SQL. Also, like the Connection, the Session features “commit as you go” behavior using the Session.commit() method

The Session doesn’t actually hold onto the Connection object after it ends the transaction. It gets a new Connection from the Engine the next time it needs to execute SQL against the database.

A session requests a physical database connection from the connection pool only when it actually needs to talk to the database.

#### How it Works Behind the Scenes
  1. Instantiation is Free:
When you call session = Session(), no database connection is opened or fetched yet. The session starts completely detached from the database network.
  3. Lazy Connection Checkout:
The moment you emit a query (e.g., session.execute() or session.query()), the session reaches out to SQLAlchemy's internal Engine, which handles the Connection Pool. The engine checks out a physical connection and hands it to the session.
  4. Connection Pooling Lifespan: 
Once a session grabs a connection, it typically holds onto it for the duration of the current transaction.
  5. Returning to the Pool:
When you call session.commit(), session.rollback(), or session.close(), the transaction ends, and the physical connection is not destroyed—it is immediately released back into the pool so other parts of your application (or other sessions) can reuse it.

Because connection allocation is lazy, you can safely create a session even if you aren't sure you'll use it. For example, in web frameworks like FastAPI or Flask, a session is usually created at the very beginning of an HTTP request, but it won't consume a database connection socket unless your code actually triggers a database read or write.

## Sessionmaker:
  The purpose of sessionmaker is to provide a factory for Session objects with a fixed configuration. As it is typical that an application will have an Engine object in module scope, the sessionmaker can provide a factory for Session objects that are constructed against this engine:

## Session() (Direct Instance):
Commit-as-you-go
  Lazily started when you emit your first query or modification.
 	BehaviorYou must manually call session.commit()
 	You must handle exceptions and call session.rollback() manually.
 	You must manually call session.close().


## Session.begin() (Context Manager)
Begin-once
	Instantly begun when entering the with block.
	Automatically commits when the block finishes cleanly.
	Automatically rolls back if an unhandled exception occurs.
	Automatically closes the session upon block exit.


## How to skip callbacks:

Mutating the Event Registry Directly. (Unregister the event)
  ```python
    # Unregister the event
		event.remove(Session, "before_flush", my_before_flush_listener)

		session.commit() # Flushes quietly

		# Re-register the event
		event.listen(Session, "before_flush", my_before_flush_listener)
  ```
The Context Manager Pattern
```python
from contextlib import contextmanager
	from sqlalchemy import event
	from sqlalchemy.orm import Session

	# 1. Define the context manager with granular flags
	@contextmanager
	def bypass_flush_events(session, before=False, after=False):
	    if before:
	        session.info["skip_before_flush"] = True
	    if after:
	        session.info["skip_after_flush"] = True
	    try:
	        yield
	    finally:
	        # Clean up keys afterward to keep session info pristine
	        session.info.pop("skip_before_flush", None)
	        session.info.pop("skip_after_flush", None)

	# 2. Wire up your listeners to respect their designated flags
	@event.listens_for(Session, "before_flush")
	def my_before_listener(session, flush_context, instances):
	    if session.info.get("skip_before_flush"):
	        return  # Bypassed!
	    print("Running before_flush logic...")

	@event.listens_for(Session, "after_flush")
	def my_after_listener(session, flush_context):
	    if session.info.get("skip_after_flush"):
	        return  # Bypassed!
	    print("Running after_flush logic...")

```

The Core SQL Execution Pattern 
```python
    
		from sqlalchemy import insert
    # Using session.execute bypasses ORM unit-of-work flushes entirely
		session.execute(
		    insert(User).values(username="silent_user", email="test@test.com")
		)

		# This commits the transaction directly without an ORM flush loop
		session.commit() 
```



