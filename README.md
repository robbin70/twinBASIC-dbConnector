<div class="center"align=center>
<h2>twinBASIC-dbConnector</h2>
Built as a twinBASIC Package.<br>
A Wraper for libmariadb and sqlite3 dll library<br><br>
    Robbin Peterson 2026
</div><br>

**Instanciate the main Class 'tbConnector' like**
<br>
> Dim db as tbConnector = new tbConector("mDB")<br>
> db.Host = IP or fqdn<br>
> db.Port = 23306 (default is 3306)<br>
> db.DatabaseName = "Bookstore"<br>
> db.User "Karl"<br>
> db.Password = "mySecret"<br>
> Dim connected as Boolean = db.Connect<br>
<br>
**After initial connection**
You get access to the Database Shema trough<br>
<br>
> Dim Tables() as string = db.GetTableName()<br>
> Dim dbShema as TableInfo = db.getTableSchema(Tables(0))<br>
You can manipulate the Sheme(Tables/Columns) with<br>
> Dim eTable as TableInfo = db.EditTable(Tables(0))<br>
<br>
Create/Manipulate Data in the Database<br>
> Dim rows as RowSet = db.SelectSQL("SELECT * FROM '" & Tables(0) & "' WHER name = '" & User & "')<br>
Or by a prepared statement<br>
> Dim prep as PreparedSQLStatement = db.Prepare("SELECT * FROM '" & Tables(0) & "' WHER name = '" & User & "')<br>
> Dim rows as RowSet = prep.SelectSQL<br>
<br>

For full Documentation se th WIKI
