<div class="center"align=center>
<h2>twinBASIC-dbConnector</h2>
Built as a twinBASIC Package.<br>
A Wraper for libmariadb and sqlite3 dll library<br><br>
    Robbin Peterson 2026
</div><br>

**Instanciate the main Class 'tbConnector' like**\
`Dim db as tbConnector = new tbConector("mDB")`  
`db.Host = IP or fqdn`\
`db.Port = 23306 (default is 3306)`\
`db.DatabaseName = "Bookstore"`\
`db.User "Karl"`\
`db.Password = "mySecret"`\
`Dim connected as Boolean = db.Connect`\

**After initial connection**\
You get access to the Database Shema trough\
`Dim Tables() as string = db.GetTableName()`\
`Dim dbShema as TableInfo = db.getTableSchema(Tables(0))`\
You can manipulate the Sheme(Tables/Columns) with\
`Dim eTable as TableInfo = db.EditTable(Tables(0))`\

**Create/Manipulate Data in the Database**\
`Dim rows as RowSet = db.SelectSQL("SELECT * FROM '" & Tables(0) & "' WHER name = '" & User & "')`\
Or by a prepared statement\
`Dim prep as PreparedSQLStatement = db.Prepare("SELECT * FROM '" & Tables(0) & "' WHER name = '" & User & "')`\
`Dim rows as RowSet = prep.SelectSQL`\

For full Documentation se th WIKI
