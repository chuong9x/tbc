<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<link rel="stylesheet" type="text/css" href="bc.css">
<script src="https://cdn.rawgit.com/google/code-prettify/master/loader/run_prettify.js" type="text/javascript"></script>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</head>

<!---

- /a/doc/revit/tbc/git/a/img/panel_boards_api_pole_breaker_count.jpg
  
- [A Guide to Console Commands](https://css-tricks.com/a-guide-to-console-commands)

twitter:

Two short notes on splitting pipes retaining connections and headless Revit, launching it with no user interface with the #RevitAPI #DynamoBim @AutodeskForge @AutodeskRevit #bim #ForgeDevCon http://bit.ly/rvtheadless

Two short notes on splitting pipes retaining connections and headless Revit, launching it with no user interface...

&ndash; 
...

linkedin:

Two short notes on splitting pipes retaining connections and headless Revit, launching it with no user interface with the #RevitAPI

http://bit.ly/rvtheadless

#bim #DynamoBim #ForgeDevCon #Revit #API #IFC #SDK #AI #VisualStudio #Autodesk #AEC #adsk

the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread

<center>
<img src="img/" alt="" title="" width="100"/>
<p style="font-size: 80%; font-style:italic"></p>
</center>

-->

### Change Room Level


#### <a name="2"></a> 

After a year of discussion in 
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [how to change the associated level of rooms](https://forums.autodesk.com/t5/revit-api-forum/how-to-change-the-associated-level-of-rooms/m-p/9374610),
Sean Page now shares an amazing solution, to quote Mahshid Motie, who raised it in the first place:

**Question:** I want to change the associated level of rooms to have them on the right building story.
Both using C# and in the user interface, the level parameter of the rooms is read-only and cannot be modified.

So far, I can only solve this by by copying, pasting, and deleting.

**Answer:** The concept of this intrigued me, so I set off on a way to make this happen.
I think I have an OK solution that does not involve groups or really any work around at all.

I don't know of any way to change the level directly.

However, if you delete a room, you get a warning that it is deleted and still exists in the project &ndash; the 'not placed' message in room schedule.

Now when you place a new room at the correct level, don't place an entirely new room, but pick the correct unplaced room:

<center>
<img src="img/new_unplaced_room.png" alt="" title="" width="800"/> <!-- 1264 -->
</center>

It will retain all the properties of the deleted one, including the room number!

Concept:

- Select the rooms to move
- Select the level to move them too
- Press OK
- Store the Room Location Point
- Unplace the Room
- Use a Failure Handler to bypass the "Unplaced room in model" warning
- Get the Plan Topology for the level selected
- Find an empty circuit in the Topology
- There has to be at least one empty circuit (Room)
- Replace the room
- Move the room to the location it was in on the other level

Here is the command code:

<pre class="prettyprint">using Autodesk.Revit.DB.Architecture;
using Autodesk.Revit.DB;
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace MyApp.Testing
{
  public partial class TestForm : System.Windows.Forms.Form
  {
    Document doc;
    public TestForm(Document _doc)
    {
      InitializeComponent();
      doc = _doc;
    }

    private void btnClose_Click(object sender, EventArgs e)
    {
      Close();
    }

    private void TestForm_Load(object sender, EventArgs e)
    {
      foreach (Room room in Helpers.Collectors.ByCategory(doc, BuiltInCategory.OST_Rooms))
      {
        if (room.Area > 0 && room.Location != null)
        {
          ListViewItem item = listView1.Items.Add(room.Number + " - " + room.Name);
          item.Tag = room;
        }
      }

      SortedList<string, Level> levels = new SortedList<string, Level>();
      foreach (Level level in Helpers.Collectors.ByCategoryNotElementType(doc, BuiltInCategory.OST_Levels))
      {
        levels.Add(level.Name, level);
      }

      cboLevel.DataSource = levels.ToList();
      cboLevel.DisplayMember = "Key";
      cboLevel.ValueMember = "Value";
    }

    private void btmOK_Click(object sender, EventArgs e)
    {
      try
      {
        using (TransactionGroup transGroup = new TransactionGroup(doc))
        {
          transGroup.Start("Move Rooms");
          foreach (ListViewItem item in listView1.CheckedItems)
          {
            Room room = (Room)item.Tag;
            LocationPoint rmPT = room.Location as LocationPoint;
            XYZ roomPoint = rmPT.Point;
            double offset = room.LimitOffset;
            using (Transaction trans1 = new Transaction(doc))
            {
              trans1.Start("Unplace Room");
              FailureHandlingOptions failOptions = trans1.GetFailureHandlingOptions();
              failOptions.SetFailuresPreprocessor(new UnplacedRoomWarning());
              trans1.SetFailureHandlingOptions(failOptions);

              room.Unplace();
              trans1.Commit();
            }

            using (Transaction trans2 = new Transaction(doc))
            {
              trans2.Start("New Room");
              Level level = (Level)cboLevel.SelectedValue;
              PlanTopology topo = doc.get_PlanTopology(level);
              PlanCircuitSet circuits = topo.Circuits;

              foreach (PlanCircuit circuit in circuits)
              {
                if (!circuit.IsRoomLocated)
                {
                  Room newRoom = doc.Create.NewRoom(room, circuit);
                  newRoom.get_Parameter(BuiltInParameter.ROOM_UPPER_LEVEL).Set(level.Id);
                  newRoom.LimitOffset = offset;
                  XYZ move = new XYZ(roomPoint.X - rmPT.Point.X, roomPoint.Y - rmPT.Point.Y, rmPT.Point.Z);
                  ElementTransformUtils.MoveElement(doc, newRoom.Id, move);
                  break;
                }
              }
              trans2.Commit();
            }
          }
          transGroup.Assimilate();
          Close();
        }
      }
      catch (Exception ex)
      {
        MessageBox.Show(ex.ToString());
      }
    }
  }

  public class UnplacedRoomWarning : IFailuresPreprocessor
  {
    public FailureProcessingResult PreprocessFailures(FailuresAccessor failuresAccessor)
    {
      IList<FailureMessageAccessor> failures = new List<FailureMessageAccessor>();
      failures = failuresAccessor.GetFailureMessages();

      foreach (FailureMessageAccessor failure in failures)
      {
        if (failure.GetSeverity() == FailureSeverity.Warning)
        {
          if (failure.GetFailureDefinitionId() == BuiltInFailures.RoomFailures.RoomUnplaceWarning || failure.GetFailureDefinitionId() == BuiltInFailures.RoomFailures.RoomHeightNegative)
          {
            failuresAccessor.DeleteWarning(failure);
          }
        }
        else
        {
          failuresAccessor.ResolveFailure(failure);
          return FailureProcessingResult.ProceedWithCommit;
        }
      }
      return FailureProcessingResult.Continue;
    }
  }
}
</pre>

The simple Win Form layout:


<center>
<img src="img/select_placed_rooms.png" alt="Select placed rooms" title="Select placed rooms" width="400"/>
</center>

Here is the Win Form designer code:

<pre class="prettyprint">
namespace MyApp.Testing
{
  partial class TestForm
  {
    /// <summary>
    /// Required designer variable.
    /// </summary>
    private System.ComponentModel.IContainer components = null;

    /// <summary>
    /// Clean up any resources being used.
    /// </summary>
    /// <param name="disposing">true if managed resources should be disposed; otherwise, false.</param>
    protected override void Dispose(bool disposing)
    {
      if (disposing && (components != null))
      {
        components.Dispose();
      }
      base.Dispose(disposing);
    }

    #region Windows Form Designer generated code

    /// <summary>
    /// Required method for Designer support - do not modify
    /// the contents of this method with the code editor.
    /// </summary>
    private void InitializeComponent()
    {
      this.btnClose = new System.Windows.Forms.Button();
      this.btmOK = new System.Windows.Forms.Button();
      this.listView1 = new System.Windows.Forms.ListView();
      this.columnHeader3 = ((System.Windows.Forms.ColumnHeader)(new System.Windows.Forms.ColumnHeader()));
      this.cboLevel = new System.Windows.Forms.ComboBox();
      this.label1 = new System.Windows.Forms.Label();
      this.label2 = new System.Windows.Forms.Label();
      this.SuspendLayout();
      // 
      // btnClose
      // 
      this.btnClose.Anchor = ((System.Windows.Forms.AnchorStyles)((System.Windows.Forms.AnchorStyles.Bottom | System.Windows.Forms.AnchorStyles.Right)));
      this.btnClose.Location = new System.Drawing.Point(216, 526);
      this.btnClose.Name = "btnClose";
      this.btnClose.Size = new System.Drawing.Size(75, 23);
      this.btnClose.TabIndex = 0;
      this.btnClose.Text = "Close";
      this.btnClose.UseVisualStyleBackColor = true;
      this.btnClose.Click += new System.EventHandler(this.btnClose_Click);
      // 
      // btmOK
      // 
      this.btmOK.Anchor = ((System.Windows.Forms.AnchorStyles)((System.Windows.Forms.AnchorStyles.Bottom | System.Windows.Forms.AnchorStyles.Right)));
      this.btmOK.Location = new System.Drawing.Point(297, 526);
      this.btmOK.Name = "btmOK";
      this.btmOK.Size = new System.Drawing.Size(75, 23);
      this.btmOK.TabIndex = 1;
      this.btmOK.Text = "OK";
      this.btmOK.UseVisualStyleBackColor = true;
      this.btmOK.Click += new System.EventHandler(this.btmOK_Click);
      // 
      // listView1
      // 
      this.listView1.Anchor = ((System.Windows.Forms.AnchorStyles)((((System.Windows.Forms.AnchorStyles.Top | System.Windows.Forms.AnchorStyles.Bottom) 
      | System.Windows.Forms.AnchorStyles.Left) 
      | System.Windows.Forms.AnchorStyles.Right)));
      this.listView1.CheckBoxes = true;
      this.listView1.Columns.AddRange(new System.Windows.Forms.ColumnHeader[] {
      this.columnHeader3});
      this.listView1.FullRowSelect = true;
      this.listView1.HeaderStyle = System.Windows.Forms.ColumnHeaderStyle.None;
      this.listView1.HideSelection = false;
      this.listView1.Location = new System.Drawing.Point(12, 27);
      this.listView1.Margin = new System.Windows.Forms.Padding(3, 5, 3, 5);
      this.listView1.Name = "listView1";
      this.listView1.Size = new System.Drawing.Size(360, 464);
      this.listView1.Sorting = System.Windows.Forms.SortOrder.Ascending;
      this.listView1.TabIndex = 2;
      this.listView1.UseCompatibleStateImageBehavior = false;
      this.listView1.View = System.Windows.Forms.View.Details;
      // 
      // columnHeader3
      // 
      this.columnHeader3.Text = "";
      this.columnHeader3.Width = 350;
      // 
      // cboLevel
      // 
      this.cboLevel.FormattingEnabled = true;
      this.cboLevel.Location = new System.Drawing.Point(78, 499);
      this.cboLevel.Name = "cboLevel";
      this.cboLevel.Size = new System.Drawing.Size(294, 21);
      this.cboLevel.TabIndex = 3;
      // 
      // label1
      // 
      this.label1.AutoSize = true;
      this.label1.Location = new System.Drawing.Point(12, 502);
      this.label1.Name = "label1";
      this.label1.Size = new System.Drawing.Size(60, 13);
      this.label1.TabIndex = 4;
      this.label1.Text = "Pick Level:";
      // 
      // label2
      // 
      this.label2.AutoSize = true;
      this.label2.Location = new System.Drawing.Point(12, 9);
      this.label2.Name = "label2";
      this.label2.Size = new System.Drawing.Size(112, 13);
      this.label2.TabIndex = 4;
      this.label2.Text = "Select Placed Rooms:";
      // 
      // TestForm
      // 
      this.AutoScaleDimensions = new System.Drawing.SizeF(6F, 13F);
      this.AutoScaleMode = System.Windows.Forms.AutoScaleMode.Font;
      this.ClientSize = new System.Drawing.Size(384, 561);
      this.Controls.Add(this.label2);
      this.Controls.Add(this.label1);
      this.Controls.Add(this.cboLevel);
      this.Controls.Add(this.listView1);
      this.Controls.Add(this.btmOK);
      this.Controls.Add(this.btnClose);
      this.Name = "TestForm";
      this.Text = "TestForm";
      this.Load += new System.EventHandler(this.TestForm_Load);
      this.ResumeLayout(false);
      this.PerformLayout();

    }

    #endregion

    private System.Windows.Forms.Button btnClose;
    private System.Windows.Forms.Button btmOK;
    private System.Windows.Forms.ListView listView1;
    private System.Windows.Forms.ColumnHeader columnHeader3;
    private System.Windows.Forms.ComboBox cboLevel;
    private System.Windows.Forms.Label label1;
    private System.Windows.Forms.Label label2;
  }
}
</pre>

To see how this works live, here is a [20-second screen recording](zip/change_room_level.webm):

<center>
<!-- <iframe width="640" height="620" style="position: absolute; left: 0; top: 0; width: 100%; height: 100%;" src="https://screencast.autodesk.com/Embed/Timeline/b427c037-3b56-415e-a2b9-55e17ef23475" frameborder="0" allowfullscreen="true" webkitallowfullscreen="true" scrolling="no"></iframe> -->
<iframe width="480" height="270" src="https://screencast.autodesk.com/Embed/Timeline/b427c037-3b56-415e-a2b9-55e17ef23475" frameborder="0" allowfullscreen="true" webkitallowfullscreen="true" scrolling="no"></iframe>
<center>

Many thanks to Sean for the great idea and simple solution!

#### <a name="3"></a>Slots in Panel Schedules

**Question:** I would like to propgrammatically assign slots in a panel schedule as Spaces or Spares.
I would also like to Lock and Unlock slots.
Can this be done with the Revit API?

If there is no way via that API, is there any other way we can achieve it?

<center>
<img src="img/panel_boards_api_pole_breaker_count.jpg" alt="" title="" width="490"/>
</center>

**Answer:** The *Autodesk.Revit.DB.Electrical* [`PanelScheduleView` member methods](https://www.revitapidocs.com/2020/05d89377-2e7d-9e8f-7f57-192ecb2f23e8.htm) include: `IsSpace`, `IsSpare`, `IsSlotLocked`, `AddSpace`, `AddSpare`, `RemoveSpace`, `RemoveSpare`, etc.

There is currently no public API for lock and unlock.

That would suggest that only the locking is not possible.

However, a more important question might be whether you really need to do that (caveat: just guessing your objectives).

Spares/Spaces can certainly be defined in a panel schedule.
However, there may be a good reason why this is not allowable in the API.
The question is, why would you want it to be?
Things like the IEE (Institute of Electrical Engineers) define that 25% spare capacity should be allowed at a new install; how would you know you’ve reached this minimum until you add a panel and start to populate it?
Are you asking for a panel to be effectively blocked once it has reached this maximum fill point? In which case, this would be a product of the Max number of ways (breakers) property, not the panel schedule.
The panel schedule used (and size) is a result of the panel configuration &ndash; single/three phase, no. of breakers.
The placement of Spares or spaces would then be subject to the allocation of circuits once these have been assigned to phase and load balancing has been taken in to consideration.
It is not just a case that the last 25% of available circuits are used, as nice an idea as that is, they could be scattered across the panel.


#### <a name="4"></a>JavaScript Debugging Console Commands

In case you ever work with the JavaScript debugging console, you might want to take a look at 
this css-tricks [guide to console commands](https://css-tricks.com/a-guide-to-console-commands).

I gleaned a bunch of interesting hints from it.
