# Introduction #
This library is also useful in AJAX applications. Use it to serve your data in JSON form for cross-platform applications.  Use it with popular AJAX libraries like [ASP.NET AJAX](http://ajaxcontroltoolkit.codeplex.com) or [JQuery](http://jquery.com).
# Code Samples #
Shown below are server- and client-side samples demonstrating the use of this helper library in those tiers.
## Server ##
VisualizationProgrammingTasks.ashx is included in the download.
It serves an application/json response of the example ProgrammingTable used in the other samples.
The sample handler also demonstrates the use of a query string value to control the data.

```
<%@ WebHandler Language="C#" Class="VisualizationProgrammingTasks" %>

using System;
using System.Data;
using System.Web;

public class VisualizationProgrammingTasks : IHttpHandler
{
    public void ProcessRequest(HttpContext context)
    {
        context.Response.ContentType = "application/json";
        new Bortosky.Google.Visualization.GoogleDataTable(
            ProgrammingTable(context.Request.QueryString["funest"])).WriteJson(context.Response.OutputStream);
    }

    public bool IsReusable
    {
        get { return false; }
    }

    private static DataTable ProgrammingTable(string mostFunStyle)
    {
        var rand = new System.Random(DateTime.Now.Millisecond);
        var dt = new DataTable();
        dt.Columns.Add("STYLE", typeof(System.String)).Caption = "Programming Style";
        dt.Columns.Add("FUN", typeof(System.Int32)).Caption = "Fun";
        dt.Columns.Add("WORK", typeof(System.Int32)).Caption = "Work";
        foreach (var s in new[] { "Hand Coding", "Using the .NET Library", "Skipping Visualization" })
        {
            dt.Rows.Add(new object[] { s, rand.Next(0, 20 * (s.Equals(mostFunStyle) ? 10 : 1)), rand.Next(0, 20) });
        };
        return dt;
    }
}
```
## Client ##
Visualizationclient.htm is included in the download.
The page uses [JQuery](http://jquery.com) to retrieve the data table from the VisualizationProgrammingTasks handler shown above.

```
<html>
<head runat="server">
    <title>Client-Side Visualization</title>
    <style type="text/css">
        body, select, button {font-family: Verdana, Geneva, Tahoma, sans-serif;}
    </style>
    <script src="http://www.google.com/jsapi" type="text/javascript"></script>
    <script src="http://code.jquery.com/jquery-1.4.3.js" type="text/javascript"></script>
    <script type="text/javascript">
        google.load("visualization", "1", { "packages": ["corechart"] });
        var opts = {
            width: 600, height: 300,
            fontName: "Verdana",
            title: "Visualization Satisfaction",
            hAxis: { title: "Programming method" },
            vAxis: { title: "International Units" }
        };
        $(document).ready(function () {
            $("#btn").click(function () {
                $.getJSON("VisualizationProgrammingTasks.ashx", { funest: $("#mostest").val() }, function (gdt) {
                    new google.visualization.ColumnChart($("#chart_div").get(0))
                        .draw(new google.visualization.DataTable(gdt, 0.5), opts);
                });
                return false;
            });
        });
    </script>
</head>
<body>
    <div>
        <div id="chart_div"></div>
        <span>Which is the most fun?</span>
        <select id="mostest">
            <option>Hand Coding</option>
            <option selected="selected">Using the .NET Library</option>
            <option>Skipping Visualization</option>
        </select>
        <button id="btn">Go</button>
    </div>
</body>
</html>

```