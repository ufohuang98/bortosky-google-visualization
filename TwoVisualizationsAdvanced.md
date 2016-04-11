It is easy to host 2 or more visualizations on a single page.


# Goals #
We want our sample page to
  * Show multiple Google Visualizations
  * Without interfering with one another
  * With the minimum amount of code

We'll need both server- and client-side code.

# Server Side #
The first thing to do is set up our data to be visualized.
We will show a table of programming activities and a table of regional sales for example.

Let's create two server-side properties on our page to provide the two DataTables we'll be visualizing.
These DataTables are just demonstration stubs. Obviously, a real appliction would provide the tables using a database or web service or some proper source of real data to visualize.
```
<script runat="server">

    private System.Data.DataTable ProgrammingTable
    {
        get
        {
            System.Data.DataTable dt = new System.Data.DataTable();
            dt.Columns.Add("STYLE", typeof(System.String)).Caption = "Programming Style";
            dt.Columns.Add("FUN", typeof(System.Int32)).Caption = "Fun";
            dt.Columns.Add("WORK", typeof(System.Int32)).Caption = "Work";
            dt.Rows.Add(new object[] { "Hand Coding", 30, 200 });
            dt.Rows.Add(new object[] { "Using the .NET Library", 300, 10 });
            dt.Rows.Add(new object[] { "Skipping Visualization", -50, 0 });
            return dt;
        }
    }
    private System.Data.DataTable RegionalSales
    {
        get
        {
            System.Data.DataTable dt = new System.Data.DataTable();
            dt.Columns.Add("REGION", typeof(System.String)).Caption = "Region";
            dt.Columns.Add("SALES", typeof(System.Int32)).Caption = "Annual Sales";
            dt.Rows.Add(new object[] { "North", 50000 });
            dt.Rows.Add(new object[] { "South", 150000 });
            dt.Rows.Add(new object[] { "East", 130000 });
            dt.Rows.Add(new object[] { "West", 120000 });
            return dt;
        }
    }
</script>
```

Next, let's provide a helper method to create a GoogleDataTable and provide it for client-side use.
This method takes a DataTable and a client-side JavaScript name for the object with which the GoogleDataTable will be initialized on the client.
It creates a memory stream, writes the GoogleDataTable to it and uses the stream to initialize a client-side variable.
You can view the rendered source of the final page to see the GoogleDataTable client script objects.
```
    private void LoadChart(System.Data.DataTable table, string jsName)
    {
        var gdt = new Bortosky.Google.Visualization.GoogleDataTable(table);
        using (var mem = new System.IO.MemoryStream())
        {
            gdt.WriteJson(mem);
            mem.Position = 0;
            using (var sr = new System.IO.StreamReader(mem))
            {
                Page.ClientScript.RegisterStartupScript(this.GetType(), jsName, string.Format("var {0} = {1};", jsName, sr.ReadToEnd()), true);
                sr.Close();
            };
        }
    }
```

Finally, on the server side, we need to call our LoadChart method once for each of the two data tables.
The Page\_Load event is a good place to do that for demonstration purposes.
We'll use the names (prog and sales) in the client-side code, as you will see.
```
    protected void Page_Load(object sender, EventArgs e)
    {
        LoadChart(this.ProgrammingTable, "prog");
        LoadChart(this.RegionalSales, "sales");
    }
```

# Client Side #
On the client, we need to
  * Declare the html div elements
  * Load the jsapi library from Google's site
  * Load the two visualization packages, columnchart and piechart
  * Initialize Google DataTables using the variables set up in the server-side code (prog and sales)
  * Draw the charts in the html div elements

First, the body section declare the html div elements.
```
<html xmlns="http://www.w3.org/1999/xhtml">
<body>
    <form id="form1" runat="server">
    <h1>Column Chart</h1>
    <div>
        <div id='divprog' style='width: 750px; height: 350px;'>
        </div>
    </div>
    <h1>Bar Chart</h1>    
    <div>
        <div id='divsales' style='width: 350px; height: 350px;'>
        </div>
    </div>
    </form>
</body>
</html>
```

Finally, add this head section to your html element. This client-side script will fill the other client-side needs as listed above.
The jsapi script is loaded from Google.
The columnchart and piechart packages are loaded by name using Google's load method.
Two functions are provided to create Google DataTables using the client-side objects created in our server-side code above (prog and sales) and draw each chart with basic formatting.
Finally, a function is provided simply to call the two drawing functions and that function is registered for the Google API code to run after the page is loaded.
```
<head>
    <script type='text/javascript' src='http://www.google.com/jsapi'></script>
    <script type='text/javascript'>
        google.load('visualization', '1', {'packages':['columnchart','piechart']});
        function drawProgrammingChart() {
            var data = new google.visualization.DataTable(prog, 0.5);
            var chart = new google.visualization.ColumnChart(document.getElementById('divprog'));
            chart.draw(data, {title: 'Visualization Satisfaction', titleX: 'Programming method', titleY: 'Units' });
        }
        function drawSalesChart() {
            var data = new google.visualization.DataTable(sales, 0.5);
            var chart = new google.visualization.PieChart(document.getElementById('divsales'));
            chart.draw(data, { title: 'Regional Sales', titleX: 'Region', titleY: '2009 Sales'});
        }
        function drawBothCharts() {
            drawProgrammingChart();
            drawSalesChart();
        }
        google.setOnLoadCallback(drawBothCharts);
    </script>
</head>
```
The complete code is available as part of the binary download. See http://www.bortosky.com/samples/vis2.aspx for an on-line sample in action.