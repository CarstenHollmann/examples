﻿# PointZilla

`PointZilla` is a console tool for quickly appending points to a time-series in an AQTS 201x system. You can also use `PointZilla` to [delete all points](#deleting-all-points-in-a-time-series) or [delete a range of existing points](#deleting-a-range-of-points-in-a-time-series) from a time-series.

Download the [latest PointZilla.exe release here](../../../../../../releases/latest)
 
Points can be specified from:
- Command line parameters (useful for appending a single point)
- Signal generators: linear, saw-tooth, square-wave, or sine-wave signals. Useful for just getting *something* into a time-series
- CSV files (including CSV exports from AQTS Springboard) from file, FTP, or HTTP sources.
- Points retrieved live from other AQTS systems, including from legacy 3.X systems.
- The results of a database query (via direct support fo SqlServer, Postgres, and MySql. ODBC connections are supported too, but require configuration)
- `CMD.EXE`, `PowerShell` or `bash`: `PointZilla` works well from within any shell.

Basic time-series will append time/value pairs. Reflected time-series also support setting grade codes and/or qualifiers to each point.

Like its namesake, Godzilla, `PointZilla` can be somewhat awesome, a little scary, and even wreak some havoc every now and then.
- We don't recommend deploying either `PointZilla` or Godzilla in a production environment.
- Don't try to use `PointZilla` to migrate your data. System-wide data migration has many unexpected challenges.
- May contain traces of peanuts.

![Rawrrr!](./PointZilla.png "Rawwr!")

# Requirements

- `PointZilla` requires the .NET 4.7 runtime, which is pre-installed on all Windows 10 and Windows Server 2016 systems, and on nearly all up-to-date Windows 7 and Windows Server 2008 systems.
- `PointZilla` is a stand-alone executable. No other dependencies or installation required.
- An AQTS 2017.2+ system

# Examples

These examples will get you through most of the heavy lifting to get some points into your time-series.

A few interesting operations include:
- [Appending a few random points](#append-something-to-a-time-series)
- [Appending a single point](#append-a-single-point-to-a-time-series)
- [Appending points from a CSV](#append-points-from-a-csv-file)
- [Appending points from Excel](#append-points-from-an-excel-spreadsheet)
- [Appending points from an HTTP request](#append-points-from-an-http-request)
- [Appending points from a database](#append-points-from-a-database-query)
- [Appending points with grades or qualifiers](#appending-grades-and-qualifiers)
- [Appending points with notes](#appending-points-with-notes)
- [Handling historical timezone transitions](#handling-historical-timezone-transitions)
- [Copy points from another time-series](#copying-points-from-another-time-series)
- [Copy points from a separate AQTS system](#copying-points-from-another-time-series-on-another-aqts-system)
- [Delete all the points from a time-series](#deleting-all-points-in-a-time-series)
- [Delete a range of points from a time-series](#deleting-a-range-of-points-in-a-time-series)
- [Export the points from a time-series](#export-the-points-from-a-time-series)
- [Compare the points from two time-series](#comparing-the-points-in-two-different-time-series)

That's quite a bit of Zilla goodness!

### Command line option syntax

All command line options are case-insensitive, and support both common shell syntaxes: either `/Name=value` (for CMD.EXE) or `-Name=value` (for bash and PowerShell).

In addition, the [`@options.txt` syntax](https://github.com/AquaticInformatics/examples/wiki/Common-command-line-options) is supported, to read options from a text file. You can mix and match individual `/name=value` and `@somefile.txt` on the same command line.

Try the `/help` option for a detailed list of options and their default values.

### Authentication credentials

The `/Server` option is required for all operations performed. The `/Username` and `/Password` options default to the stock "admin" credentials, but can be changed as needed.

### Use positional arguments to save typing

Certain frequently used options do not need to be specified using the `/name=value` or `-name=value` syntax.

The `/Command=`, `/TimeSeries=`, and `/CsvFile=` options can all omit their option name. `PointZilla` will be able to determine the appropriate option name from the command line context.

## Append *something* to a time-series

With only a server and a target time-series, `PointZilla` will used its built-in signal generator and append one day's worth of 1-minute values, as a sine wave between 1 and -1, starting at "right now".

```cmd
C:\> PointZilla /Server=myserver Stage.Label@MyLocation

15:02:36.118 INFO  - Generated 1440 SineWave points.
15:02:36.361 INFO  - Connected to myserver (2017.4.79.0)
15:02:36.538 INFO  - Appending 1440 points to Stage.Label@MyLocation (ProcessorBasic) ...
15:02:39.493 INFO  - Appended 1440 points (deleting 0 points) in 3.0 seconds.
```

- The built-in signal generator supports `SineWave`, `SquareWave`, `SawTooth`, and `Linear` signal generation, with configurable amplitude, phase, offset, and period settings.
- Use the `/StartTime=yyyy-mm-ddThh:mm:ssZ` option to change where the generated points will start.

## Append a single point to a time-series

Need one specific value in a time-series? Just add that value to the command line.

This example appends the value 12.5, using the default timestamp of "right now".

```sh
$ ./PointZilla.exe -server=myserver Stage.Label@MyLocation 12.5

15:10:04.176 INFO  - Connected to myserver (2017.4.79.0)
15:10:04.313 INFO  - Appending 1 points to Stage.Label@MyLocation (ProcessorBasic) ...
15:10:05.405 INFO  - Appended 1 points (deleting 0 points) in 1.1 seconds.
```

- You can add as many numeric values on the command line as needed.
- Each generated point will be spaced one `/PointInterval` duration apart (defaults to 1-minute)
- Use the `/StartTime=yyyy-mm-ddThh:mm:ssZ` option to change where the generated points will start.

## Append an explicit Gap in between points

AQTS 2019.1 adds the ability to explicitly append a gap between two points from an external source (like PointZilla!). Explicit gaps can be appended to basic time-series or reflected time-series.

The constraints for appending a gap between two points are:
- The app server must be running AQTS 2019.1-or-higher
- A point with `point.Type = Gap` must exist between two points with valid timestamps. The gap will be inserted at the midpoint of the two timestamps.
- The target timeseries must have a gap tolerance of `NoGaps` at the point where the explicit gap will be inserted.

An explicit gap can be specified in two ways:
#### 1) Add an explicit gap on the command line

Use the case-insensitive positional keyword `Gap` instead of a numeric value to represent a gap.

This example inserts a gap between the values 12.5 and 15.3.

```
$ ./PointZilla.exe -server=myserver Stage.Label@MyLocation 12.5 Gap 15.3

17:05:34.827 INFO  - Connecting to myserver ...
17:05:35.449 INFO  - Connected to myserver (2019.1.110.0)
17:05:35.723 INFO  - Appending 3 points [2019-04-27T00:05:34Z to 2019-04-27T00:06:34Z] to Stage.Label@MyLocation (ProcessorBasic) ...
17:05:44.897 INFO  - Appended 2 points (deleting 0 points) in 9.2 seconds.
```

#### 2) Use the `Gap` keyword instead of a timestamp in a CSV file

When reading data from a CSV file, use the case-insensitive keyword `Gap` in a timestamp or value column to represent an explicit gap.

## Append points from a CSV file

`PointZilla` can also read times, values, grade codes, and qualifiers from a CSV file.

All the CSV parsing options are configurable, but will default to values which match the CSV files exported from AQTS Springboard from 201x systems.

The `-csvFormat=` option supports four pre-configured formats:

| **Format** | Equivalent options |
|---|---|
| `-csvFormat=NG` | `-csvSkipRows=0` <br/> `-csvComment="#"` <br/> `-csvDateTimeField="ISO 8601 UTC"` <br/> `-csvValueField=Value` <br/> `-csvGradeField=Grade` <br/> `-csvQualifiersField=Qualifiers` <br/> |
| `-csvFormat=3X` | `-csvSkipRows=2` <br/> `-csvDateTimeField=Date-Time` <br/> `-csvValueField=Value` <br/> `-csvGradeField=Grade` <br/> `-csvDateTimeFormat="MM/dd/yyyy HH:mm:ss"` |
| `-csvFormat=PointZilla` | `-csvSkipRows=0` <br/> `-csvComment="#"` <br/> `-csvDateTimeField="ISO 8601 UTC"` <br/> `-csvValueField=Value` <br/> `-csvGradeField=Grade` <br/> `-csvQualifiersField=Qualifiers` <br/> |
| `-csvFormat=NWIS` | `-csvSkipRows=0` <br/> `-csvSkipRowsAfterHeader=1` <br/> `-csvComment="#"` <br/> `-csvDelimiter="%09"` <br/> `-csvDateTimeField=datetime` <br/> `-csvDateTimeFormat="yyyy-MM-dd HH:mm"` <br/> `-csvTimezoneField=tz_cd` <br/> `-csvValueField=/_00060/` to match the first QR series. <br/>  <br/> And the `/TimezoneAliases=` are pre-configured to match the USGS timezone names. |

Uploading points exported from an AQTS 20xx system:

```sh
$ ./PointZilla.exe -server=myserver Stage.Label@MyLocation Downloads/Stage.Historical@A001002.EntireRecord.csv

15:29:20.984 INFO  - Loaded 621444 points from 'Downloads/Stage.Historical@A001002.EntireRecord.csv'.
15:29:21.439 INFO  - Connected to myserver (2017.4.79.0)
15:29:21.767 INFO  - Appending 621444 points to Stage.Label@MyLocation (ProcessorBasic) ...
15:29:40.086 INFO  - Appended 621444 points (deleting 0 points) in 18.3 seconds.
```

Parsing CSV files exported from AQTS 3.X systems requires a different CSV parsing configuration.

```sh
$ ./PointZilla.exe -server=myserver Stage.Label@MyLocation Downloads/ExportedFrom3x.csv -csvFormat=3x

13:45:49.400 INFO  - Loaded 250 points from 'Downloads/ExportedFrom3x.csv'.
13:45:49.745 INFO  - Connected to myserver (2017.4.79.0)
13:45:49.944 INFO  - Appending 250 points to Stage.Label@MyLocation (ProcessorBasic) ...
13:45:51.143 INFO  - Appended 250 points (deleting 0 points) in 1.2 seconds.
```

### Use column names or 1-based column indexes to reference a column from your CSV

You can reference a column either by a header name (eg. `-CsvDateTimeField="ISO 8601 UTC"`) or by a 1-based column index (eg. `-CsvDateTimeField=1`). When at least one field has a column name, the `-CsvHasHeaderRow=true` option is assumed.

Referencing columns by name has some nice benefits:
- Columns can appear in any order in the header line.
- Column name matching is case-insensitive.

Referencing columns by name is usually more robust, but you may not have control over the format of the CSV file being consumed.

### When your data isn't at the start of your CSV

Some data files have extra rows at the start. PointZilla has a few options to help locate the start of the data to extract:

The `/CsvComment={prefix}` option tells the CSV parser to skip over any lines that begin with the given prefix.

The `/CsvSkipRows={integer}` option tells the CSV parser to skip over the specified number of lines before parsing the data. Skipped rows are not counted for lines matching the `/CsvComment=` test.

The `/CsvHeaderStartsWith={hint1, hint2, ..., hintN}` option provides the CSV parser with a header-row detection hint, a comma-separated list of expected column names:

- Each hint is trimmed of leading/trailing whitespace.
- Column name matching is case-insensitive.
- If none of the expected column hints are empty, then the match is performed against non-empty fields from the header row. This is usually what you want.
- If any of the expected column hints are empty, then the match is performed column-by-column and blank hints must match blank columns in the header row.

So `/CsvHeaderStartsWith="Date, Time, Value, Grade"` and `/CsvHeaderStartsWith=Date,Time,Value,Grade` will both match:

```csv
Date, Time, Value, Grade, Status, Note
2021-Oct-12, 12:56, 4.5, Good, Normal, Things are fine
```

And will also this CSV with 3 blank columns between the `Date` and `Time` columns:

```csv
Date,,,,Time,Value,Grade,Status,Note
2021-Oct-12,,,,12:56,4.5,Good,Normal,Things are fine
```

Adding empty hint colums like `/CsvHeaderStartsWith="Date, , , , Time, Value, Grade"` or `/CsvHeaderStartsWith=Date,,,,Time,Value,Grade` will only match the second CSV.

### Reading timestamps from CSV files

Timestamps can be extracted in a few ways:
- **DateTime** - From a single column, using the `/CsvDateTimeField` and `/CsvDateTimeFormat` options. This is the default behaviour.
- **DateOnly** and **TimeOnly** - From separate date and time columns, using the `/CsvDateOnlyField`, `/CsvDateOnlyFormat`, `/CsvTimeOnlyField`, and `/CsvTimeOnlyFormat` options.
- When `/CsvDateOnlyField` is used, but no `/CsvTimeOnlyField` option is set, the `/CsvDefaultTimeOfDay` value is used for the time component. (defaulting to midnight at the start of the day).
- When `/CsvDateOnlyField` is used, the `/UtcOffset` option will determine the UTC offset of the timestamp.
- You cannot combine the DateTime options with DateOnly or TimeOnly options.

### Setting the UTC offset for your point timestamps

The AQTS Acquisition API requires unambiguous timestamps. Each timestamp must somehow specify a UTC offset, or use 'Z' to indicate UTC time.

PointZilla supports a few different methods for assigning a UTC offset to each point.
- Extract the offset from the source data.
- Use the `/UtcOffset=` option to set an explicit offset.
- Use the `/Timezone=` option to specify an [IANA timezone name](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) which can adjust for "spring-forward" / "fall-back" daylight saving events.
- Assume the current UTC offset of the computer running PointZilla.

#### Method 1 - Extract the UTC offset from the source data

If your CSV file contains a UTC offset, the `/CsvDateTimeFormat=` or `CsvDateOnlyFormat=` options can be configured to read the the UTC offset from the CSV column field, using the `Z` pattern.

The default `/CsvDateTimeFormat=` value is `yyyy'-'MM'-'dd'T'HH':'mm':'ss;FFFFFFF'Z'`, which is the ISO 8601 standard pattern. See the [NodaTime docs](https://nodatime.org/1.3.x/userguide/instant-patterns) for details on custom format patterns.

If your CSV file contains a timezone column, use the `/CsvTimezoneField=` option to extract the name from the column.

You may also want to set multiple timezone aliases if the built-in names don't match your data. If your CSV has `PST` and `PDT` in its data, but those three-letter abbrievations aren't found in the timezone list, you could use `/TimezoneAliases=PST:UTC-08` and `/TimezoneAliases=PDT:UTC-07` to map the offsets correctly.

See [Handling historical timezone transitions](#handling-historical-timezone-transitions) for more details.

#### Method 2 - Use the `/UtcOffset=` option to set an explicit offset

The `/UtcOffset=` option can be used to set an explicit offset from UTC.

UTC offsets can be +HH[:mm], -HH[:mm], or the [ISO 8601 Duration](https://www.w3.org/TR/xmlschema-2/#duration) format.

The following options are all equivalent ways of specifying Eastern Standard Time (UTC-05:00):
- `/UtcOffset=-05:00`
- `/UtcOffset=-05`
- `/UtcOffset=-5`
- `/UtcOffset=-PT5H`
- `/UtcOffset=PT-5H`

The following options are all equivalent ways of specifying Australian Central Standard Time (UTC+09:30):
- `/UtcOffset=+09:30`
- `/UtcOffset=9:30`
- `/UtcOffset=09:30`
- `/UtcOffset=PT9H30M`

When the `/UtcOffset` value is explicitly set, the value will also be used when creating any time-series or locations.

#### Method 3 - Use the `/Timezone=` option to use a timezone's historical adjustment rules

Some CSV data uses "wall-clock" timestamps, which can shift around during "spring-forward" and "fall-back" daylight saving events.

One way this might show up is when you see PointZilla complaining about duplicate timestamps for a 1-hour period during the fall, as the same timestamps occur twice when the clocks turn back an hour in late October/early November (for the Northern hemisphere, or late April/early May in the Southern hemisphere).

```
14:46:34.068 WARN  - Discarding duplicate CSV point at 2012-11-04T06:00:00Z with value 3.46
14:46:34.068 WARN  - Discarding duplicate CSV point at 2012-11-04T06:15:00Z with value 3.46
14:46:34.070 WARN  - Discarding duplicate CSV point at 2012-11-04T06:30:00Z with value 3.46
14:46:34.070 WARN  - Discarding duplicate CSV point at 2012-11-04T06:45:00Z with value 3.46
14:46:34.071 WARN  - Discarding duplicate CSV point at 2013-11-03T06:00:00Z with value 2.27
14:46:34.072 WARN  - Discarding duplicate CSV point at 2013-11-03T06:15:00Z with value 2.27
14:46:34.072 WARN  - Discarding duplicate CSV point at 2013-11-03T06:30:00Z with value 2.27
14:46:34.072 WARN  - Discarding duplicate CSV point at 2013-11-03T06:45:00Z with value 2.27
14:46:34.073 WARN  - Discarding duplicate CSV point at 2014-11-02T06:00:00Z with value 2.34
14:46:34.073 WARN  - Discarding duplicate CSV point at 2014-11-02T06:15:00Z with value 2.34
14:46:34.074 WARN  - Discarding duplicate CSV point at 2014-11-02T06:30:00Z with value 2.34
14:46:34.074 WARN  - Discarding duplicate CSV point at 2014-11-02T06:45:00Z with value 2.34
```

In that 15-minute signal, there are 4 duplicate points every November as the wall-clock switches from 01:xx AM Eastern Daylight Time back to 01:xx AM Eastern Standard Time.

Ugh ... just ... ugh ...

Using `/Timezone=America/New_York` resolves the issue.

#### Method 4 - Assume the current UTC offset of the computer running PointZilla

When no `/UtcOffset=` or `/Timezone=` or `/CsvTimezoneField=` options are set, and the timestamps are still ambiguous, PointZilla will use its current UTC offset as the offset for all the points it will append.

## Append points from an Excel spreadsheet

All the CSV parsing options also apply to parsing Excel workbooks.

By default, the first sheet in the workbook will be parsed according to the CSV parsing rules.

You can use the `/ExcelSheetNumber=integer` or `/ExcelSheetName=name` options to parse a different sheet in the workbook.

## Append points from an HTTP request

All the CSV parsing options also apply to text downloaded via FTP or HTTP requests.

This approach works when the web request returns a text stream for its response payload.

Here is a an example HTTP request which uses the [USGS NWIS service](https://help.waterdata.usgs.gov/faq/automated-retrievals#Examples) to fetch the last 24 hours of Stage points (HG in AQTS, code 00065 in NWIS) points from a location.

https://nwis.waterservices.usgs.gov/nwis/iv/?format=rdb&sites=16010000&period=P1D

The NWIS data response includes some commented lines at the start, followed by a 2-line header row, and then the tab-delimited (not comma delimited) data rows follow.

```
# Data provided for site 16010000
#            TS   parameter     Description
#         42061       00060     Discharge, cubic feet per second
#         42062       00065     Gage height, feet
#
# Data-value qualification codes included in this output:
#     P  Provisional data subject to revision.
# 
agency_cd	site_no	datetime	tz_cd	42061_00060	42061_00060_cd	42062_00065	42062_00065_cd
5s	15s	20d	6s	14n	10s	14n	10s
USGS	16010000	2022-03-10 00:00	HST	5.34	P	2.08	P
USGS	16010000	2022-03-10 00:05	HST	5.34	P	2.08	P
USGS	16010000	2022-03-10 00:10	HST	5.34	P	2.08	P
USGS	16010000	2022-03-10 00:15	HST	5.34	P	2.08	P
```

This command line will fetch the data, extract the points from the "datetime" and "42062_00065" columns, and append them to an AQTS series.

```sh
$ ./PointZilla.exe -server=doug-vm2019 "Stage.Working@Location" "https://nwis.waterservices.usgs.gov/nwis/iv/?format=rdb&sites=16010000&period=P1D" -CsvDelimiter=%09 -CsvComment="#" -CsvDateTimeField=datetime -CsvValueField=42062_00065 -CsvDateTimeFormat="yyyy-MM-dd HH:mm" -CsvIgnoreInvalidRows=true -CsvTimezoneField=tz_cd -TimezoneAliases=HST:US/Hawaii
16:38:30.539 INFO  - PointZilla v1.0.0.0
16:38:30.592 INFO  - Fetching data from https://nwis.waterservices.usgs.gov/nwis/iv/?format=rdb&sites=16010000&period=P1D ...
16:38:31.653 INFO  - Fetched 23.5 KB in 1 second, 40 milliseconds.
16:38:31.810 INFO  - Loaded 461 points [2022-03-10T08:00:00Z to 2022-03-11T22:20:00Z] from 'https://nwis.waterservices.usgs.gov/nwis/iv/?format=rdb&sites=16010000&period=P1D'.
16:38:31.813 INFO  - Connecting to doug-vm2019 ...
16:38:31.984 INFO  - Connected to doug-vm2019 (2021.4.77.0)
16:38:32.627 INFO  - Appending 461 points [2022-03-10T08:00:00Z to 2022-03-11T22:20:00Z] to Stage.Working@Location (ProcessorBasic) ...
16:38:33.202 INFO  - Appended 461 points and 0 notes (deleting 0 points and 0 notes) in 0.6 seconds.
```

The `-CsvFormat=NWIS` option will preconfigure PointZilla to consume the RDB format of NWIS output.
You can also use the `-CsvValueField=/{regex}/` syntax to more easily match a known NWIS parameter code without caring about the NWIS series ID.

```cmd
PointZilla.exe -server=doug-vm2019 "Stage.Working@Location" -csvFormat=NWIS -CsvValueField=/_00065/ "https://nwis.waterservices.usgs.gov/nwis/iv/?format=rdb&sites=16010000&period=P1D"
16:38:30.539 INFO  - PointZilla v1.0.0.0
16:38:30.592 INFO  - Fetching data from https://nwis.waterservices.usgs.gov/nwis/iv/?format=rdb&sites=16010000&period=P1D ...
16:38:31.653 INFO  - Fetched 23.5 KB in 1 second, 40 milliseconds.
16:38:31.810 INFO  - Loaded 461 points [2022-03-10T08:00:00Z to 2022-03-11T22:20:00Z] from 'https://nwis.waterservices.usgs.gov/nwis/iv/?format=rdb&sites=16010000&period=P1D'.
16:38:31.813 INFO  - Connecting to doug-vm2019 ...
16:38:31.984 INFO  - Connected to doug-vm2019 (2021.4.77.0)
16:38:32.627 INFO  - Appending 461 points [2022-03-10T08:00:00Z to 2022-03-11T22:20:00Z] to Stage.Working@Location (ProcessorBasic) ...
16:38:33.202 INFO  - Appended 461 points and 0 notes (deleting 0 points and 0 notes) in 0.6 seconds.
```

Note: Support for other common web formats like XML, JSON, or Parquet files is not yet supported.

## Append points from a database query

PointZilla can also execute a database query and import the results from the query as a time-series.

Four options control the type of DB connection and the query to execute.

| DbOption | Description |
|---|---|
| `-DbType=type` | The database connection type.<br/>Must be one of `SqlServer`, `Postgres`, `MySql`, or `Odbc`. |
| `-DbConnectionString=connectionString` | The connection string, the syntax of which varies by DB type.<br/><br/>See [https://www.connectionstrings.com/](https://www.connectionstrings.com/) for plenty of examples. |
| `-DbQuery=queryToExecute` | The SQL query to execute, in one of two forms:<br/><br/>**Inline SQL:** (the entire SQL query as a single line of text)<br/>`-DbQuery="SELECT Time, Value FROM Sensor WHERE Parameter='HG' AND Location='Loc33' ORDER BY Time"`<br/><br/>**External SQL File:** (an @ sign, followed by a path to the SQL file)<br/>`-DbQuery=@somePath\myQuery.sql` |
| `-DbNotesQuery=queryToExecute` | An optional query to fetch notes using time ranges.<br/><br/>The query should produce the columns named by the `/NoteStartField`,  `/NoteEndField`,  `/NoteTextField` options.<br/><br/>`/DbNotesQuery="SELECT StartTime, EndTime, NoteText FROM ExternalNotes"`<br/><br/>Both **inline SQL** and **external SQL files** are supported here as well. |

The remaining CSV parsing field options can be used to specify the result columns, by index or by name, to use for constructing the points.

### Use the `SqlServer`, `Postgres`, or `MySql` types if you can. `ODBC` database connections require more setup.

PointZilla comes with easy support for Microsoft SQL Server, Postgres, and MySQL databases. If your database source is one of these types, then use the included native driver support fo easier access.

But PointZilla also supports the ODBC standard, which supports [dozens (more than 50)](https://www.connectionstrings.com/net-framework-data-provider-for-odbc/use-an-odbc-driver-from-net/) of database drivers.

While there is almost always an ODBC driver for every database type, ODBC connections are more complex to configure, since an ODBC-driver for your database must be installed on the system running PointZilla. The 3 native drivers do not require any extra software to be installed.

But sometimes an ODBC is your only option. The following example will query points from a Microsoft Access database using an ODBC connection.

```sh
$ ./PointZilla.exe -server=doug-vm2019 Stage.Working@MyLocation -DbType=Odbc -DbConnectionString="Driver={Microsoft Access Driver (*.mdb, *.accdb)};Dbq=C:\Users\Doug.Schmidt\Documents\Northwind.accdb;Uid=Admin;Pwd=;" -DbQuery="SELECT Orders.[Order Date], Orders.[Shipping Fee] FROM Orders" -CsvDateTimeField="Order Date" -CsvValueField="Shipping Fee"
13:16:13.411 INFO  - PointZilla v1.0.0.0
13:16:13.433 INFO  - Querying Odbc database for points ...
13:16:13.668 INFO  - Connecting to doug-vm2019 ...
13:16:14.316 INFO  - Connected to doug-vm2019 (2020.4.195.0)
13:16:22.795 INFO  - Appending 48 points  [2006-01-15T07:00:00Z to 2006-06-23T07:00:00Z] to Stage.Working@MyLocation (ProcessorBasic) ...
13:16:33.768 INFO  - Appended 48 points (deleting 0 points) in 11 seconds.
```

Note that PointZilla is a 64-bit app, which means your ODBC driver must also be 64-bit. Most organizations are still running 32-bit Microsoft Office, so chances are slim that your system will have the 64-bit Access drivers installed. You will probably need to [follow this guidance](https://knowledge.autodesk.com/support/autocad/learn-explore/caas/sfdcarticles/sfdcarticles/How-to-install-64-bit-Microsoft-Database-Drivers-alongside-32-bit-Microsoft-Office.html) to install the 64-bit Microsoft Access driver.

(See? I told you ODBC was more complex!)

## Realigning CSV points with the `/StartTime` value

When `/CsvRealign=true` is set, all the imported CSV/Excel/DB rows will be realigned to the `/StartTime` option.

This option can be a useful technique to "stitch together" a simulated signal with special shapes at specific times.

## Creating missing time-series or locations (Use with caution!)

By default (the `/CreateMode=Never` option), trying to append points to a non-existent time-series will quickly return an error.

The `/CreateMode=Basic` and `/CreateMode=Reflected` options can be used to quickly create a missing time-series if necessary.
When time-series creation is enabled, a location will also be created if needed.

This mode is useful for testing, but is not recommended for production systems, since the default configurations chosen by PointZilla are likely not correct.

### Caveats about creating a time-series using PointZilla

There are a number of command line options which help you create a time-series correctly (see the "Time Series creation options" section in the `--help` page).

The basic rules for the setting created time-series properties are:
- Use reasonable defaults when possible.
- The parameter's default unit, interpolation type, and monitoring method are used as a starting point.
- No gap tolerance is configured by default.
- The UTC offset of the location will be used as the UTC offset of the time-series, unless the `/UtcOffset=` option is explicitly set.
- If you are copying a time-series from another AQTS system using the [`/SourceTimeSeries=`](#copying-points-from-another-time-series) option, copy as many of the source time-series properties as possible.
- Any command line options you set will override any automatically inferred defaults.

So where does this approach fall down? What are the scenarios where using PointZilla to create and copy a time-series won't give me an exact match of the original?
- PointZilla just copies the corrected points and uses those values as raw point values. You lose the entire correction history.
- PointZilla can't copy the gap tolerance or interpolation type from an AQTS 3.X system. If you need a different value, you'll need to set a `/GapTolerance=` or `/InterpolationType=` command line option explicitly.

### I created my time-series incorrectly, oh no! What do I do now?

[LocationDeleter](https://github.com/AquaticInformatics/examples/blob/master/TimeSeries/PublicApis/SdkExamples/LocationDeleter/Readme.md) (aka. "DeleteZilla") is your friend here.

If your PointZilla command-line creates a time-series incorrectly, just use `LocationDeleter` in [Time-Series Deletion Mode](https://github.com/AquaticInformatics/examples/blob/master/TimeSeries/PublicApis/SdkExamples/LocationDeleter/Readme.md#deleting-time-series) to delete the borked time-series and try again.

## Appending grades and qualifiers

- AQTS reflected time-series have always been able to append grade codes and qualifiers along with point values.
- Starting with AQTS 2019.2 Update 1 (build 19.2.185), you can also append grade codes and qualifiers to basic time-series.

Any grade codes or qualifiers imported from CSV rows or manually set via the `/GradeCode` or `/Qualifiers` options will be appended along with the core timestamp and values.

When specifying point values on the command line, you must specify the `/GradeCode` or `/Qualifiers` option before specifying the numeric value.

To completely disable grade codes or qualifiers from the appended points, set the `/IgnoreGrades=true` or the `/IgnoreQualifiers=true` options. These options can be useful when reading points from files or other AQTS systems which have these metadata, but you only want the timestamp and point values.

Grade codes and qualifiers will not be appended to basic time-series before AQTS 19.2.185.

### Mapping between different grades or qualifiers

If your source data has a different grade or qualifier configuration, you can specify the `/MappedGradeCodes=sourceValue:mappedValue` and `/MappedQualifiers=sourceValue:mappedValue` options multiple times, to coerce grades and qualifiers into the required ranges.

There are some helpful options to give you full control over the mapped metadata:
- If the `sourceValue` component of `sourceValue:mappedValue` is empty, then mapped value is used when none exists in the source data. The works for either grades or qualifiers.
- Mapped grades can also use a `lowSouceValue,highSourceValue:mappedValue` that spans a range of grades by separating the low and high values with a comma.

Using an [`@options.txt` file](https://github.com/AquaticInformatics/examples/wiki/Common-command-line-options) to store the mapping is recommended, since the required configuration may require hundreds of lines.

Ex. If your source file defines grades from 1 (best) to 5 (worst), and 6 through 10 as unusable, but the AQTS system defines grades from 1 (worst) to 50 (best), with -2 being unusable, a `@GradeMap.txt` might look like this:

```
# Map from 1-5 into 50-down-to-1
/MappedGrades=1:50
/MappedGrades=2:40
/MappedGrades=3:30
/MappedGrades=4:20
/MappedGrades=5:10

# Map all the weird grades to the stock AQTS Unusable grade of -2
/MappedGrades=6,10:-2
```

## Appending points with notes

PointZilla v1.0.332+ adds comprehensive support for dealing with time-series notes.

Time-series notes have a start time, and end time, and a text value. Notes can overlap in time, meaning more than one note can apply to any given point (just like qualifiers).

- Notes can be read from a source time-series in the same or separate AQTS system.
- Notes can be read from a CSV, Excel, or database column using the `/CsvNotesField=indexOrName` option.
- Notes can be read from a separate CSV file using the `/CsvNotesFile=path`, `/NoteStartField=`, `/NoteEndField=`, and `/NoteTextField=` options.
- Notes can be read from a separate database query using the `/DbNotesQuery=query`, `/NoteStartField=`, `/NoteEndField=`, and `/NoteTextField=` options.
- Note support can be disabled by setting the `/IgnoreNotes=true` option.

## Handling historical timezone transitions

Dealing with historical timezone transitions (daylight saving adjustments) in the spring and fall is a very tricky problem to handle correctly for all countries for all of modern history.

PointZilla has a few options to help you ingest data that is following a "wall-clock" and needs to be unambiguously resolved to UTC.

PointZilla contains a recent copy of the [IANA timezone database](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones), which contains a history of each timezone's changes to/from daylight saving time (if it changes at all). PointZilla also has access to the historical timezones installed on your Windows computer.

Each timezone is identified by a name, so your goal is to pick the most specific timezone that applies to your data. If your sensor is located in Vancouver, you will want to use the `America/Vancouver` zone instead of `Pacific Standard Time`, since the former knows about quirks about Vancouver's times.
You will often encounter extreme time adjustments during the World War Two era, as cities were trying their best to maximize daylight hours to conserve electricity.

- The `/Timezone=` option can be set to specify a zone to use when converting points from external sources, like CSV or Excel files.
- The `/CsvTimezoneField=` option can be set to pull the timezone name from a column of your data.
- The `/TimezoneAliases=` option can be use to supply some aliases to known offsets. If your CSV has `PST` and `PDT` in its data, but those three-letter abbrievations aren't found in the timezone lists, you could use `/TimezoneAliases=PST:UTC-08` and `/TimezoneAliases=PDT:UTC-07` to map the offsets correctly.

### Discovering the available timezone names with the `/FindTimezones=` option

Many of the timezone names are not what you might first expect. So you can use the `/FindTimezones=` option to see what PointZilla thinks.

When the `/FindTimezones=` option is used, PointZilla will just try its best to find a matching timezone, or suggest a similar match.

Both the recent min/max offset (within the last 10 years) and the historical min/max offsets are shown for each match or partial match.

Try a name of a nearby big city, or a country:
```
$ ./PointZilla.exe -findtimezones="madrid"
15:50:03.108 INFO  - 'madrid' did not exactly match any known timezone.
15:50:03.145 INFO  -
15:50:03.145 INFO  - Current local timezone:
15:50:03.146 INFO  - =======================
15:50:03.159 INFO  - 'America/Los_Angeles' Recent:[Min=-08, Max=-07] Historical:[Min=-08, Max=-07]
15:50:03.160 INFO  - 'Pacific Standard Time' Recent:[Min=-08, Max=-07] Historical:[Min=-08, Max=-07]
15:50:03.187 INFO  -
15:50:03.187 INFO  - Partially matched timezones:
15:50:03.188 INFO  - ============================
15:50:03.190 INFO  - 'Europe/Madrid' Recent:[Min=+01, Max=+02] Historical:[Min=-00:14:44, Max=+02]
```

PointZilla is event a bit forgiving of typos:
```
$ ./PointZilla.exe -findtimezones=Portgual
15:52:09.628 INFO  - 'Portgual' did not exactly match any known timezone.
15:52:09.663 INFO  -
15:52:09.663 INFO  - Current local timezone:
15:52:09.664 INFO  - =======================
15:52:09.676 INFO  - 'America/Los_Angeles' Recent:[Min=-08, Max=-07] Historical:[Min=-08, Max=-07]
15:52:09.677 INFO  - 'Pacific Standard Time' Recent:[Min=-08, Max=-07] Historical:[Min=-08, Max=-07]
15:52:09.718 INFO  -
15:52:09.718 INFO  - Similarly named timezones:
15:52:09.719 INFO  - ==========================
15:52:09.720 INFO  - Did you mean 'Portugal' Recent:[Min=+00, Max=+01] Historical:[Min=-00:36:45, Max=+02]?
```

Or if you know the two offsets, try it in min/max form to see which zones match that pattern:
```
$ ./PointZilla.exe -findtimezones=UTC-08/UTC-07
15:55:42.905 INFO  - 'UTC-08/UTC-07' did not exactly match any known timezone.
15:55:42.940 INFO  -
15:55:42.941 INFO  - Current local timezone:
15:55:42.942 INFO  - =======================
15:55:42.952 INFO  - 'America/Los_Angeles' Recent:[Min=-08, Max=-07] Historical:[Min=-08, Max=-07]
15:55:42.952 INFO  - 'Pacific Standard Time' Recent:[Min=-08, Max=-07] Historical:[Min=-08, Max=-07]
15:55:42.993 INFO  -
15:55:42.994 INFO  - Equivalent timezones:
15:55:42.995 INFO  - =====================
15:55:43.136 INFO  - 'UTC-08/UTC-07' matches 17 timezones:
15:55:43.136 INFO  - 'America/Dawson' Recent:[Min=-08, Max=-07] Historical:[Min=-09:17:40, Max=-07]
15:55:43.137 INFO  - 'America/Ensenada' Recent:[Min=-08, Max=-07] Historical:[Min=-08, Max=-07]
15:55:43.138 INFO  - 'America/Fort_Nelson' Recent:[Min=-08, Max=-07] Historical:[Min=-08:10:47, Max=-07]
15:55:43.138 INFO  - 'America/Los_Angeles' Recent:[Min=-08, Max=-07] Historical:[Min=-08, Max=-07]
15:55:43.139 INFO  - 'America/Santa_Isabel' Recent:[Min=-08, Max=-07] Historical:[Min=-08, Max=-07]
15:55:43.139 INFO  - 'America/Tijuana' Recent:[Min=-08, Max=-07] Historical:[Min=-08, Max=-07]
15:55:43.139 INFO  - 'America/Vancouver' Recent:[Min=-08, Max=-07] Historical:[Min=-08:12:28, Max=-07]
15:55:43.140 INFO  - 'America/Whitehorse' Recent:[Min=-08, Max=-07] Historical:[Min=-09:00:12, Max=-07]
15:55:43.140 INFO  - 'Canada/Pacific' Recent:[Min=-08, Max=-07] Historical:[Min=-08:12:28, Max=-07]
15:55:43.140 INFO  - 'Canada/Yukon' Recent:[Min=-08, Max=-07] Historical:[Min=-09:00:12, Max=-07]
15:55:43.141 INFO  - 'Mexico/BajaNorte' Recent:[Min=-08, Max=-07] Historical:[Min=-08, Max=-07]
15:55:43.142 INFO  - 'PST8PDT' Recent:[Min=-08, Max=-07] Historical:[Min=-08, Max=-07]
15:55:43.142 INFO  - 'US/Pacific' Recent:[Min=-08, Max=-07] Historical:[Min=-08, Max=-07]
15:55:43.143 INFO  - 'US/Pacific-New' Recent:[Min=-08, Max=-07] Historical:[Min=-08, Max=-07]
15:55:43.145 INFO  - 'Pacific Standard Time' Recent:[Min=-08, Max=-07] Historical:[Min=-08, Max=-07]
15:55:43.145 INFO  - 'Pacific Standard Time (Mexico)' Recent:[Min=-08, Max=-07] Historical:[Min=-08, Max=-07]
15:55:43.146 INFO  - 'Yukon Standard Time' Recent:[Min=-08, Max=-07] Historical:[Min=-07, Max=-07]
```

## Copying points from another time-series

When the `/SourceTimeSeries` option is set, the corrected point values from the source time-series will be copied to the target `/TimeSeries`.

Unlike the target time-series, which are restricted to basic or reflected time-series types, a source time-series can be of any type.

- Corrected point values will be copied
- Corrected point grade codes will be copied, unless the `/IgnoreGrades=true` option is set.
- Corrected point qualifier codes will be copied, unless the `/IgnoreQualifiers=true` option is set.
- Corrected point notes will be copied, unless the `/IgnoreNotes=true` option is set.
- The correction history of the source series will be converted into time-series notes in the destination series, unless the `/IgnoreNotes=true` option is set.

The `/SourceQueryFrom` and `/SourceQueryTo` options can be used to restrict the range of points and metadata copied. When omitted, the entire record will be copied.

```sh
$ ./PointZilla.exe -server=myserver Stage.Label@MyLocation -sourceTimeSeries=Stage.Working@OtherLocation

15:18:32.711 INFO  - Connected to myserver (2017.4.79.0)
15:18:35.255 INFO  - Loaded 1440 points from Stage.Working@OtherLocation
15:18:35.356 INFO  - Connected to myserver (2017.4.79.0)
15:18:35.442 INFO  - Appending 1440 points to Stage.Label@MyLocation (ProcessorBasic) ...
15:18:37.339 INFO  - Appended 1440 points (deleting 0 points) in 1.9 seconds.
```

## Copying points from another time-series on another AQTS system

The `/SourceTimeSeries=[otherserver]parameter.label@location` syntax can be used to copy time-series points from a completely separate AQTS system.

If different credentials are required for the other server, use the `[otherserver:username:password]parameter.label@location` syntax.

```sh
$ ./PointZilla.exe -server=myserver Stage.Label@MyLocation -sourcetimeseries="[otherserver]Stage.Working@OtherLocation"

13:31:57.829 INFO  - Connected to otherserver (3.10.905.0)
13:31:58.501 INFO  - Loaded 250 points from Stage.Working@OtherLocation
13:31:58.658 INFO  - Connected to myserver (2017.4.79.0)
13:31:58.944 INFO  - Appending 250 points to Stage.Label@MyLocation (ProcessorBasic) ...
13:32:00.148 INFO  - Appended 0 points (deleting 0 points) in 1.2 seconds.
```

The source time-series system can be any AQTS system as far back as AQUARIUS Time-Series 3.8.

## Export the points from a time-series

Use the `/SaveCsvPath=fileOrFolder` and the `/SourceTimeSeries=` options to save the extracted points to a CSV file.

```
$ PointZilla -server=myserver -sourceTimeSeries="Stage.Label@MyLocation" -saveCsvPath=myfolder -stopAfterSavingCsv=true

11:11:33.651 INFO  - Connecting to myserver to retrieve points ...
11:11:39.735 INFO  - Connected to myserver (2020.4.195.0)
11:11:46.565 INFO  - Loading points from 'Stage.Label@MyLocation' ...
11:11:47.632 INFO  - Loaded 14695 points from Stage.Label@MyLocation
11:11:47.686 INFO  - Saving 14695 extracted points to '.\Stage.Label@MyLocation.EntireRecord.csv' ...
```

If the `/SaveCsvPath=` option specifies an existing folder, then PointZilla will generate filename like `{folder}\{timeseriesIdentifier}.{exportedTimeRange}.csv`.
If the `/SaveCsvPath=` option doesn't match an existing folder, then the value is used as the CSV filename.

If you only want to save the CSV, and don't want PointZilla to actually append the points anywhere, add the `/StopAfterSavingCsv=true` option to just exit.

The saved CSV has the following shape:

```
# Stage.Label@MyLocation.EntireRecord.csv generated by PointZilla v1.0.310.0
#
# Time series identifier: Stage.Label@MyLocation
# Location: MyLocation
# UTC offset: (UTC-07:00)
# Value units: m
# Value parameter: Stage
# Interpolation type: InstantaneousValues
# Time series type: ProcessorDerived
#
# Export options: Corrected signal from StartOfRecord to EndOfRecord
#
# CSV data starts at line 15.
#
ISO 8601 UTC, Value, Grade, Qualifiers
1954-06-08T21:00:00Z, 33.3375062721, 0,
1954-06-09T21:00:00Z, 30.2475987522, 0,
1954-06-10T21:00:00Z, 27.3137423987, 0,
```

### Exporting the notes from a time-series

The `/SaveNotesMode=Disabled|WithPoints|SeparateCsv` option controls how any loaded notes are exported.

By default, the `/SaveNotesMode=Disabled` option is assumed, since dealing with multi-line notes in CSV files can be tricky for some external tools. PointZilla follows the de-facto CSV standard of enclosing any text in double quotes `"` and allowing the text to span multiple lines. But there is no true CSV standard for this feature and many popular tools (*cough* Excel! *cough*) can easily be confused when a line of text spans multiple lines.

The `/SaveNotesMode=WithPoints` option will add a fifth column, named "Notes" to the CSV file, and append all the notes in effect at any given time. `WithPoints` mode can quickly explode the size of the exported CSV file, since a year long note of the 20-character note "This data is suspect" applied to a 15-minute signal will create 35,000 copies of that note, taking roughly 770KB to do so.

```
# Stage.Label@MyLocation.EntireRecord.csv generated by PointZilla v1.0.331.0
#
# Time series identifier: Stage.Label@MyLocation
# Location: MyLocation
# UTC offset: (UTC-07:00)
# Value units: m
# Value parameter: Stage
# Interpolation type: InstantaneousValues
# Time series type: ProcessorDerived
#
# Export options: Corrected signal from StartOfRecord to EndOfRecord
#
# CSV data starts at line 15.
#
ISO 8601 UTC, Value, Grade, Qualifiers, Notes
1954-06-08T21:00:00Z, 33.3375062721, 0, This data is suspect
1954-06-09T21:00:00Z, 30.2475987522, 0, This data is suspect
1954-06-10T21:00:00Z, 27.3137423987, 0, This data is suspect
...
```

The `/SaveNotesMode=SeparateCsv` option will export the notes into a separate "**.Notes.csv" file, using just a single line to represent the year-long comment. This yields the smallest  CSV export, but with the trade-off of required two files per series.

```
# Stage.Label@MyLocation.EntireRecord.Notes.csv generated by PointZilla v1.0.331.0
#
# Time series identifier: Stage.Label@MyLocation
# Location: MyLocation
# UTC offset: (UTC-07:00)
#
# Export options: Corrected signal notes from StartOfRecord to EndOfRecord
#
# CSV data starts at line 11.
#
StartTime, EndTime, NoteText
1950-01-01T00:00:00Z, 2005-04-01T00:00:00Z, This data is suspect
...
```

## Comparing the points in two different time-series

This builds on the previous export scenario, to export two series to two CSV files, and then use standard text differencing tools to see if anything is different.

Here is a bash script which compares the saved CSV output of two time-series. This only compares the corrected points, but that is usually a good indicator of "sameness".

```sh
$ ./PointZilla.exe -saveCsvPath=system1 -stopAfterSavingCsv=true -sourceTimeSeries="[old3xServer]Stage.Primary@Location1"
$ ./PointZilla.exe -saveCsvPath=system2 -stopAfterSavingCsv=true -sourceTimeSeries="[newNgServer]Stage.Primary@Location1"
$ diff system1/Stage.Primary@Location1.EntireRecord.csv system2/Stage.Primary@Location1.EntireRecord.csv && echo "Time-series are identical." || echo "Nope, they are different"
```

## Deleting all points in a time-series

The `DeleteAllPoints` command can be used to delete the entire record of point values and notes from a time-series.

```sh
$ ./PointZilla.exe -server=myserver Stage.Label@MyLocation deleteallpoints

15:27:17.220 INFO  - Connected to myserver (2017.4.79.0)
15:27:17.437 INFO  - Deleting all existing points from Stage.Label@MyLocation (ProcessorBasic) ...
15:27:21.456 INFO  - Appended 0 points (deleting 622884 points) in 4.0 seconds.
```

With great power ... yada yada yada. Please don't wipe out your production data with this command.

## Deleting a range of points in a time-series

You can delete a range of points and notes in a basic or reflected time-series by:
- specifying the `DeleteTimeRange` command
- specifying the `/TimeRange=startTime/endTime` option to define the exact time range to be replaced with no points at all

```sh
$ ./PointZilla.exe -server=myserver Stage.Label@Location DeleteTimeRange -TimeRange=2018-04-25T00:00:00Z/2018-04-29T00:00:00Z
17:02:23.889 INFO  - Connected to myserver (2018.1.98.0)
17:02:24.076 INFO  - Appending 0 points within TimeRange=2018-04-25T00:00:00Z/2018-04-29T00:00:00Z to Stage.Label@Location (ProcessorBasic) ...
17:02:24.719 INFO  - Appended 0 points (deleting 1440 points) in 0.6 seconds.
```

## Command line options

Like `curl`, the `PointZilla` tool has dozens of command line options, which can be a bit overwhelming. Fortunately, you'll rarely need to use all the options at once.

Try the `/Help` option to see the entire list of supported options and read the [wiki for the @optionsFile syntax](https://github.com/AquaticInformatics/examples/wiki/Common-command-line-options).

```
Append points to an AQTS time-series.

usage: PointZilla [-option=value] [@optionsFile] [command] [identifierOrGuid] [value] [csvFile] ...

Supported -option=value settings (/option=value works too):

  -Server                   AQTS server name
  -Username                 AQTS username [default: admin]
  -Password                 AQTS password [default: admin]
  -SessionToken
  -Wait                     Wait for the append request to complete [default: True]
  -AppendTimeout            Timeout period for append completion, in .NET TimeSpan format.
  -BatchSize                Maximum number of points to send in a single append request [default: 500000]

  ========================= Time-series options:
  -TimeSeries               Target time-series identifier or unique ID
  -TimeRange                Time-range for overwrite in ISO8061/ISO8601 (defaults to start/end points)
  -Command                  Append operation to perform.  One of Auto, Append, OverwriteAppend, Reflected, DeleteAllPoints, DeleteTimeRange. [default: Auto]
  -GradeCode                Optional grade code for all appended points
  -Qualifiers               Optional qualifier list for all appended points

  ========================= Metadata options:
  -IgnoreGrades             Ignore any specified grade codes. [default: False]
  -IgnoreQualifiers         Ignore any specified qualifiers. [default: False]
  -IgnoreNotes              Ignore any specified notes. [default: False]
  -MappedGrades             Grade mapping in sourceValue:mappedValue syntax. Can be set multiple times.
  -MappedQualifiers         Qualifier mapping in sourceValue:mappedValue syntax. Can be set multiple times.
  -ManualNotes              Set a time-series note, in StartTime/EndTime/NoteText format. Can be set multiple times.
  -CsvNotesFile             Load time-series notes from a file with StartTime, EndTime, and NoteText columns.
  -NoteStartField           CSV column index or name for note start times [default: NoteStartField:'StartTime']
  -NoteEndField             CSV column index or name for note end times [default: NoteEndField:'EndTime']
  -NoteTextField            CSV column index or name for note text [default: NoteTextField:'NoteText']

  ========================= Time-series creation options:
  -CreateMode               Mode for creating missing time-series. One of Never, Basic, Reflected. [default: Never]
  -GapTolerance             Gap tolerance for newly-created time-series. [default: "MaxDuration"]
  -UtcOffset                UTC offset for any created time-series or location. [default: Use system timezone]
  -Unit                     Time-series unit
  -InterpolationType        Time-series interpolation type. One of InstantaneousValues, PrecedingConstant, PrecedingTotals, InstantaneousTotals, DiscreteValues, SucceedingConstant.
  -Publish                  Publish flag. [default: False]
  -Description              Time-series description [default: Created by PointZilla]
  -Comment                  Time-series comment
  -Method                   Time-series monitoring method
  -ComputationIdentifier    Time-series computation identifier
  -ComputationPeriodIdentifier Time-series computation period identifier
  -SubLocationIdentifier    Time-series sub-location identifier
  -TimeSeriesType           Time-series type. One of Unknown, ProcessorBasic, ProcessorDerived, Reflected.
  -ExtendedAttributeValues  Extended attribute values in UPPERCASE_COLUMN_NAME@UPPERCASE_TABLE_NAME=value syntax. Can be set multiple times.

  ========================= Copy points from another time-series:
  -SourceTimeSeries         Source time-series to copy. Prefix with [server2] or [server2:username2:password2] to copy from another server
  -SourceQueryFrom          Start time of extracted points in ISO8601 format.
  -SourceQueryTo            End time of extracted points

  ========================= Point-generator options:
  -StartTime                Start time of generated points, in ISO8601 format. [default: the current time]
  -PointInterval            Interval between generated points, in .NET TimeSpan format. [default: 00:01:00]
  -NumberOfPoints           Number of points to generate. If 0, use NumberOfPeriods [default: 0]
  -NumberOfPeriods          Number of waveform periods to generate. [default: 1]
  -WaveformType             Waveform to generate. One of Linear, SawTooth, SineWave, SquareWave. [default: SineWave]
  -WaveformOffset           Offset the generated waveform by this constant. [default: 0]
  -WaveformPhase            Phase within one waveform period [default: 0]
  -WaveformScalar           Scale the waveform by this amount [default: 1]
  -WaveformPeriod           Waveform period before repeating [default: 1440]
  -WaveFormTextX            Select the X values of the vectorized text
  -WaveFormTextY            Select the Y values of the vectorized text

  ========================= CSV parsing options:
  -CSV                      Parse the CSV file
  -CsvDateTimeField         CSV column index or name for combined date+time timestamps
  -CsvDateTimeFormat        Format of CSV date+time fields [default: ISO8601 format]
  -CsvDateOnlyField         CSV column index or name for date-only timestamps
  -CsvDateOnlyFormat        Format of CSV date-only fields
  -CsvTimeOnlyField         CSV column index or name for time-only timestamps
  -CsvTimeOnlyFormat        Format of CSV time-only fields
  -CsvDefaultTimeOfDay      Time of day value when no time field is used [default: 00:00]
  -CsvValueField            CSV column index or name for values
  -CsvGradeField            CSV column index or name for grade codes
  -CsvQualifiersField       CSV column index or name for qualifiers
  -CsvNotesField            CSV column index or name for notes
  -CsvTimezoneField         CSV column index or name for timezone
  -CsvComment               CSV comment lines begin with this prefix
  -CsvSkipRows              Number of CSV rows to skip before parsing [default: 0]
  -CsvSkipRowsAfterHeader   Number of CSV rows to skip after the header row, but before parsing [default: 0]
  -CsvHasHeaderRow          Does the CSV have a header row naming the columns. [default: true if any columns are referenced by name]
  -CsvHeaderStartsWith      A comma separated list of of the first expected header column names
  -CsvIgnoreInvalidRows     Ignore CSV rows that can't be parsed [default: False]
  -CsvRealign               Realign imported CSV points to the /StartTime value [default: False]
  -CsvRemoveDuplicatePoints Remove duplicate points in the CSV before appending. [default: True]
  -CsvDelimiter             Delimiter between CSV fields. (use %20 for space or %09 for tab) [default: ,]
  -CsvNanValue              Special value text used to represent NaN values
  -CsvFormat                Shortcut for known CSV formats. One of NG, 3X, PointZilla, NWIS. [default: NG]
  -ExcelSheetNumber         Excel worksheet number to parse [default to first sheet]
  -ExcelSheetName           Excel worksheet name to parse [default to first sheet]

  ========================= Timezone options:
  -Timezone                 The IANA timezone name. See https://en.wikipedia.org/wiki/List_of_tz_database_time_zones for details.
  -FindTimezones            Show all the known timezones matching the pattern
  -TimezoneAliases          Timezone aliases in sourceValue:mappedValue syntax. Can be set multiple times.

  ========================= DB parsing options:
  -DbType                   Database type. Should be one of: SqlServer, Postgres, MySql, Odbc
  -DbConnectionString       Database connection string. See https://connectionstrings.com for examples.
  -DbQuery                  SQL query to fetch the time-series points
  -DbNotesQuery             SQL query to fetch the time-series notes from StartTime, EndTime, and NoteText columns.

  ========================= CSV saving options:
  -SaveCsvPath              When set, saves the extracted/generated points to a CSV file. If only a directory is specified, an appropriate filename will be generated.
  -SaveNotesMode            Controls how extracted notes are save. Should be one of: Disabled, WithPoints, SeparateCsv [default: Disabled]
  -StopAfterSavingCsv       When true, stop after saving a CSV file, before appending any points. [default: False]

Use the @optionsFile syntax to read more options from a file.

  Each line in the file is treated as a command line option.
  Blank lines and leading/trailing whitespace is ignored.
  Comment lines begin with a # or // marker.
```
