This post will focus on how to work with a flexible repository pattern in Go, focusing on a practical implementation using a `Prepare` method that can handle both contexts and transactions seamlessly.

### Understanding the Repository Pattern
The repository pattern provides an abstraction over data storage, allowing developers to interact with data sources without exposing details of the underlying data access logic. This pattern is especially useful when dealing with databases, as it enables easy swapping of data sources and simplifies unit testing by decoupling the database logic from the business logic.

### Implementing a Flexible Repository Pattern in Go
To demonstrate a flexible repository pattern in Go, we will create a repository that can operate with both a Context and a Transaction. This flexibility allows our repository methods to work seamlessly with either a non-transactional context or a transactional one, depending on the requirements of the operation.

### The Database Interface
The cornerstone of our flexible repository is the Database interface. This interface defines a Prepare method that allows us to create SQL statements for execution.

```go
package repository

import (
	"database/sql"
	gofrSQL "gofr.dev/pkg/gofr/datasource/sql"
)

type Database interface {
	Prepare(query string) (*sql.Stmt, error)
}

```

### Context and Transaction Types
Next, we define the Context and Transaction types, each implementing the Prepare method. These types will allow us to interact with the database in different scopes.
```go
type Context struct {
    SQL *gofrSQL.DB
}

type Transaction struct {
    SQL *gofrSQL.Tx
}

func (ctx *Context) Prepare(query string) (*sql.Stmt, error) {
    return ctx.SQL.Prepare(query)
}

func (tx *Transaction) Prepare(query string) (*sql.Stmt, error) {
    return tx.SQL.Prepare(query)
}
```
Here, gofrSQL.DB and gofrSQL.Tx represent a context and a transaction, respectively. The Prepare method in both types allows us to compile SQL statements that are ready for execution.

### The Repository Struct
```go
type Repository struct {
	DB Database
}
```

### Using the Repository
To use the repository, we instantiate it with either a Context or a Transaction. Here's an example:
```go
package main

import (
	"fmt"
	"repository"
	"gofr.dev/pkg/gofr/datasource/sql"
)

func main() {
	// Assuming we have a valid SQL database connection
	db := gofrSQL.NewDB("your-database-connection-string")

	// Prepare a statement using the Context
	ctx := &repository.Context{SQL: db}
	repoCtx := &repository.Repository{DB: ctx}
	stmtCtx, err := repoCtx.DB.Prepare("SELECT * FROM users WHERE id = ?")
	if err != nil {
		fmt.Println("Error preparing statement with context:", err)
	} else {
		fmt.Println("Statement prepared with context:", stmtCtx)
	}

	// Prepare a statement using the Transaction
	tx, err := db.Begin()
	if err != nil {
		panic(err)
	}
	transaction := &repository.Transaction{SQL: tx}
	repoTx := &repository.Repository{DB: transaction}
	stmtTx, err := repoTx.DB.Prepare("INSERT INTO users(name) VALUES(?)")
	if err != nil {
		fmt.Println("Error preparing statement with transaction:", err)
        tx.Rollback()
	}
    tx.Commit()
}

```