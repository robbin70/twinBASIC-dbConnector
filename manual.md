<div class="center">
  <p align=center><h1>User Manual for DBconnector Package.</h1> <p align=center>
  <span class="description">Robbin Petersson 2026</span>
</div>

## Classes
  - tbDatabase(main)
  - RowSet
  - DatabaseRow
  - DatabaseColumn
  - TableInfo
  - ColumnInfo
  - PreparedSQLStatement


### tbDatabasse
This is the central entry point of the package. It acts as the connection manager and factory method provider, coordinating all direct API communication with the runtime driver libraries (libmariadb.dll and sqlite3.dll).
  - **Properties**
    - **Host As String (Read/Write)**\
      The IP address or domain name of the MariaDB server.
    - **Port As Long (Read/Write)**\
      The port number for the MariaDB connection (defaults to 3306).
    - **User As String (Read/Write)**\
      The username for database authentication for the MariaDB connection.
    - **Password As String (Read/Write)**\
      The password for database authentication for the MariaDB connection.
    - **DatabaseName As String (Read/Write)**\
      The name of the active database on the MariaDB server.
    - **SSLSACertPath As String (Read/Write)**\
      File path to the CA certificate (.pem).
      > [!IMPORTANT]\
      > Mandatory if a secure SSL connection is required by the MariaDB Server.
    - **DatabaseFile As String (Read/Write)**\
      The absolute path with file name to the local SQLite database file on disk.
    - **EncryptionKey As String (Read/Write)**\
      The passphrase if the SQLite database is password-protected (SQLCipher).
    - **LoggLvl As Long (Write-Only)**\
      Configures internal logging level. For a released software, use level 2 as a default value.\
         1 = Critical errors only.\
         2 = Warnings and milestones.\
         3 = Verbose SQL and driver API tracing.\
         4 = Detailed Debug logging.
    - **hasConnected As Boolean (Read-Only)**\
      Returns True if this class instance has successfully established a database connection at least once.
    - **isConnected As Boolean (Read-Only)**\
      Checks if Connection is still Valid or not.
    - **ServerCollation As String (Read-Only)**\
      Returns the active connection/server text collation (e.g., utf8mb4_unicode_ci).
    - **DatabaseCollation As String (Read-Only)**\
      Returns the default text collation configured for the selected database on the server.

  - **Methods**
    - **Sub New(ByVal dbType As String, Optional ByVal iLog As Object = Nothing)**\
      Constructor. Acceptable dbType parameters:\
      "mDB" = mySQL/MariaDB type library\
      "SQLITE" = SQLite type Library\
      iLog se RegisterLoggObj for more instruction.
    - **Sub RegisterLoggObj(iLog As Object)**\
      Attaches or replaces the external logging object at runtime.\
      Object should be a Class, with a Method as Folowing:\
      `Write(message as string, Level as Long,Optional Source as string = “what ever”)`
    - **Function Connect() As Boolean**\
      Initializes and opens a full connection, reads the remote schema into the local memory cache, and populates available collation arrays.
    - **Function reConnect() As Boolean**\
      High-speed connection recovery. Instantly restores the raw database pointer after network dropouts without wasting overhead by re-reading the entire schema metadata cache.
    - **Function SelectSQL(ByVal SQL As String) As RowSet**\
      Executes a SELECT query against the database using a thunking-free processor execution pipeline, returning a populated RowSet.
    - **Function ExecuteSQL(ByVal SQL As String) As Boolean**\
      Executes a raw data modification query (INSERT, UPDATE, DELETE) with complete thunking immunity, returning True upon success.
    - **Function BeginTransaction() As Boolean**\
      Initiates a global database transaction context.
    - **Function Commit() As Boolean**\
      Saves all pending modifications permanently to the database storage engine and terminates the active transaction.
    - **Function Rollback() As Boolean**\
      Reverts all data alterations performed since the beginning of the active transaction.
    - **Function Prepare(ByVal SQL As String) As PreparedSQLStatement**\
      Factory method that compiles an SQL statement on the server and returns a parameterized PreparedSQLStatement object.
    - **Function NewRowSet(ByVal TableName As String) As RowSet**\
      Creates a clean, unpopulated RowSet containing a blank, editable DatabaseRow structured precisely according to the matching table schema.
    - **Function NewTable(ByVal TableName As String) As TableInfo**\
      Factory method that instantiates a blank TableInfo container used to compile and create a brand-new table structure.
    - **Function EditTable(ByVal TableName As String) As TableInfo**\
      Factory method that retrieves a clone of an existing table layout from the schema cache and opens it in structural modification mode (EditMode).
    - **Function GetTableNames() As String()**\
      Returns a string array containing the names of all user-defined tables present in the connected database.
    - **Function GetTableSchema(ByVal Name As String) As TableInfo**\
      Fetches the column architecture and structural layout of a specified table straight from the local memory cache.
    - **Function GetAvailableCollations() As String()**\
      Queries and cache-loads a string matrix detailing all sorting and collation methods supported by the active engine.
    - **Function DropTable(ByVal TableName As String) As Boolean**\
      Permanently deletes a table from the storage engine and purges its metadata profile from the internal schema cache.
    - **Function Disconnect() As Boolean**\
      Safely terminates the database link and cleans up all unmanaged handle assets allocated within the driver libraries.

  - **Events**
    - **Event hasLogg(ByVal Msg As String, ByVal Level As Long, Source as String)**\
      Fires only if no LoggObjet has been provided whit the New or RegisterLoggObj routine.\
      Msg is the mesage, Level se property LoggLvl, Sourrce is "Class name - function/sub name.


### RowSet
This class holds the results of a database query and is utilized to traverse through rows, as well as providing an interface to append or adjust records using the built-in Object-Relational Mapping (ORM) layer.
  - **Properties**
    - **RowCount As Long (Read-Only)**\
      The total number of record rows contained in the query output matrix.
    - **ColumnBy(ByVal Name As String) As DatabaseColumn (Read-Only)**\
      Direct shorthand accessor to pull a specific column object from the current active row by its string name identifier (e.g.,s.ColumnBy("name")).
    - **CurrentRow As DatabaseRow (Read-Only)**\
      Returns the full DatabaseRow entity representing the record at the current pointer position.
    - **EOF As Boolean (Read-Only)**\
      Returns True if the row navigator index has advanced past the final row of the record set.
    - **BOF As Boolean (Read-Only)**\
    - Returns True if the row navigator index resides before the first row of the record set.

  - **Methods**
    - **Sub MoveFirst()**\
      Repositions the record index marker back to the very first item in the query payload.
    - **Sub MoveNext()**\
      Increments the record navigator row index forward by one step.
    - **Sub MovePrevious()**\
      Decrements the record navigator row index backward by one step.
    - **Sub MoveLast()**\
      Advances the record index marker to the absolute final row of the payload.
    - **Sub EditRow()**\
      Activates modification mode on the current row and deploys a fail-safe transaction data lock (unless a global transaction is already operating).
    - **Sub RemoveRow()**\
      Issues a DELETE query against the underlying table matching this record and strips the row element from the local RowSet container.
    - **Function SaveRow() As LongLong**\
      Persists modified states to the server. Triggers an INSERT block if the row was spawned via NewRowSet, or an UPDATE command if the row was opened via EditRow. Returns the newly generated Auto-Increment value upon row insertion.


### DatabaseRow
Represents an individual structural entry line. It maps an internal group of DatabaseColumn entities and supports direct item looping using native For Each loop structures.
  - **Properties**
    - **Column(ByVal Name As String) As DatabaseColumn (Read/Write)**\
      Eigenschaft to extract or override an individual column instance utilizing its semantic text name string. Supports object routing syntax using Property Set.
    - **ColumnCount As Long (Read-Only)**\
      The absolute count of columns structured within the current record line.
    - **LastColumnIndex As Long (Read-Only)**\
      The upper bound 0-based index locator of the row column list (ColumnCount - 1).
    - **SourceTable As String (Read/Write)**\
    - The textual name of the database table from which the columns of this row were populated.

  - **Methods**
    - **Sub New(Optional ByVal row As RowSet = Nothing)**\
      Constructor. Can be instantiated empty or initialized as a deep replica (Constructor-copy) of the active row inside an evaluated RowSet.
    - **Function ColumnAt(ByVal Index As Long) As DatabaseColumn**\
      Pulls a specific column element based on its 0-based numerical index position.
    - **Function Iterator() As IUnknown**\
      Exposes the object collection layout, letting developers loop through the columns using a standard For Each block.

### DatabaseColumn
This class encapsulates the literal cell variant value. It features a strongly typed interface according to Xojo standards, guaranteeing that transitions between raw bytes and twinBASIC types transpire cleanly.
  - **Properties**
    - **Name As String (Read/Write)**\
      The name of the column.
    - **Type As Long (Read/Write)**\
      The internal field indicator index or type constraint assigned to the column.
    - **Value As Variant (Read/Write)**\
      The core underlying cellar value in its raw unmanaged configuration.
    - **StringValue As String (Read/Write)**\
      Processes the cell contents directly as a text string. Smoothly processes Null database values without risking a runtime error.
    - **LongValue As Long (Read/Write)**\
      Maps the cell information directly to a standard signed 32-bit integer.
    - **IntegerValue As Integer (Read/Write)**\
      Maps the cell information directly to a signed 16-bit integer.
    - **DoubleValue As Double (Read/Write)**\
      Maps the cell information directly to a double-precision floating-point value.
    - **BooleanValue As Boolean (Read/Write)**\
      Processes the cell contents as a logical True/False evaluation.
    - **IsNull As Boolean (Read-Only)**\
      Evaluates to True if the underlying record location holds a blank (Null) database mark.


### TableInfo
Supervises all operations regarding a table's morphology, layout, and schema definitions. It functions as a DDL coordinator, generating and dispatching complex CREATE TABLE and ALTER TABLE queries.
  - **Properties**
    - **Name As String (Read/Write)**\
      The name of the table in the database file or server schema.
    - **Columns As Collection (Read-Only)**\
      Exposes the underlying storage group containing all mapped column properties (ColumnInfo items).

  - **Methods**
    - **Sub AddColumn(ByVal iColumn As ColumnInfo)**\
      Appends a new column configuration parameter to the very tail end of the architectural blueprint array.
    - **Sub AddColumnAfter(ByVal TargetColumnOrName As Variant, ByVal NewColumn As ColumnInfo)**\
      Inserts a column element immediately following a targeted field component name identifier.
    - **Sub AddColumnBefore(ByVal TargetColumnOrName As Variant, ByVal NewColumn As ColumnInfo)**\
      Inserts a column element immediately preceding a targeted field component name identifier.
    - **Sub MoveColumnAfter(ByVal SourceColumnOrName As Variant, ByVal TargetColumnOrName As Variant)**\
      Alters column hierarchy, shifting an existing structural item so it resides after a destination element.
    - **Sub MoveColumnBefore(ByVal SourceColumnOrName As Variant, ByVal TargetColumnOrName As Variant)**\
      Alters column hierarchy, shifting an existing structural item so it resides before a destination element.
    - **Sub RenameColumn(ByVal OldName As String, ByVal NewName As String)**\
      Rebrands a column name value in the cache memory structure and refreshes its indexing reference key without disturbing its layout order index.
    - **Sub ModifyColumn(ByVal ModifiedColumn As ColumnInfo)**\
      Swaps out a column layout item with a modernized ColumnInfo package (to change types or NOT NULL rules) while preserving its exact index position.
    - **Sub DeleteColumn(ByVal ColumnOrName As Variant)**\
      Cuts a column element out of the current architectural list structure in memory.
    - **Function Iterator() As IUnknown**\
      Exposes the structural layout mapping, letting consumers loop through column schemas utilizing standard For Each arrays.
    - **Function SaveTable() As Boolean**\
      Traffic manager utility. Syncs structural shapes to disk or server. Deploys a CREATE TABLE script for fresh items, or executes modular updates (ALTER TABLE ADD/MODIFY/RENAME) for existing layouts.


### ColumnInfo
This entity catalogs detailed architectural settings for an isolated table field item. It integrates the global data type enumeration flags to grant full IntelliSense code-completion assistance.
  - **Properties**
    - **Index As Long (Read/Write)**\
      The column positional rank locator value (0-based sequential layout index).
    - **Name As String (Read/Write)**\
      The name of the table column.
    - **OriginalName As String (Read/Write)**\
      Holds the initial column label. Utilized internt to locate and update modified titles on the database server during table saving cycles.
    - **DataType As Variant (Read/Write)**\
      The structural data type constraint. Can be assigned a raw definition string (such as "VARCHAR(100)") or any of the built-in IntelliSense system identifiers like mdb_VARCHAR_100 or lite_TEXT.
    - **Collation As String (Read/Write)**\
      The specific sorting collation assigned to text processing fields (such as utf8mb3_general_ci).
    - **IsPrimaryKey As Boolean (Read/Write)**\
      Evaluates to True if this column acts as the table's unique index identifier.
    - **IsNotNull As Boolean (Read/Write)**\
      Evaluates to True if this column enforces a value presence rule (NOT NULL).
    - **IsAutoIncrement As Boolean (Read/Write)**\
      Evaluates to True if the field value should increment sequentially automatically via the engine upon row generation.
    - **IsUnique As Boolean (Read/Write)**\
      Evaluates to True if every entries recorded within this field segment must be globally unique.


### Global Type Enums Available in tbDatabase:
When creating columns, your consumer application gains auto-completion access to these global structures:

**Enums for MariaDB/MySQL**\
`Public Enum mdbDatatypes`\
`   mdb_INT = 1`\
`   mdb_BIGINT = 2`\
`   mdb_DOUBLE = 3`\
`   mdb_DECIMAL = 4`\
`   mdb_VARCHAR_50 = 5`\
`   mdb_VARCHAR_100 = 6`\
`   mdb_VARCHAR_255 = 7`\
`   mdb_TEXT = 8`\
`   mdb_LONGTEXT = 9`\
`   mdb_DATE = 10`\
`   mdb_DATETIME = 11`\
`End Enum`\

**Enums For SQLite**\
`Public Enum liteDatatypes`\
`   lite_INTEGER = 1`\
`   lite_REAL = 2`\
`   lite_TEXT = 3`\
`   lite_BLOB = 4`\
`   lite_NUMERIC = 5`\
`End Enum`\

