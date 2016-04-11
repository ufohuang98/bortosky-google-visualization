#summary Usage and other information
#labels Phase-Implementation

# Introduction #

Google Inc. did not develop the .NET Visualization helper library and is not responsible for it.
The helper library writes a JSON [Google DataTable](http://code.google.com/apis/visualization/documentation/reference.html#dataparam) from a System.Data.DataTable object.

## Usage ##
The library is easy to use.
  * Set up your System.Data.DataTable from any server-side operation. The code below just stubs out a demo.
  * Get the Google DataTable JSON representation of your System.Data.DataTable using the library as shown.
  * Initialize a Google DataTable on the client using the JSON and ClientScript.
  * Pass the Google DataTable to the Visualization API as [documented by Google](http://code.google.com/apis/visualization/documentation/).

### New with Version 1.3 ###
  * JSON names are enclosed in quotation marks. The JSON date format mentioned in Google's documentation and implemented by this library is [controversial](http://stackoverflow.com/questions/206384/how-to-format-json-date).
  * Samples have been updated to use corechart and type inference
  * [New samples for AJAX applications](UsingInAJAXApplications.md)

## Code Sample ##

```
<%@ Page Language="C#" %>

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">

<script runat="server">

    protected void Page_Load(object sender, EventArgs e)
    {
        var googleDataTable = new Bortosky.Google.Visualization.GoogleDataTable(this.ProgrammingTable);
        Page.ClientScript.RegisterStartupScript(
            this.GetType(), "vis", string.Format("var fundata = {0};", googleDataTable.GetJson()), true);
    }
    protected System.Data.DataTable ProgrammingTable
    {
        get // a DataTable filled in any way, for example:
        {
            var dt = new System.Data.DataTable();
            dt.Columns.Add("STYLE", typeof(System.String)).Caption = "Programming Style";
            dt.Columns.Add("FUN", typeof(System.Int32)).Caption = "Fun";
            dt.Columns.Add("WORK", typeof(System.Int32)).Caption = "Work";
            dt.Rows.Add(new object[] { "Hand Coding", 30, 200 });
            dt.Rows.Add(new object[] { "Using the .NET Library", 300, 10 });
            dt.Rows.Add(new object[] { "Skipping Visualization", -50, 0 });
            return dt;
        }
    }
</script>

<html xmlns="http://www.w3.org/1999/xhtml">
<head id="Head1" runat="server">
    <title>.NET Visualization Helper Sample</title>

    <script type="text/javascript" src="http://www.google.com/jsapi"></script>

    <script type="text/javascript">
        google.load("visualization", "1", { "packages": ["corechart"] });
        google.setOnLoadCallback(function () {
            var data = new google.visualization.DataTable(fundata, 0.5);
            var chart = new google.visualization.ColumnChart(document.getElementById("chart_div"));
            chart.draw(data, { title: "Visualization Satisfaction", hAxis: { title: "Programming method" }, vAxis: { title: "Units"} });
        });
    </script>
</head>
<body>
    <form id="form1" runat="server">
    <h1>Visualization Column Chart Sample</h1>
    <div id="chart_div" style="width: 700px; height: 300px;"></div>
    </form>
</body>
</html>
```

Several other sample files are included in the binary download.
  * Simple usage, as shown above
  * Using two visualizations on a single page
  * Using this library in a AJAX application
