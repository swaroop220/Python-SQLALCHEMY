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

# SQLALCHEMY ORM

* Declare Models

     Here, we define module-level constructs that will form the structures which we will be querying from the database. This structure, known as a Declarative Mapping, defines at once both a Python object model, as well as database metadata that describes real SQL tables that exist, or will exist, in a particular database.

	The Declarative Mapping is the typical way that mappings are constructed in modern SQLAlchemy. The most common pattern is to first construct a base class using the DeclarativeBase superclass. The resulting base class, when subclassed will apply the declarative mapping process to all subclasses that derive from it, relative to a particular registry that is local to the new base by default. The example below illustrates the use of a declarative base which is then used in a declarative table mapping:

```python
	from sqlalchemy import Integer, String, ForeignKey
	from sqlalchemy.orm import DeclarativeBase
	from sqlalchemy.orm import Mapped
	from sqlalchemy.orm import mapped_column
	
	
	# declarative base class
	class Base(DeclarativeBase):
	    pass
	
	
	# an example mapping using the base
	class User(Base):
	    __tablename__ = "user"
	
	    id: Mapped[int] = mapped_column(primary_key=True)
	    name: Mapped[str]
	    fullname: Mapped[str] = mapped_column(String(30))
	    nickname: Mapped[Optional[str]]
	   
```
* Database Metadata

  The term “metadata” generally refers to “data that describes data”; data that itself represents the format and/or structure of some other kind of data. In SQLAlchemy, the term “metadata” typically refers to the MetaData construct, which is a collection of information about the tables, columns, constraints, and other DDL objects that may exist in a particular database.
```Python
	from sqlalchemy import MetaData
	from sqlalchemy import Table, Column, Integer, String
	metadata_obj = MetaData()
	user_table = Table("user_account", metadata_obj, Column("id", Integer, primary_key=True),     Column("name", String(30)), Column("fullname", String), )
```


# Session:

Session establishes all conversations with the database and represents a “holding zone” for all the objects which you’ve loaded or associated with it during its lifespan. It provides the interface where SELECT and other queries are made that will return and modify ORM-mapped objects. The ORM objects themselves are maintained inside the Session, inside a structure called the identity map - a data structure that maintains unique copies of each object, where “unique” means “only one object with a particular primary key”.

* Identity Map:
  
  A mapping between Python objects and their database identities. The identity map is a collection that’s associated with an ORM Session object, and maintains a single instance of every database object keyed to its identity. The advantage to this pattern is that all operations which occur for a particular database identity are transparently coordinated onto a single object instance. When using an identity map in conjunction with an isolated transaction, having a reference to an object that’s known to have a particular primary key can be considered from a practical standpoint to be a proxy to the actual database row.

* Unit of Work:

  The ORM objects maintained by a Session are instrumented such that whenever an attribute or a collection is modified in the Python program, a change event is generated which is recorded by the Session. Whenever the database is about to be queried, or when the transaction is about to be committed, the Session first flushes all pending changes stored in memory to the database. This is known as the unit of work pattern.


The fundamental transactional / database interactive object when using the ORM is called the Session. It refers to a Connection internally which it uses to emit SQL. Also, like the Connection, the Session features “commit as you go” behavior using the Session.commit() method


 The Session doesn’t actually hold onto the Connection object after it ends the transaction. It gets a new Connection from the Engine the next time it needs to execute SQL against the database.

 A session requests a physical database connection from the connection pool only when it actually needs to talk to the database.

* How it Works Behind the Scenes
	* Instantiation is Free:

   		When you call session = Session(), no database connection is opened or fetched yet. The session starts completely detached from the database network

	* Lazy Connection Checkout:

  	  The moment you emit a query (e.g., session.execute() or session.query()), the session reaches out to SQLAlchemy's internal Engine, which handles the Connection Pool. The engine checks out a physical connection and hands it to the session.

	* Connection Pooling Lifespan: 

 		Once a session grabs a connection, it typically holds onto it for the duration of the current transaction.

	* Returning to the Pool:
	
		When you call session.commit(), session.rollback(), or session.close(), the transaction ends, and the physical connection is not destroyed—it is immediately released back into the pool so other parts of your application (or other sessions) can reuse it.

	Because connection allocation is lazy, you can safely create a session even if you aren't sure you'll use it. For example, in web frameworks like FastAPI or Flask, a session is usually created at the very beginning of an HTTP request, but it won't consume a database connection socket unless your code actually triggers a database read or write.


* Session() (Direct Instance):

	Commit-as-you-go
	Lazily started when you emit your first query or modification.
 	* Behavior:
  		* You must manually call session.commit()
    	* You must handle exceptions and call session.rollback() manually.
     	* You must manually call session.close().


* **Opening and closing Session:**
	```python
		from sqlalchemy import create_engine
		from sqlalchemy.orm import Session
		
		# an Engine, which the Session will use for connection
		# resources
		engine = create_engine("postgresql+psycopg2://scott:tiger@localhost/")
		
		# create session and add objects
		with Session(engine) as session:
		    session.add(some_object)
		    session.add(some_other_object)
		    session.commit()
	```

	Above, the Session is instantiated with an Engine associated with a particular database URL. It is then used in a Python context manager (i.e. with: statement) so that it is automatically closed at the end of the block; this is equivalent to calling the Session.close() method.

	The call to Session.commit() is optional, and is only needed if the work we’ve done with the Session includes new data to be persisted to the database. If we were only issuing SELECT calls and did not need to write any changes, then the call to Session.commit() would be unnecessary.

* Session.close()
  
  Close out the transactional resources and ORM objects used by this Session.
  
	 This expunges all ORM objects associated with this Session, ends any transaction in progress and releases any Connection objects which this Session itself has checked out from associated Engine objects. The operation then leaves the Session in a state which it may be used again.

	 In the default running mode the Session.close() method does not prevent the Session from being used again. The Session itself does not actually have a distinct “closed” state; it merely means the Session will release all database connections and ORM objects.

	Setting the parameter Session.close_resets_only to False will instead make the close final, meaning that any further action on the session will be forbidden.

* Session.commit()

	Flush pending changes and commit the current transaction.

	When the COMMIT operation is complete, all objects are fully expired, erasing their internal contents, which will be automatically re-loaded when the objects are next accessed. In the interim, these objects are in an expired state and will not function if they are detached from the Session. Additionally, this re-load operation is not supported when using asyncio-oriented APIs. The Session.expire_on_commit parameter may be used to disable this behavior.

	When there is no transaction in place for the Session, indicating that no operations were invoked on this Session since the previous call to Session.commit(), the method will begin and commit an internal-only “logical” transaction, that does not normally affect the database unless pending flush changes were detected, but will still invoke event handlers and object expiration rules.

	**Note** that after Session.commit() is called, either explicitly or when using a context manager, all objects associated with the Session are expired, meaning their contents are erased to be re-loaded within the next transaction. If these objects are instead detached, they will be non-functional until re-associated with a new Session, unless the Session.expire_on_commit parameter is used to disable this behavior.

  
* Session.begin() (Context Manager)

	* Begin-once
 	* Instantly begun when entering the with block.
  	* Automatically commits when the block finishes cleanly.
  	* Automatically rolls back if an unhandled exception occurs.
  	* Automatically closes the session upon block exit.
  
  We may also enclose the Session.commit() call and the overall “framing” of the transaction within a context manager for those cases where we will be committing data to the database. By “framing” we mean that if all operations succeed, the Session.commit() method will be called, but if any exceptions are raised, the Session.rollback() method will be called so that the transaction is rolled back immediately, before propagating the exception outward. In Python this is most fundamentally expressed using a try: / except: / else: block such as:

```python
	# verbose version of what a context manager will do
	with Session(engine) as session:
	    session.begin()
	    try:
	        session.add(some_object)
	        session.add(some_other_object)
	    except:
	        session.rollback()
	        raise
	    else:
	        session.commit()
```

* Sessionmaker:

  The purpose of sessionmaker is to provide a factory for Session objects with a fixed configuration. As it is typical that an application will have an Engine object in module scope, the sessionmaker can provide a factory for Session objects that are constructed against this engine:

	```python
		from sqlalchemy import create_engine
		from sqlalchemy.orm import sessionmaker
		
		# an Engine, which the Session will use for connection
		# resources, typically in module scope
		engine = create_engine("postgresql+psycopg2://scott:tiger@localhost/")
		
		# a sessionmaker(), also in the same scope as the engine
		Session = sessionmaker(engine)
		
		# we can now construct a Session() without needing to pass the
		# engine each time
		with Session() as session:
		    session.add(some_object)
		    session.add(some_other_object)
		    session.commit()
		# closes the session
	```

* session.add()

  * Place an object into this Session.
  * Objects that are in the transient state when passed to the Session.add() method will move to the pending state, until the next flush, at which point they will move to the persistent state.
  * Objects that are in the detached state when passed to the Session.add() method will move to the persistent state directly.
  * If the transaction used by the Session is rolled back, objects which were transient when they were passed to Session.add() will be moved back to the transient state, and will no longer be present within this Session.
 
	```python
	user1 = User(name="user1")
	user2 = User(name="user2")
	session.add(user1)
	session.add(user2)
	
	session.commit()  # write changes to the database
	```
 
* session.add_all()

  	Add the given collection of instances to this Session.
	```python
	user1 = User(name="user1")
	user2 = User(name="user2")
	session.add_all([user1, user2])
	```

* session.delete()
  
  * Mark an instance as deleted.
  * The object is assumed to be either persistent or detached when passed; after the method is called, the object will remain in the persistent state until the next flush proceeds. During this time, the object will also be a member of the Session.deleted collection.
  * When the next flush proceeds, the object will move to the deleted state, indicating a DELETE statement was emitted for its row within the current transaction. When the transaction is successfully committed, the deleted object is moved to the detached state and is no longer present within this Session.

  **Rules:**
  * Rows that correspond to mapped objects that are related to a deleted object via the relationship() directive are not deleted by default. If those objects have a foreign key constraint back to the row being deleted, those columns are set to NULL. This will cause a constraint violation if the columns are non-nullable.

	```python
  		class User(Base):
		    # ...
		
		    addresses = relationship("Address")

  		sess.delete(user1)
		sess.commit()

	  UPDATE address SET user_id=? WHERE address.id = ?
	  (None, 1)
	  UPDATE address SET user_id=? WHERE address.id = ?
	  (None, 2)
	  DELETE FROM user WHERE user.id = ?
      (1,)
	  COMMIT
	```
  * To change the “SET NULL” into a DELETE of a related object’s row, use the delete cascade on the relationship().

	* delete

		The delete cascade indicates that when a “parent” object is marked for deletion, its related “child” objects should also be marked for deletion.
		```python
		class User(Base):
		    # ...
		
		    addresses = relationship("Address", cascade="all, delete")
		
		user1 = sess1.scalars(select(User).filter_by(id=1)).first()
		address1, address2 = user1.addresses
		
		sess.delete(user1)
		sess.commit()
		
		DELETE FROM address WHERE address.id = ?
		((1,), (2,))
		DELETE FROM user WHERE user.id = ?
		(1,)
		COMMIT
		```
  * Rows that are in tables linked as “many-to-many” tables, via the relationship.secondary parameter, are deleted in all cases when the object they refer to is deleted.
  * When related objects include a foreign key constraint back to the object being deleted, and the related collections to which they belong are not currently loaded into memory, the unit of work will emit a SELECT to fetch all related rows, so that their primary key values can be used to emit either UPDATE or DELETE statements on those related rows. In this way, the ORM without further instruction will perform the function of ON DELETE CASCADE, even if this is configured on Core ForeignKeyConstraint objects.
  * The relationship.passive_deletes parameter can be used to tune this behavior and rely upon “ON DELETE CASCADE” more naturally; when set to True, this SELECT operation will no longer take place, however rows that are locally present will still be subject to explicit SET NULL or DELETE. Setting relationship.passive_deletes to the string "all" will disable all related object update/delete.

	* passive_deletes=False

    	Indicates loading behavior during delete operations.

    	A value of True indicates that unloaded child items should not be loaded during a delete operation on the parent. Normally, when a parent item is deleted, all child items are loaded so that they can either be marked as deleted, or have their foreign key to the parent set to NULL. Marking this flag as True usually implies an ON DELETE <CASCADE|SET NULL> rule is in place which will handle updating/deleting child rows on the database side.
    
		Additionally, setting the flag to the string value ‘all’ will disable the “nulling out” of the child foreign keys, when the parent object is deleted and there is no delete or delete-orphan cascade enabled. This is typically used when a triggering or error raise scenario is in place on the database side. Note that the foreign key attributes on in-session child objects will not be changed after a flush occurs so this is a very special use-case setting. Additionally, the “nulling out” will still occur if the child object is de-associated with the parent.

* When the DELETE occurs for an object marked for deletion, the object is not automatically removed from collections or object references that refer to it. When the Session is expired, these collections may be loaded again so that the object is no longer present. However, it is preferable that instead of using Session.delete() for these objects, the object should instead be removed from its collection and then delete-orphan should be used so that it is deleted as a secondary effect of that collection removal.

	* delete-orphan
 		cascade adds behavior to the delete cascade, such that a child object will be marked for deletion when it is de-associated from the parent, not just when the parent is marked for deletion. This is a common feature when dealing with a related object that is “owned” by its parent, with a NOT NULL foreign key, so that removal of the item from the parent collection results in its deletion.

		delete-orphan cascade implies that each child object can only have one parent at a time, and in the vast majority of cases is configured only on a one-to-many relationship. For the much less common case of setting it on a many-to-one or many-to-many relationship, the “many” side can be forced to allow only a single object at a time by configuring the relationship.single_parent argument, which establishes Python-side validation that ensures the object is associated with only one parent at a time, however this greatly limits the functionality of the “many” relationship and is usually not what’s desired.




## Flushing

When the Session is used with its default configuration, the flush step is nearly always done transparently. Specifically, the flush occurs before any individual SQL statement is issued as a result of a Query or a 2.0-style Session.execute() call, as well as within the Session.commit() call before the transaction is committed. It also occurs before a SAVEPOINT is issued when Session.begin_nested() is used.

A Session flush can be forced at any time by calling the Session.flush() method:

* session.flush()

	Flush all the object changes to the database.

	Writes out all pending object creations, deletions and modifications to the database as INSERTs, DELETEs, UPDATEs, etc. Operations are automatically ordered by the Session’s unit of work dependency solver.

	Database operations will be issued in the current transactional context and do not affect the state of the transaction, unless an error occurs, in which case the entire transaction is rolled back. You may flush() as often as you like within a transaction to move changes from Python to the database’s transaction buffer.

	Parameters:
	objects –

	Optional; restricts the flush operation to operate only on elements that are in the given collection.

	This feature is for an extremely narrow set of use cases where particular objects may need to be operated upon before the full flush() occurs. It is not intended for general use.

The flush which occurs automatically within the scope of certain methods is known as autoflush. Autoflush is defined as a configurable, automatic flush call which occurs at the beginning of methods including:

* Session.execute() and other SQL-executing methods, when used against ORM-enabled SQL constructs, such as select() objects that refer to ORM entities and/or ORM-mapped attributes

* When a Query is invoked to send SQL to the database

* Within the Session.merge() method before querying the database

* When objects are refreshed

* When ORM lazy load operations occur against unloaded object attributes.

There are also points at which flushes occur unconditionally; these points are within key transactional boundaries which include:

* Within the process of the Session.commit() method

* When Session.begin_nested() is called

* When the Session.prepare() 2PC method is used.

The autoflush behavior, as applied to the previous list of items, can be disabled by constructing a Session or sessionmaker passing the Session.autoflush parameter as False:
```python
Session = sessionmaker(autoflush=False)
```

Additionally, autoflush can be temporarily disabled within the flow of using a Session using the Session.no_autoflush context manager:
```python
with mysession.no_autoflush:
    mysession.add(some_object)
    mysession.flush()
```
**To reiterate:** The flush process always occurs when transactional methods such as Session.commit() and Session.begin_nested() are called, regardless of any “autoflush” settings, when the Session has remaining pending changes to process.

As the Session only invokes SQL to the database within the context of a DBAPI transaction, all “flush” operations themselves only occur within a database transaction (subject to the isolation level of the database transaction), provided that the DBAPI is not in driver level autocommit mode. This means that assuming the database connection is providing for atomicity within its transactional settings, if any individual DML statement inside the flush fails, the entire operation will be rolled back.

When a failure occurs within a flush, in order to continue using that same Session, an explicit call to Session.rollback() is required after a flush fails, even though the underlying transaction will have been rolled back already (even if the database driver is technically in driver-level autocommit mode). This is so that the overall nesting pattern of so-called “subtransactions” is consistently maintained. 

* **Expiring/Refreshing:**
  

  



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



