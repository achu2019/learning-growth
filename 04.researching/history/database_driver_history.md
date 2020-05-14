# database driver 的历史

参考资料

https://go-review.googlesource.com/c/go/+/174182/
https://github.com/golang/go/issues/29835

Goals of the db and db/dbimpl packages:

* Provide a generic database API for a variety of SQL or SQL-like
  databases.  There currently exist Go libraries for SQLite, MySQL,
  and Postgres, but all with a very different feel, and often
  a non-Go-like feel.

* Feel like Go.

* Care mostly about the common cases. Common SQL should be portable.
  SQL edge cases or db-specific extensions can be detected and
  conditionally used by the application.  It is a non-goal to care
  about every particular db's extension or quirk.

* Separate out the basic implementation of a database driver
  (implementing the db/dbimpl interfaces) vs the implementation
  of all the user-level types and convenience methods.
  In a nutshell:

  User Code ---> db package (concrete types) ---> db/dbimpl (interfaces)
  Database Driver -> db (to register) + dbimpl (implement interfaces)

* To type casting/conversions consistently between all drivers. To
  achieve this, most of the type conversions are done in the db
  package, not in each driver.  The drivers then only have to deal
  with a smaller set of types.

* Be flexible with type conversions, but be paranoid about silent
  truncation or other loss of precision.

* Handle concurrency well.  Users shouldn't need to care about the
  database's per-connection thread safety issues (or lack thereof),
  and shouldn't have to maintain their own free pools of connections.
  The 'db' package should deal with that bookkeeping as needed.  Given
  a *db.DB, it should be possible to share that instance between
  multiple goroutines, without any extra synchronization.

* Push complexity, where necessary, down into the db+dbimpl packages,
  rather than exposing it to users. Said otherwise, the db package
  should expose an ideal database that's not finnicky about how it's
  accessed, even if that's not true.

* Provide optional interfaces in dbimpl for drivers to implement
  for special cases or fastpaths.  But the only party that knows about
  those is the 'db' package.  To user code, some stuff just might start
  working or start working slightly faster.

  摘自于：https://codereview.appspot.com/4973055


### database driver sql.go 的 history

https://go.googlesource.com/go/+log/refs/heads/master/src/database/sql/sql.go
https://github.com/golang/go/commits/master/src/database/sql/sql.go

### 其他的一些 proposal

https://github.com/golang/go/issues/30870
https://github.com/golang/go/issues/22544
