<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<link rel="stylesheet" type="text/css" href="bc.css">
<script src="run_prettify.js" type="text/javascript"></script>
<!---
<script src="https://google-code-prettify.googlecode.com/svn/loader/run_prettify.js" type="text/javascript"></script>
-->
</head>

<!---


#dotnet #csharp
#fsharp #python
#grevit
#responsivedesign #typepad
#ah8 #augi #dotnet
#stingray #rendering
#3dweb #3dviewAPI #html5 #threejs #webgl #3d #mobile #vr #ecommerce
#Markdown #Fusion360 #Fusion360Hackathon
#javascript
#RestSharp #restAPI
#mongoosejs #mongodb #nodejs
#rtceur
#xaml
#3dweb #a360 #3dwebaccel #webgl @adskForge
@AutodeskReCap @Adsk3dsMax
#revitAPI #bim #aec #3dwebcoder #adsk #adskdevnetwrk @jimquanci @keanw
#au2015 #rtceur
#eraofconnection
#RMS @researchdigisus
@adskForge #3dwebaccel
#a360
@github

Revit API, Jeremy Tammik, akn_include

 #revitAPI #3dwebcoder @AutodeskRevit #bim #aec #adsk #adskdevnetwrk

Here is another bunch of issues addressed in
the Revit API discussion forum and
elsewhere in the past day or two
&ndash; The Building Coder blog source text and index
&ndash; Reloading add-in DLL for debugging without restart
&ndash; Drawing curves from a list of points
&ndash; Determining the distance of a family instance to the floor or elevation
&ndash; Deleting a PrintSetup or ViewSheetSetting...

-->

### Implementing the TrackChangesCloud External Event

Today, I'll discuss this morning's work:

- [TrackChanges Enhancement Idea](#2)
- [The TrackChangesCloud add-in](#3)
- [The TrackChangesCloud external event](#4)
- [Creating and raising an external event](#5)
- [Raising the external event from a separate thread](#6)
- [First test run](#7)
- [Trigger immediate execution by setting Revit foreground window](#8)
- [Complete external application module](#9)
- [Next steps](#10)
- [Download](#11)

I implemented a suggestion
for [tracking element modification](http://thebuildingcoder.typepad.com/blog/2016/01/tracking-element-modification.html) a
month ago and created
the [TrackChanges GitHub repository](https://github.com/jeremytammik/TrackChanges) to host its solution and source code.

That met with significant interest and triggered the following idea for an enhancement.

#### <a name="2"></a>TrackChanges Enhancement Idea

I am pondering an enhancement of this external command that I suggested to Tim Corneliussen in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160) thread
on [dynamic model update after loading family](http://forums.autodesk.com/t5/revit-api/dynamic-model-update-after-loading-family/m-p/6052310):

Tim says he needs to track changes, and wondered whether to use the DocumentChanged event or the dynamic model updater framework DMU to do so.

I suggested that he take a look at TrackChanges instead.

**Response:** Your solution looks really impressive. I haven't had the chance to implement the main fundamentals in my project yet. As a starting programmer, the concept of hash code is still new to me but it looks like the right way to go. My main concern is how it will affect the performance of the routine.

The main purpose of my tool is that each addition or modification will be registered by modifying some parameters including a "time-parameter" and "date-parameter". To do so, but correct me if I'm wrong, I need to trigger an event or use a DMU to determine when an element is added or modified.

Maybe I can use your snapshot technique combining it with a DMU. But doing so the DMU also collects a lot of data next to the snapshot routine. Is this necessary? Are there alternatives to avoid this sort of useless multiple data collecting?

Do elements themselves contain relevant information about their own creation or modification (perhaps a certain property that most people aren't aware of)? If so, I can use a single event (sort of like your suggestion on your blog), for example the DocumentSavingAs event. Last possible solution I can think of at the moment is a way to look even deeper in to the updaterdata/-information hoping it contains more general information about the addition or modification of the relevant elements.

Hoping that you'll understand the scenario I'm describing. For now I will try to use the snapshot routine combining it with a DMU and a viewactivating event. Last mentioned will be used to determine whether another document becomes active (when a user has opened multiple projects). I will place an update when I have successfully created a working solution to discuss the results with you fellow readers. If someone can tell me if this solution probably won't really work please do so.

**Answer:** Thank you very much for your appreciation.

I think the main characteristic of the modification tracker is simplicity, rather than impressiveness.

Of course, simplicity is much more impressive than impressiveness :-)

If you want to be notified on every single modification of an element and store that information immediately, then indeed you can and have to use either DMU or the DocumentChanged event.

The latter does not allow you to modify anything in the same transaction, though, whereas the former does.

If you want to guarantee that your date and time markers stored in Revit parameters are always up to date, immediately, then you need to use DMU.

But do you really need that?

You need to understand that DMU is complex and adds a significant burden to Revit, depending on how many elements trigger it, which in your case would be many.

Do you really need to keep track of the element modification on a split second-by-second basis?

Would it not be enough to track changes every minute, or every ten minutes?

If so, then you can vastly simplify your approach and vastly reduce the burden on Revit by using the modification tracker and completely avoiding DMU and the DocumentChanged event.

Just track changes based on snapshots taken every X minutes, for instance.

Regarding the issue of the hash code: that is a minor detail, and pretty irrelevant.

You can just store the full data. Depending on what criteria you use to define when an element has changed, you might need to store a lot of information for each element.

I suggested the hash code as a way to reduce and unify that data storage, but that has absolutely nothing to do with the fundamental concept.

To quote the original post: "We use the hash code to determine whether the state has been modified compared to a new element state snapshot made at a later time. We could obviously also store the entire original string representation instead of using a hash code. The hash code is small and handy, whereas the entire string contains all the original data. It is up to you to choose which you would like to use."

The hash code will not affect performance much, just reduce the memory used to cache the starting snapshot.

I would recommend thinking this through in depth and peace and quiet.

If you do not require split-second time slice data, I would avoid the DMU and DocumentChanged events, both, completely.

I am very much looking forward to hearing and discussing your further thoughts on this.


#### <a name="3"></a>The TrackChangesCloud Add-In

I am thinking of testing the suggestions I propose above myself.

My idea is:

- Trigger automatic snapshots of element modifications every couple of minutes.
- Store the results in a cloud-based database.

That will also represent another step in my explorations on connecting the desktop and the cloud.

I created a new add-in [TrackChangesCloud](https://github.com/jeremytammik/TrackChangesCloud) to explore them in.

I prefer to leave the original TrackChanges unmodified, since it clearly and simply demonstrates the snapshot idea.

It would be a shame to make it more confusing by adding the complexity required by the external application and event handling.

One way to trigger the snapshots at regular intervals, and probably the cleanest approach, is by implementing an external event.


#### <a name="4"></a>The TrackChangesCloud External Event

The Building Coder topic group
on [Idling and external events](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.28) is
growing pretty large and explains all the background information on this in full detail.

I implemented a skeleton for the external event named `ModificationLogger` that does nothing so far, except get raised when I say so.

Therefore, the implementation is completely trivial:

<pre class="code">
&nbsp; <span class="blue">class</span> <span class="teal">ModificationLogger</span> : <span class="teal">IExternalEventHandler</span>
&nbsp; {
&nbsp; &nbsp; <span class="gray">///</span><span class="green"> </span><span class="gray">&lt;summary&gt;</span>
&nbsp; &nbsp; <span class="gray">///</span><span class="green"> Required IExternalEventHandler interface </span>
&nbsp; &nbsp; <span class="gray">///</span><span class="green"> method returning a descriptive name.</span>
&nbsp; &nbsp; <span class="gray">///</span><span class="green"> </span><span class="gray">&lt;/summary&gt;</span>
&nbsp; &nbsp; <span class="blue">public</span> <span class="blue">string</span> GetName()
&nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; <span class="blue">return</span> <span class="maroon">&quot;TrackChangesCloud ModificationLogger&quot;</span>;
&nbsp; &nbsp; }
&nbsp;
&nbsp; &nbsp; <span class="gray">///</span><span class="green"> </span><span class="gray">&lt;summary&gt;</span>
&nbsp; &nbsp; <span class="gray">///</span><span class="green"> Execute method invoked by Revit via the </span>
&nbsp; &nbsp; <span class="gray">///</span><span class="green"> external event as a reaction to a call </span>
&nbsp; &nbsp; <span class="gray">///</span><span class="green"> to its Raise method.</span>
&nbsp; &nbsp; <span class="gray">///</span><span class="green"> </span><span class="gray">&lt;/summary&gt;</span>
&nbsp; &nbsp; <span class="blue">public</span> <span class="blue">void</span> Execute( <span class="teal">UIApplication</span> a )
&nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; <span class="teal">Util</span>.Log( <span class="maroon">&quot;ModificationLogger.Execute&quot;</span> );
&nbsp; &nbsp; }
&nbsp; }
</pre>

Later on, the `Execute` method will do more, of course, e.g. create a modification tracking snapshot and store it in the cloud.


#### <a name="5"></a>Creating and Raising an External Event

These two steps are both completely trivial one-liners, implemented in the external application module `App.cs`.

In the `OnStartup` method, I subscribe to the `ApplicationInitialized` event:

<pre class="code">
&nbsp; a.ControlledApplication.ApplicationInitialized
&nbsp; &nbsp; += OnApplicationInitialized;
</pre>

In the event handler, I create the external event by instantiating my implementation:

<pre class="code">
&nbsp; <span class="green">// Create our custom external event.</span>
&nbsp;
&nbsp; _event = <span class="teal">ExternalEvent</span>.Create(
&nbsp; &nbsp; <span class="blue">new</span> <span class="teal">ModificationLogger</span>() );
</pre>

To trigger the event, causing Revit to wait for an opportune moment and then call its `Execute` method, I call its `Raise` method:

<pre class="code">
&nbsp; _event.Raise();
</pre>

That's all there is to it.

But how do we ensure that the `Raise` call is executed at the times we wish?

For that I implemented a separate driver thread, which is also very simple.


#### <a name="6"></a>Raising the External Event from a Separate Thread

In order to control the regular raising of my external event, I can run an infinite triggering loop in a separate thread.

The loop is implemented in the following `TriggerModificationLogger` method:

<pre class="code">
&nbsp; <span class="gray">///</span><span class="green"> </span><span class="gray">&lt;summary&gt;</span>
&nbsp; <span class="gray">///</span><span class="green"> Trigger a modification tracker snapshot at </span>
&nbsp; <span class="gray">///</span><span class="green"> regular intervals. Relinquish control and wait </span>
&nbsp; <span class="gray">///</span><span class="green"> for the specified timeout period between each </span>
&nbsp; <span class="gray">///</span><span class="green"> snapshot. This method runs in a separate thread.</span>
&nbsp; <span class="gray">///</span><span class="green"> </span><span class="gray">&lt;/summary&gt;</span>
&nbsp; <span class="blue">static</span> <span class="blue">void</span> TriggerModificationLogger()
&nbsp; {
&nbsp; &nbsp; <span class="blue">while</span>( <span class="blue">true</span> )
&nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; ++_nSnapshots;
&nbsp;
&nbsp; &nbsp; &nbsp; <span class="teal">Util</span>.Log( <span class="blue">string</span>.Format(
&nbsp; &nbsp; &nbsp; &nbsp; <span class="maroon">&quot;TriggerModificationLogger snapshot {0}&quot;</span>,
&nbsp; &nbsp; &nbsp; &nbsp; _nSnapshots ) );
&nbsp;
&nbsp; &nbsp; &nbsp; _event.Raise();

&nbsp; &nbsp; &nbsp; <span class="green">// Wait and relinquish control </span>
&nbsp; &nbsp; &nbsp; <span class="green">// before next snapshot.</span>
&nbsp;
&nbsp; &nbsp; &nbsp; <span class="teal">Thread</span>.Sleep( _timeout );
&nbsp; &nbsp; }
&nbsp; }
</pre>

It endlessly loops, raising the external event at regular intervals specified by the `_timeout` constant.

I start the looping in the `ApplicationInitialized` event handler, directly after instantiating my external event:

<pre class="code">
&nbsp; <span class="blue">void</span> OnApplicationInitialized(
&nbsp; &nbsp; <span class="blue">object</span> sender,
&nbsp; &nbsp; <span class="teal">ApplicationInitializedEventArgs</span> e )
&nbsp; {
&nbsp; &nbsp; <span class="green">// Create our custom external event.</span>
&nbsp;
&nbsp; &nbsp; _event = <span class="teal">ExternalEvent</span>.Create(
&nbsp; &nbsp; &nbsp; <span class="blue">new</span> <span class="teal">ModificationLogger</span>() );
&nbsp;
&nbsp; &nbsp; <span class="green">// Start a thread to raise it regularly.</span>
&nbsp;
&nbsp; &nbsp; _thread = <span class="blue">new</span> <span class="teal">Thread</span>(
&nbsp; &nbsp; &nbsp; TriggerModificationLogger );
&nbsp;
&nbsp; &nbsp; _thread.Start();
&nbsp; }
</pre>


#### <a name="7"></a>First Test Run

This works perfectly, with one teeny weeny little flaw: even though the external event is raised at the moment I specify, the `Execute` method is not called until the Revit window is activated:

<pre>
TrackChanges 10:59:14.067 TriggerModificationLogger snapshot 1
TrackChanges 10:59:14.635 ModificationLogger.Execute
TrackChanges 11:00:14.074 TriggerModificationLogger snapshot 2
*manually activated Revit*
TrackChanges 11:00:44.050 ModificationLogger.Execute
TrackChanges 11:01:14.075 TriggerModificationLogger snapshot 3
*manually activated Revit*
TrackChanges 11:01:23.877 ModificationLogger.Execute
</pre>

This may or may not be a problem, depending on the intended use.

Maybe you don't care if the external event is not triggered as long as Revit is in the background.

However, this is quite easy to rectify, so let's do so.


#### <a name="8"></a>Trigger Immediate Execution by Setting Revit Foreground Window

I implemented the following helper method to check whether Revit is the current foreground window.

If it is not, it is temporarily brought to the foreground using the Windows API:

<pre class="code">
&nbsp; <span class="green">// DLL imports from user32.dll to set focus to</span>
&nbsp; <span class="green">// Revit to force it to forward the external event</span>
&nbsp; <span class="green">// Raise to actually call the external event </span>
&nbsp; <span class="green">// Execute.</span>
&nbsp;
&nbsp; <span class="gray">///</span><span class="green"> </span><span class="gray">&lt;summary&gt;</span>
&nbsp; <span class="gray">///</span><span class="green"> The GetForegroundWindow function returns a </span>
&nbsp; <span class="gray">///</span><span class="green"> handle to the foreground window.</span>
&nbsp; <span class="gray">///</span><span class="green"> </span><span class="gray">&lt;/summary&gt;</span>
&nbsp; [<span class="teal">DllImport</span>( <span class="maroon">&quot;user32.dll&quot;</span> )]
&nbsp; <span class="blue">static</span> <span class="blue">extern</span> <span class="teal">IntPtr</span> GetForegroundWindow();
&nbsp;
&nbsp; <span class="gray">///</span><span class="green"> </span><span class="gray">&lt;summary&gt;</span>
&nbsp; <span class="gray">///</span><span class="green"> Move the window associated with the passed </span>
&nbsp; <span class="gray">///</span><span class="green"> handle to the front.</span>
&nbsp; <span class="gray">///</span><span class="green"> </span><span class="gray">&lt;/summary&gt;</span>
&nbsp; [<span class="teal">DllImport</span>( <span class="maroon">&quot;user32.dll&quot;</span> )]
&nbsp; <span class="blue">static</span> <span class="blue">extern</span> <span class="blue">bool</span> SetForegroundWindow(
&nbsp; &nbsp; <span class="teal">IntPtr</span> hWnd );
&nbsp;
&nbsp; <span class="blue">static</span> <span class="blue">void</span> SetFocusToRevit()
&nbsp; {
&nbsp; &nbsp; <span class="teal">IntPtr</span> hRevit = <span class="teal">ComponentManager</span>.ApplicationWindow;
&nbsp; &nbsp; <span class="teal">IntPtr</span> hBefore = GetForegroundWindow();
&nbsp;
&nbsp; &nbsp; <span class="blue">if</span>( hBefore != hRevit )
&nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; SetForegroundWindow( hRevit );
&nbsp; &nbsp; &nbsp; SetForegroundWindow( hBefore );
&nbsp; &nbsp; }
&nbsp; }
</pre>

Now, if I add a call to `SetFocusToRevit` directly after calling `Raise`, the `Execute` method is invoked immediately, regardless of whether Revit is in the background or not:

<pre>
TrackChanges 11:26:23.431 TriggerModificationLogger snapshot 1
TrackChanges 11:26:24.047 ModificationLogger.Execute
TrackChanges 11:27:23.445 TriggerModificationLogger snapshot 2
TrackChanges 11:27:23.499 ModificationLogger.Execute
TrackChanges 11:28:23.506 TriggerModificationLogger snapshot 3
TrackChanges 11:28:23.560 ModificationLogger.Execute
TrackChanges 11:29:23.569 TriggerModificationLogger snapshot 4
TrackChanges 11:29:23.623 ModificationLogger.Execute
TrackChanges 11:30:23.638 TriggerModificationLogger snapshot 5
TrackChanges 11:30:23.697 ModificationLogger.Execute
TrackChanges 11:31:23.706 TriggerModificationLogger snapshot 6
TrackChanges 11:31:23.790 ModificationLogger.Execute
</pre>

No manual intervention required.

However, the screen will flash!

If the purpose of this tool is to track modifications made to the BIM by end users, then obviously they will have the Revit window up and visible, so the call to `SetFocusToRevit` can be removed again.


#### <a name="9"></a>Complete External Application Module

To show how the external event and thread handling work together, here is the complete implementation of the external application module `App.cs`:

<pre class="code">
<span class="blue">class</span> <span class="teal">App</span> : <span class="teal">IExternalApplication</span>
{
&nbsp; <span class="gray">///</span><span class="green"> </span><span class="gray">&lt;summary&gt;</span>
&nbsp; <span class="gray">///</span><span class="green"> Store the external event.</span>
&nbsp; <span class="gray">///</span><span class="green"> </span><span class="gray">&lt;/summary&gt;</span>
&nbsp; <span class="blue">static</span> <span class="teal">ExternalEvent</span> _event = <span class="blue">null</span>;
&nbsp;
&nbsp; <span class="gray">///</span><span class="green"> </span><span class="gray">&lt;summary&gt;</span>
&nbsp; <span class="gray">///</span><span class="green"> Separate thread running a timer to trigger</span>
&nbsp; <span class="gray">///</span><span class="green"> the modification tracker at regular intervals.</span>
&nbsp; <span class="gray">///</span><span class="green"> </span><span class="gray">&lt;/summary&gt;</span>
&nbsp; <span class="blue">static</span> <span class="teal">Thread</span> _thread = <span class="blue">null</span>;
&nbsp;
&nbsp; <span class="gray">///</span><span class="green"> </span><span class="gray">&lt;summary&gt;</span>
&nbsp; <span class="gray">///</span><span class="green"> Count total number of modification tracker</span>
&nbsp; <span class="gray">///</span><span class="green"> snapshots taken so far in this session.</span>
&nbsp; <span class="gray">///</span><span class="green"> </span><span class="gray">&lt;/summary&gt;</span>
&nbsp; <span class="blue">static</span> <span class="blue">int</span> _nSnapshots = 0;
&nbsp;
&nbsp; <span class="blue">static</span> <span class="blue">int</span> _timeout_minutes = 1;
&nbsp;
&nbsp; <span class="gray">///</span><span class="green"> </span><span class="gray">&lt;summary&gt;</span>
&nbsp; <span class="gray">///</span><span class="green"> Number of milliseconds to wait and relinquish</span>
&nbsp; <span class="gray">///</span><span class="green"> CPU control before next snapshot.</span>
&nbsp; <span class="gray">///</span><span class="green"> </span><span class="gray">&lt;/summary&gt;</span>
&nbsp; <span class="blue">static</span> <span class="blue">int</span> _timeout = 1000 * 60 * _timeout_minutes;
&nbsp;
<span class="blue">&nbsp; #region</span> SetFocusToRevit
&nbsp; <span class="green">// DLL imports from user32.dll to set focus to</span>
&nbsp; <span class="green">// Revit to force it to forward the external event</span>
&nbsp; <span class="green">// Raise to actually call the external event </span>
&nbsp; <span class="green">// Execute.</span>
&nbsp;
&nbsp; <span class="gray">///</span><span class="green"> </span><span class="gray">&lt;summary&gt;</span>
&nbsp; <span class="gray">///</span><span class="green"> The GetForegroundWindow function returns a </span>
&nbsp; <span class="gray">///</span><span class="green"> handle to the foreground window.</span>
&nbsp; <span class="gray">///</span><span class="green"> </span><span class="gray">&lt;/summary&gt;</span>
&nbsp; [<span class="teal">DllImport</span>( <span class="maroon">&quot;user32.dll&quot;</span> )]
&nbsp; <span class="blue">static</span> <span class="blue">extern</span> <span class="teal">IntPtr</span> GetForegroundWindow();
&nbsp;
&nbsp; <span class="gray">///</span><span class="green"> </span><span class="gray">&lt;summary&gt;</span>
&nbsp; <span class="gray">///</span><span class="green"> Move the window associated with the passed </span>
&nbsp; <span class="gray">///</span><span class="green"> handle to the front.</span>
&nbsp; <span class="gray">///</span><span class="green"> </span><span class="gray">&lt;/summary&gt;</span>
&nbsp; [<span class="teal">DllImport</span>( <span class="maroon">&quot;user32.dll&quot;</span> )]
&nbsp; <span class="blue">static</span> <span class="blue">extern</span> <span class="blue">bool</span> SetForegroundWindow(
&nbsp; &nbsp; <span class="teal">IntPtr</span> hWnd );
&nbsp;
&nbsp; <span class="blue">static</span> <span class="blue">void</span> SetFocusToRevit()
&nbsp; {
&nbsp; &nbsp; <span class="teal">IntPtr</span> hRevit = <span class="teal">ComponentManager</span>.ApplicationWindow;
&nbsp; &nbsp; <span class="teal">IntPtr</span> hBefore = GetForegroundWindow();
&nbsp;
&nbsp; &nbsp; <span class="blue">if</span>( hBefore != hRevit )
&nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; SetForegroundWindow( hRevit );
&nbsp; &nbsp; &nbsp; SetForegroundWindow( hBefore );
&nbsp; &nbsp; }
&nbsp; }
<span class="blue">&nbsp; #endregion</span> <span class="green">// SetFocusToRevit</span>
&nbsp;
&nbsp; <span class="gray">///</span><span class="green"> </span><span class="gray">&lt;summary&gt;</span>
&nbsp; <span class="gray">///</span><span class="green"> Trigger a modification tracker snapshot at </span>
&nbsp; <span class="gray">///</span><span class="green"> regular intervals. Relinquish control and wait </span>
&nbsp; <span class="gray">///</span><span class="green"> for the specified timeout period between each </span>
&nbsp; <span class="gray">///</span><span class="green"> snapshot. This method runs in a separate thread.</span>
&nbsp; <span class="gray">///</span><span class="green"> </span><span class="gray">&lt;/summary&gt;</span>
&nbsp; <span class="blue">static</span> <span class="blue">void</span> TriggerModificationLogger()
&nbsp; {
&nbsp; &nbsp; <span class="blue">while</span>( <span class="blue">true</span> )
&nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; ++_nSnapshots;
&nbsp;
&nbsp; &nbsp; &nbsp; <span class="teal">Util</span>.Log( <span class="blue">string</span>.Format(
&nbsp; &nbsp; &nbsp; &nbsp; <span class="maroon">&quot;TriggerModificationLogger snapshot {0}&quot;</span>,
&nbsp; &nbsp; &nbsp; &nbsp; _nSnapshots ) );
&nbsp;
&nbsp; &nbsp; &nbsp; _event.Raise();
&nbsp;
&nbsp; &nbsp; &nbsp; <span class="green">// Set focus to Revit for a moment.</span>
&nbsp; &nbsp; &nbsp; <span class="green">// Without this, Revit will not forward the </span>
&nbsp; &nbsp; &nbsp; <span class="green">// event Raise to the external event handler </span>
&nbsp; &nbsp; &nbsp; <span class="green">// Execute method until the Revit window is</span>
&nbsp; &nbsp; &nbsp; <span class="green">// activated. This causes the screen to flash.</span>
&nbsp;
&nbsp; &nbsp; &nbsp; SetFocusToRevit();
&nbsp;
&nbsp; &nbsp; &nbsp; <span class="green">// Wait and relinquish control </span>
&nbsp; &nbsp; &nbsp; <span class="green">// before next snapshot.</span>
&nbsp;
&nbsp; &nbsp; &nbsp; <span class="teal">Thread</span>.Sleep( _timeout );
&nbsp; &nbsp; }
&nbsp; }
&nbsp;
&nbsp; <span class="blue">public</span> <span class="teal">Result</span> OnStartup( <span class="teal">UIControlledApplication</span> a )
&nbsp; {
&nbsp; &nbsp; a.ControlledApplication.ApplicationInitialized
&nbsp; &nbsp; &nbsp; += OnApplicationInitialized;
&nbsp;
&nbsp; &nbsp; <span class="blue">return</span> <span class="teal">Result</span>.Succeeded;
&nbsp; }
&nbsp;
&nbsp; <span class="blue">void</span> OnApplicationInitialized(
&nbsp; &nbsp; <span class="blue">object</span> sender,
&nbsp; &nbsp; <span class="teal">ApplicationInitializedEventArgs</span> e )
&nbsp; {
&nbsp; &nbsp; <span class="green">// Create our custom external event.</span>
&nbsp;
&nbsp; &nbsp; _event = <span class="teal">ExternalEvent</span>.Create(
&nbsp; &nbsp; &nbsp; <span class="blue">new</span> <span class="teal">ModificationLogger</span>() );
&nbsp;
&nbsp; &nbsp; <span class="green">// Start a thread to raise it regularly.</span>
&nbsp;
&nbsp; &nbsp; _thread = <span class="blue">new</span> <span class="teal">Thread</span>(
&nbsp; &nbsp; &nbsp; TriggerModificationLogger );
&nbsp;
&nbsp; &nbsp; _thread.Start();
&nbsp; }
&nbsp;
&nbsp; <span class="blue">public</span> <span class="teal">Result</span> OnShutdown( <span class="teal">UIControlledApplication</span> a )
&nbsp; {
&nbsp; &nbsp; _thread.Abort();
&nbsp; &nbsp; _thread = <span class="blue">null</span>;
&nbsp; &nbsp; _event.Dispose();
&nbsp;
&nbsp; &nbsp; <span class="blue">return</span> <span class="teal">Result</span>.Succeeded;
&nbsp; }
}
</pre>


#### <a name="10"></a>Next Steps

The next steps are obvious:

- Implement the cloud database to hold the modification tracking data.
- Implement the database upload.

Both of these are clearly and extensively demonstrated by the cloud storage approach developed for
the [FireRatingCloud project](https://github.com/jeremytammik/FireRatingCloud).


#### <a name="11"></a>Download

This project is hosted in
the [TrackChanges GitHub repository](https://github.com/jeremytammik/TrackChanges),
and the version discussed above
is [release 2016.0.0.4](https://github.com/jeremytammik/TrackChangesCloud/releases/tag/2016.0.0.4).

<center>
<img src="img/track_changes_diff.png" alt="TrachChanges diff" width="340">
</center>
