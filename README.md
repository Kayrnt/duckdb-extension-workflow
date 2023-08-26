# duckdb-extension-workflow

## Table function workflow

Here is the workflow of a table function in DuckDB from the user perspective.
Let's assume that the extension is called `example extension` and it implements a table function called `example_function`.

```mermaid
sequenceDiagram
    actor DuckDB user
    participant DuckDB engine
    participant Example extension

    DuckDB user->>DuckDB engine: INSTALL example.duckdb_extension;
    DuckDB engine->>Example extension: INSTALL
    Example extension->>DuckDB engine: duckdb_library_version()
    Note over Example extension,DuckDB engine: ensure the library is compatible with current engine

    DuckDB user->>DuckDB engine: LOAD example;
    DuckDB engine->>Example extension: request extension init
    Example extension->>DuckDB engine: example_extension_init()
    Note over Example extension,DuckDB engine: initialize extension provided functions

    DuckDB user->>DuckDB engine: SELECT * FROM example_function();
    Note over DuckDB user,DuckDB engine: example_function is a table function

    DuckDB engine->>Example extension: request example_function bind
    Example extension->>Example extension: example_function.set_bind_data(bind data)
    Example extension->>DuckDB engine: example_function.bind(function params)
    Note over Example extension,DuckDB engine: Bind is the stage to connect to the data layer <br/> (eg: create a connection to another DB) <br/> it allows to return columns for column pushdown too.

    DuckDB engine->>Example extension: request example_function init
    Example extension->>Example extension: example_function.set_init_data(init data)
    Example extension->>DuckDB engine: example_function.init(bind state)
    Note over Example extension,DuckDB engine: function to init a global function state <br/> (eg: generate the query with filter pushdown & track global progress)
    
    DuckDB engine->>Example extension: request example_function local_init
    Example extension->>Example extension: example_function.set_local_init_data(local init data)
    Example extension->>DuckDB engine: example_function.init(bind state)
    Note over Example extension,DuckDB engine: Optional function to init a thread local function state <br/> (eg: track thread status if you're sharding the workload)

    DuckDB engine->>Example extension: request example_function function
    Example extension->>DuckDB engine: example_function.function(bind data, init data, local init data, output chunk)
    Note over Example extension,DuckDB engine: The "main" function where you insert the data in the output chunk <br/> It's going to be called until the global state is `done` <br/> and should be returned once the output chunk is full

    DuckDB engine->>DuckDB user: Request result
```
