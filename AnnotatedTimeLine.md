#summary Using SetGoogleDateType on an Annotated Timeline



Use SetGoogleDateType to set how the Visualization library will treat a System.DateTime column.


We'll need both server- and client-side code.

# Server Side #
The first thing to do is set up our data to be visualized.
We will show a table of random bank balances for example.

Let's create a server-side property on our page to provide the DataTable we'll be visualizing.
This DataTable is just a demonstration stub. Obviously, a real appliction would provide the table using a database or web service or some proper source of real data to visualize.
See the call to SetGoogleDateType to which you pass the System.DateTime column and an GoogleDateType enumeration of Date, DateTime or TimeOfDay.
The [Annotated Time Line](http://code.google.com/apis/visualization/documentation/gallery/annotatedtimeline.html) visualization won't work with TimeOfDay.
```
<script runat="server">

    private System.Data.DataTable BalanceTable
    {
        get
        {
            System.DateTime today = System.DateTime.Now;
            System.Random rnd = new System.Random(today.Millisecond);
            System.Data.DataTable dt = new System.Data.DataTable();
            dt.Columns.Add("DATE", typeof(System.DateTime)).Caption = "Date";
            dt.Columns.Add("CHECKING", typeof(System.Decimal)).Caption = "Checking";
            dt.Columns.Add("SAVINGS", typeof(System.Decimal)).Caption = "Savings";
            for (var d = 0; d < 30; d++)
                dt.Rows.Add(new object[] { today.AddDays(-d), d * rnd.Next(500), d * rnd.Next(500) });
            Bortosky
                .Google
                .Visualization
                .GoogleDataTable
                .SetGoogleDateType(dt.Columns["Date"], Bortosky.Google.Visualization.GoogleDateType.Date);
            return dt;
        }
    }
    
</script>
```

Next, let's provide a helper method to create a GoogleDataTable and provide it for client-side use.
Use the GetJson method to return the GoogleDataTable's JSON string value to the client page in a StartupScript.
You can view the rendered source of the final page to see the GoogleDataTable client script objects.
```
    private void LoadChart(System.Data.DataTable table, string jsName)
    {
        Page.ClientScript.RegisterStartupScript(this.GetType(),
            jsName,
            string.Format("var {0} = {1};", jsName, new Bortosky.Google.Visualization.GoogleDataTable(table).GetJson()),
            true);
    }
```

Finally, on the server side, we need to call our LoadChart for the data table.
The Page\_Load event is a good place to do that for demonstration purposes.
```
    protected void Page_Load(object sender, EventArgs e)
    {
        LoadChart(this.BalanceTable, "balances");
    }
```

# Client Side #
On the client, we need to
  * Declare the html div element
  * Load the jsapi library from Google's site
  * Load the visualization package for the annotatedtimeline
  * Initialize Google DataTable using the variables set up in the server-side code (balances)
  * Set up some options for the [Annotated Time Line](http://code.google.com/apis/visualization/documentation/gallery/annotatedtimeline.html)
  * Draw the chart in the html div element

First, the body section declare the html div element.
```
<html xmlns="http://www.w3.org/1999/xhtml">
<body>
    <form id="form1" runat="server">
        <h1>Annotated Time Line</h1>
        <div>
            <div id='divbalances' style='width: 500px; height: 300px;'></div>
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
<head id="Head1" runat="server">
    <script type='text/javascript' src='http://www.google.com/jsapi'></script>
    <script type='text/javascript'>
        google.load('visualization', '1', {'packages':['annotatedtimeline']});
        function drawBalanceChart() {
            var data = new google.visualization.DataTable(balances, 0.5);
            var chart = new google.visualization.AnnotatedTimeLine(document.getElementById('divbalances'));
            var options = { thickness: 1, displayExactValues: true };
            chart.draw(data, options);
        }
        google.setOnLoadCallback(drawBalanceChart);
    </script>
</head>
```
The complete code is available as part of the binary download. See http://www.bortosky.com/samples/visdates.aspx for an on-line sample in action.