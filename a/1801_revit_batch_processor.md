<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<link rel="stylesheet" type="text/css" href="bc.css">
<script src="https://cdn.rawgit.com/google/code-prettify/master/loader/run_prettify.js" type="text/javascript"></script>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</head>

<!---

- Revit Batch Processor (RBP)
  https://github.com/bvn-architecture/RevitBatchProcessor
  pointed out by
  Automatic Heating and cooling load analysis with Revit API
  https://forums.autodesk.com/t5/revit-api-forum/automatic-heating-and-cooling-load-analysis-with-revit-api/m-p/9149375
  There are so many truly wonderful pieces of software sitting around there that I am anaware of, real works of art, pinnacles of perfection, that I only happen upon by chance.
  In this case, the ... thread on ... pointed out the ...

twitter:

The Revit Batch Processor RBP drives mass execution of #RevitAPI add-ins and #DynamoBim scripts! @AutodeskForge @AutodeskRevit #bim #ForgeDevCon http://bit.ly/batchprocessor

There are so many truly wonderful pieces of software sitting around there that I am unaware of, real works of art, pinnacles of perfection, that I only happen upon by chance
&ndash; Revit Batch Processor (RBP)
&ndash; Latest version
&ndash; FAQ
&ndash; Use cases
&ndash; Features
&ndash; Unlimited power...

&ndash; 
...

linkedin:

The Revit Batch Processor RBP drives mass execution of #RevitAPI add-ins and #DynamoBim scripts!

http://bit.ly/batchprocessor

There are so many truly wonderful pieces of software sitting around there that I am unaware of, real works of art, pinnacles of perfection, that I only happen upon by chance:

- Revit Batch Processor (RBP)
- Latest version
- FAQ
- Use cases
- Features
- Unlimited power...

#bim #DynamoBim #ForgeDevCon #Revit #API #IFC #SDK #AI #VisualStudio #Autodesk #AEC #adsk

the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread

<p style="font-size: 80%; font-style:italic"></p>

-->

### The Revit Batch Processor RBP

There are so many truly wonderful pieces of software sitting around there that I am unaware of, real works of art, pinnacles of perfection, that I only happen upon by chance.

In this case, *@alexandrecafarofr* happens to mention a powerful utility in 
his [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [automatic heating and cooling load analysis](https://forums.autodesk.com/t5/revit-api-forum/automatic-heating-and-cooling-load-analysis-with-revit-api/m-p/9149375):

The [Revit Batch Processor (RBP)](https://github.com/bvn-architecture/RevitBatchProcessor) a project maintained
by [Daniel Rumery](https://github.com/DanRumery), ex-[BVN](http://www.bvn.com.au):

- [Revit Batch Processor (RBP)](#2)
- [Latest version](#3)
- [FAQ](#4)
- [Use cases](#5)
- [Features](#6)
- [Unlimited power](#7)
- [Addendum &ndash; questions](#8)

<center>
<img src="img/captain_dan.jpg" alt="Captain Dan" width="180"> <!--460-->
</center>

Here is an excerpt of the GitHub project readme to make your mouth water:

####<a name="2"></a> Revit Batch Processor (RBP)

Fully automated batch processing of Revit files with your own Python or Dynamo task scripts!

####<a name="3"></a> Latest version

[Installer for Revit Batch Processor v1.5.3](https://github.com/bvn-architecture/RevitBatchProcessor/releases/download/v1.5.3/RevitBatchProcessorSetup.exe)

See the [Releases](https://github.com/bvn-architecture/RevitBatchProcessor/releases) page for more information.

####<a name="4"></a> FAQ

See the [Revit Batch Processor FAQ](https://github.com/bvn-architecture/RevitBatchProcessor/wiki/Revit-Batch-Processor-FAQ).

####<a name="5"></a> Use Cases

This tool doesn't _do_ any of these things, but it _allows_ you to do them:

- Open all the Revit files across your Revit projects and run a health-check script against them. Keeping an eye on the health and performance of many Revit files is time-consuming. You could use this to check in on all your files daily and react to problems before they get too gnarly.
- Perform project and family audits across your Revit projects.
- Run large scale queries against many Revit files.
- Mine data from your Revit projects for analytics or machine learning projects.
- Automated milestoning of Revit projects.
- Automated housekeeping tasks (e.g. place elements on appropriate worksets)
- Batch upgrading of Revit projects and family files.
- Testing your own Revit API scripts and Revit addins against a variety of Revit models and families in an automated manner.
- Essentially anything you can do to one Revit file with the Revit API or a Dynamo script, you can now do to many!

<center>
<img src="img/BatchRvt_Screenshot.png" alt="Revit batch processor user interface" width="800"> <!--1010-->
</center>


####<a name="6"></a> Features

- Batch processing of Revit files (.rvt and .rfa files) using either a specific version of Revit or a version that matches the version of Revit the file was saved in. Currently supports processing files in Revit versions 2015 through 2020. (Of course, the required version of Revit must be installed!)
- Custom task scripts written in Python or Dynamo! Python scripts have full access to the Revit API. Dynamo scripts can of course do whatever Dynamo can do :)
- Option to create a new Python task script at the click of a button that contains the minimal amount of code required for the custom task script to operate on an opened Revit file. The new task script can then easily be extended to do some useful work. It can even load and execute your existing functions in a C# DLL (see [Executing functions in a C# DLL](#executing-functions-in-a-c-dll)).
- Option for custom pre- and post-processing task scripts. Useful if the overall batch processing task requires some additional setup / tear down work to be done.
- Central file processing options (Create a new local file, Detach from central).
- Option to process files (of the same Revit version) in the same Revit session, or to process each file in its own Revit session. The latter is useful if Revit happens to crash during processing, since this won't block further processing.
- Automatic Revit dialog / message box handling. These, in addition to Revit error messages are handled and logged to the GUI console. This makes the batch processor very likely to complete its tasks without any user intervention required!
- Ability to import and export settings. This feature combined with the simple [command-line interface](#command-line-interface) allows for batch processing tasks to be setup to run automatically on a schedule (using the Windows Task Scheduler) without the GUI.
- Generate a .txt-based list of Revit model file paths compatible with RBP. The *New List* button in the GUI will prompt for a folder path to scan for Revit files. Optionally you can specify the type of Revit files to scan for and also whether to include subfolders in the scan.

####<a name="7"></a> Unlimited Power

> "With great power come great responsibility"
[-- Spiderman](https://quoteinvestigator.com/2015/07/23/great-power/)

This tool enables you to do things with Revit files on a very large scale. Because of this ability, Python or Dynamo scripts that make modifications to Revit files (esp. workshared files) should be developed with the utmost care! You will need to be confident in your ability to write Python or Dynamo scripts that won't ruin your files en-masse. The Revit Batch Processor's 'Detach from Central' option should be used both while testing and for scripts that do not explicitly depend on working with a live workshared Central file.

Ever so many thanks to Dan for implementing, sharing, and, above all, so wonderfully documenting and maintaining this very impressive and powerful application!

####<a name="8"></a> Addendum &ndash; Questions

We have discussed similar batch processing topics here in the past, e.g.:

- [Batch processing Revit documents](https://thebuildingcoder.typepad.com/blog/2015/08/batch-processing-dwfx-links-and-future-proofing.html#4)
- [Batch processing Revit families and documents](https://thebuildingcoder.typepad.com/blog/2019/04/batch-processing-and-aspects-of-asstringvalue.html#2)

Based on that, I raised [issue #51](https://github.com/bvn-architecture/RevitBatchProcessor/issues/51) to
ask some important questions on failure handling and recovery:

1. Are the actions taken automatically logged?
&ndash; Why? In case of a problem, it might be handy to know how far you got successfully before the problem appeared, so that you know where to pick up again.
2. Is the Revit.exe health monitored?
&ndash; Why? Well, Revit is not built for batch processing, so one might expect it to crash and die if it is force fed too many files in a single session.
3. Is Revit restarted for each new document? That would reduce the risk of the aforementioned point.

Dan very kindly responds:
 
1. There is a log file that records the console output seen in the GUI. By default, log files by default are written to the folder %APPDATA%\BatchRvt. It logs stuff like process Revit startup and monitoring, files processed, file actions (open, closing, detach, file type, path, etc.). Any custom actions performed by a script would be left up to the script to record action-specific progress. If using a Python script, there is a utility function `Output` for logging to RBP's console. Dynamo scripts need their own mechanism for logging.
2. The Revit process(es) are monitored to detect process exit and process busy events. This info is logged to the console. If a Revit session crashes, RBP starts up a fresh Revit session for any files that remain to be processed (so only the Revit file that lead to the crash will be skipped). Indeed, I found this can happen when processing a large amount of Revit files, hence the recovery mechanism. You'll see in the UI there is an option to process the files in the same Revit session (when possible) or process every file in its own session; as far as crash recovery is concerned, however, the behaviour is the same. Note that Dynamo scripts are always made to run in a separate Revit session per file due to some limitations I encountered when closing an active UI document. Python scripts don't have this limitation and hence will process a lot faster!
3. See 2) :)

There is also a per-file time-out feature that can be used to terminate processing of a file if the time limit is reached.
In that scenario, the Revit process is forcibly terminated and a fresh session started up to process the remaining files.