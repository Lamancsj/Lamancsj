<!DOCTYPE html>
<html>
<head>
    <title>Spread.Sheets ExcelIO</title>

    <script src="http://code.jquery.com/jquery-2.1.3.min.js" type="text/javascript"></script>
    <script src="http://code.jquery.com/ui/1.11.4/jquery-ui.min.js" type="text/javascript"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/FileSaver.js/2014-11-29/FileSaver.min.js"></script>

    <link href="./lib/gc.spread.sheets.excel2013white.10.0.0.css" rel="stylesheet" type="text/css" />
    <script type="text/javascript" src="http://cdn.grapecity.com/spreadjs/hosted/scripts/gc.spread.sheets.all.10.0.0.min.js"></script>
    <script type="text/javascript" src="http://cdn.grapecity.com/spreadjs/hosted/scripts/interop/gc.spread.excelio.10.0.0.min.js"></script>

    <script type="text/javascript">
        var workbook, excelIO, jsonData;
        $(document).ready(function () {
            $.support.cors = true;
            workbook = new GC.Spread.Sheets.Workbook(document.getElementById("ss"));

            excelIO = new GC.Spread.Excel.IO();
            
            function ImportFile() {
                var excelUrl = "./test.xlsx";
                //var excelUrl = $("#importUrl").val();

                var oReq = new XMLHttpRequest();
                oReq.open('get', excelUrl, true);
                oReq.responseType = 'blob';
                oReq.onload = function () {
                    var blob = oReq.response;
                    excelIO.open(blob, LoadSpread, function (message) {
                        console.log(message);
                    });
                };
                oReq.send(null);
            }

            function LoadSpread(json) {
                jsonData = json;
                workbook.fromJSON(json);

                workbook.setActiveSheet("Revenues (Sales)");
            }

            document.getElementById("addRevenue").onclick = function () {
                workbook.suspendPaint();
                workbook.suspendCalcService();

                var sheet = workbook.getActiveSheet();
                sheet.addRows(11, 1);
                sheet.copyTo(10, 1, 11, 1, 1, 29, GC.Spread.Sheets.CopyToOptions.style);

                sheet.setValue(11, 1, "Revenue 8");

                for (var c = 3; c < 15; c++) {
                    sheet.setValue(11, c, Math.floor(Math.random() * 200) + 10);
                }

                var data = new GC.Spread.Sheets.Range(11, 3, 1, 12);
                var setting = new GC.Spread.Sheets.Sparklines.SparklineSetting();
                setting.options.seriesColor = "Text 2";
                setting.options.lineWeight = 1;
                setting.options.showLow = true;
                setting.options.showHigh = true;
                setting.options.lowMarkerColor = "Text 2";
                setting.options.highMarkerColor = "Text 1";

                sheet.setSparkline(11, 2, data, GC.Spread.Sheets.Sparklines.DataOrientation.horizontal, GC.Spread.Sheets.Sparklines.SparklineType.line, setting);

                sheet.setFormula(11, 15, "=SUM([@[Jan]:[Dec]])")
                sheet.setValue(11, 16, 0.15);
                sheet.copyTo(10, 17, 11, 17, 1, 13, GC.Spread.Sheets.CopyToOptions.formula);

                workbook.resumeCalcService();
                workbook.resumePaint();
            }

            document.getElementById("export").onclick = function () {
                ExportFile();
            }

            function ExportFile() {
                var fileName = $("#exportFileName").val();
                if (fileName.substr(-5, 5) !== '.xlsx') {
                    fileName += '.xlsx';
                }
                var json = JSON.stringify(workbook.toJSON());

                excelIO.save(json, function (blob) {
                    saveAs(blob, fileName);
                }, function (e) {
                    if (e.errorCode === 1) {
                        alert(e.errorMessage);
                    }
                });
            }

            ImportFile();
        });
    </script>
</head>
<body>
    <div id="ss" style="height:700px ; width :100%; "></div>
    <input type="text" id="importUrl" value="http://www.testwebsite.com/files/TestExcel.xlsx" style="width:300px" />
    <button id="addRevenue">Add Revenue</button>
    <button id="export">Export File</button>
    <input type="text" id="exportFileName" placeholder="Export file name" value="export.xlsx" />
</body>
</html>
