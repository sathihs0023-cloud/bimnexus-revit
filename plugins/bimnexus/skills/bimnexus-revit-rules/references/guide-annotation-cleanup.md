# Guide: Plan annotation cleanup

> BIMNEXUS note: written with revit-mcp tool names. Use the matching BIMNEXUS tools; anything that needs custom C# (execute_csharp / send_code_to_revit) has no BIMNEXUS equivalent – do it with BIMNEXUS tools or ask the user. Every model change still needs the user's confirmation.

**Summary:** Room-tag clash resolver (least-cost move inside room), leaders off, lift tags, door tags centred - with code.
**Related rules:** ANNO-001, ANNO-002, ANNO-003, ANNO-004, ANNO-005, REJ-001 (entries bnx-rule-…)

## Overview

This skill holds the clean-up passes that follow a template/tag rollout (see **revit-plan-visual-standard**). It works on any building. Read **revit-mcp-csharp-runner** first for the code rules.

## The user's expectations

- **Readable tags inside their room, without leaders.** Tags must not overlap dimensions, other tags, door/window tags, text, walls, columns, doors or stairs. Furniture overlap is tolerated more than walls.
- **No "outside room with a leader" fallback.** An attempt that pushed tags into corridors with leaders was rejected and had to be reverted by script. If a room is too small, keep the least-bad spot inside it and list it for manual review.
- **Exception – area tags on area plans (ANNO-005).** On BUA/GFA area-calculation plans, an area tag whose area is too small for the text (narrow facade terraces, small core rooms) gets a leader. The tag stays horizontal, and the text goes in clear space outside the area: white space beyond the facade, or the nearest free area for core rooms. The text never overlaps other annotation. The leader is short and straight (horizontal preferred) and ends in the middle of the area. Try a least-cost move inside the area first. Clean tags are never moved, and duplicate tags on one area are reported, not moved. Turn the leader on before moving the head outside, or Revit rolls the change back silently.
- **Door tags:** the centre of the tag circle sits exactly on the midpoint of the door opening (the door insertion point), with no leader.
- **Lift tags:** every lift shaft room (PL, SL, LIFT, etc.) visible in the plan gets a tag.
- Fix **all** overall/partials/part plans, then report counts per view. Offer a spreadsheet with a Summary tab using COUNTIF/SUMIF formulas, a per-view tab, and a method tab.

## Order of passes (per view, 1–3 views per call)

1. **Leaders off** on room tags and door tags with misaligned leaders. Use a separate call, because bounding boxes include the leader and the next passes need clean boxes.
2. **Missing lift tags.**
3. **Room-tag clash resolver:** ID/room-name tags first, then unit tags.
4. **Door-tag centring.**
5. Read-back verification and the report.

For speed, never scan a view with a plain `FilteredElementCollector(doc, viewId)`. Use an `ElementMulticategoryFilter` of the annotation categories you need. Read all positions first, then do all moves. Calling `get_BoundingBox(view)` after a move forces a regeneration and caused timeouts.

## Pass 2 – missing lift tags

1. For each loaded link (`RevitLinkInstance`, not `IsHidden(v)`), collect the rooms whose name or number matches lift patterns. Keep those on the view's level, i.e. whose location, transformed to host coordinates, is inside the crop box.
2. Skip rooms already tagged in the view: compare against `RoomTag.TaggedRoomId.LinkedElementId` together with the link instance id.
3. Create the tag with `document.Create.NewRoomTag(new LinkElementId(link.Id, room.Id), new UV(p.X, p.Y), v.Id)`, where `p = link.GetTotalTransform().OfPoint(roomPt)`.
4. Set the standard unit tag type.

## Pass 3 – least-cost room tag resolver

**Idea:** for each tag, score candidate positions on a grid around its current spot. The score is a weighted overlap area with obstacles plus a distance penalty. Move the tag to the cheapest candidate whose head and box centre are both inside the tagged room.

Obstacles, all as host-XY boxes:
- **Annotation** in the view: RoomTags, DoorTags, WindowTags, Dimensions, SpotElevations, SpotSlopes, Viewers, Elev, TextNotes, GenericAnnotation, CalloutHeads. Ignore boxes larger than 400 mm on paper, which are giant dimension strings or elevation boxes.
- **Linked model elements** from links whose title contains `-AR-`, `-ID-` or `-STR-` (adapt to the project's naming) and that are not hidden in the view. Collect them with a `BoundingBoxIntersectsFilter` built from the crop box transformed into link coordinates, with z from level + 0.1 ft to level + 4 ft.

There are three weight classes:

| Class | Weight | Contents |
|---|---|---|
| annotation | 1000 | the annotation obstacles above |
| hard | 100 | Walls, Columns, StructuralColumns, Doors, Windows, Stairs, StairsRailing, SpecialityEquipment |
| soft | 5 | Furniture, FurnitureSystems, Casework, PlumbingFixtures, GenericModel |

Scoring rules:
- A model obstacle that fully contains the tag box and is more than 4× its area costs ×0.1, because it is likely a big bounding box such as a long wall or a stair. Specialty equipment never gets this discount.
- Use a spatial grid index with 1 m (3.28 ft) cells. Without it the pass times out.
- Candidates sit on a ±R grid with a 1.5 mm paper step. Unit tags use R = 8 mm and a distance penalty of 3 per mm. Other tags use R = 20 mm and 0.5 per mm.
- Move only when the cost improves by more than 1. After a move, register the tag's new box as an obstacle for later tags.

Sketch (adapt names; test on one view first):

```csharp
View v=document.GetElement(new ElementId(VIEW_ID)) as View;
double P=v.Scale/304.8;                      // model ft per paper mm
BoundingBoxXYZ crop=v.CropBox; Transform ct=crop.Transform;
XYZ c0=ct.OfPoint(crop.Min), c1=ct.OfPoint(crop.Max);
double lz=v.GenLevel.Elevation;
var box=new List<double[]>();               // {minx,miny,maxx,maxy,weight,id,isSpec}
var grid=new Dictionary<long,List<int>>(); double CELL=3.28;
Func<double,long> cell=(t)=>(long)Math.Floor(t/CELL);
Action<int> index=(i)=>{ double[] b=box[i]; for(long gx=cell(b[0]);gx<=cell(b[2]);gx++) for(long gy=cell(b[1]);gy<=cell(b[3]);gy++){ long k=gx*1000003L+gy; List<int> l; if(!grid.TryGetValue(k,out l)){l=new List<int>();grid[k]=l;} l.Add(i);} };
Action<double,double,double,double,double,long,double> add=(a,b,cc,d,w,id,sp)=>{ box.Add(new double[]{a,b,cc,d,w,id,sp}); index(box.Count-1); };

// annotation obstacles
var ac=new List<BuiltInCategory>{BuiltInCategory.OST_RoomTags,BuiltInCategory.OST_DoorTags,BuiltInCategory.OST_WindowTags,BuiltInCategory.OST_Dimensions,BuiltInCategory.OST_SpotElevations,BuiltInCategory.OST_SpotSlopes,BuiltInCategory.OST_Viewers,BuiltInCategory.OST_Elev,BuiltInCategory.OST_TextNotes,BuiltInCategory.OST_GenericAnnotation,BuiltInCategory.OST_CalloutHeads};
var tagsId=new List<RoomTag>(); var tagsUnit=new List<RoomTag>();
foreach(Element e in new FilteredElementCollector(document,v.Id).WherePasses(new ElementMulticategoryFilter(ac)).WhereElementIsNotElementType()){
  BoundingBoxXYZ bb=e.get_BoundingBox(v); if(bb==null) continue;
  if((bb.Max.X-bb.Min.X)/P>400||(bb.Max.Y-bb.Min.Y)/P>400) continue;
  add(bb.Min.X,bb.Min.Y,bb.Max.X,bb.Max.Y,1000,e.Id.Value,0);
  RoomTag rt=e as RoomTag; if(rt!=null && rt.GroupId.Value<=0){ string fn=rt.RoomTagType.FamilyName; if(fn.Contains("ID")) tagsId.Add(rt); else tagsUnit.Add(rt);} }

// linked model obstacles: for each RevitLinkInstance li (title filter, !li.IsHidden(v))
//   Transform tr=li.GetTotalTransform(), inv=tr.Inverse;
//   Outline from inv.OfPoint of the crop corners with z lz+0.1 .. lz+4 (also inverted);
//   collect hard/soft categories via ElementMulticategoryFilter + BoundingBoxIntersectsFilter;
//   bb=e.get_BoundingBox(null); transform its 4 XY corners with tr, take the AABB, add with weight 100 or 5 (isSpec=1 for SpecialityEquipment).

Func<double,double,double,double,long,double> cost=(x0,y0,x1,y1,self)=>{
  double s=0; var seen=new HashSet<int>(); double tA=(x1-x0)*(y1-y0);
  for(long gx=cell(x0);gx<=cell(x1);gx++) for(long gy=cell(y0);gy<=cell(y1);gy++){ List<int> l; if(!grid.TryGetValue(gx*1000003L+gy,out l)) continue;
    foreach(int i in l){ if(!seen.Add(i)) continue; double[] b=box[i]; if((long)b[5]==self||b[4]==0) continue;
      double w=Math.Min(x1,b[2])-Math.Max(x0,b[0]), h=Math.Min(y1,b[3])-Math.Max(y0,b[1]); if(w<=0||h<=0) continue;
      double wt=b[4]; bool inside=x0>=b[0]&&x1<=b[2]&&y0>=b[1]&&y1<=b[3]; double oA=(b[2]-b[0])*(b[3]-b[1]);
      if(wt<1000 && b[6]==0 && inside && oA>4*tA) wt*=0.1;
      s+=wt*(w*h)/(P*P); } }
  return s; };
// For each tag (tagsId first, then tagsUnit):
//   room: LinkElementId lr=rt.TaggedRoomId; linked → li=(RevitLinkInstance)document.GetElement(lr.LinkInstanceId), room=li.GetLinkDocument().GetElement(lr.LinkedElementId) as Room, inv=li.GetTotalTransform().Inverse; host → document.GetElement(lr.HostElementId).
//   inRoom(x,y)= room.IsPointInRoom(inv.OfPoint(new XYZ(x,y,lz+1)))  (z = room level + 1 ft in link coords)
//   base cost at the current box; loop dx,dy over -R..R step 1.5 (paper mm) → model = *P; require inRoom(head+d) && inRoom(centre+d);
//   c = cost(box+d) + penalty*sqrt(dx²+dy²); keep the best.
//   if best < base-1: ElementTransformUtils.MoveElement(document, rt.Id, d); mark the old obstacle entry weight 0; add the new box (weight 1000).
// Return a string: moved / unchanged / no-room counts and remaining high-cost tag ids for manual review.
```

Notes:
- A tag without a leader cannot be placed outside its room. Revit errors with "Head position of the tag with no leader could not be located outside". The in-room check prevents it; do not switch leaders on as a workaround.
- 1:200 overalls barely improve, because the tags are large relative to rooms. Report them rather than forcing moves.
- Known hard cases are tiny rooms (bathroom beside a column, GB room). List these for manual review.

## Pass 4 – door tag centring

Read everything first, then write. With leaders already off:

```csharp
var tl=new List<IndependentTag>(); var dl=new List<XYZ>();
foreach(Element e in new FilteredElementCollector(document,v.Id).OfCategory(BuiltInCategory.OST_DoorTags).WhereElementIsNotElementType()){
  IndependentTag t=e as IndependentTag; if(t==null||t.GroupId.Value>0) continue;
  XYZ dp=null;
  foreach(LinkElementId le in t.GetTaggedElementIds()){
    Element door=null; Transform tr=Transform.Identity;
    if(le.LinkInstanceId!=ElementId.InvalidElementId){ RevitLinkInstance li=document.GetElement(le.LinkInstanceId) as RevitLinkInstance; if(li==null||li.GetLinkDocument()==null) continue; door=li.GetLinkDocument().GetElement(le.LinkedElementId); tr=li.GetTotalTransform(); }
    else door=document.GetElement(le.HostElementId);
    LocationPoint lp=door==null?null:door.Location as LocationPoint; if(lp!=null) dp=tr.OfPoint(lp.Point); }
  BoundingBoxXYZ bb=t.get_BoundingBox(v); if(dp==null||bb==null) continue;
  XYZ c=(bb.Min+bb.Max)*0.5; tl.Add(t); dl.Add(new XYZ(dp.X-c.X,dp.Y-c.Y,0)); }
for(int i=0;i<tl.Count;i++){ if(tl[i].HasLeader) tl[i].HasLeader=false; tl[i].TagHeadPosition=tl[i].TagHeadPosition+dl[i]; }
```
The door `LocationPoint` is the midpoint of the opening, on the wall centreline. Report tags skipped because the door was unloaded or missing.

## Verify and report

After each batch, run a separate read-back call that recounts the clashes, tag positions and tag types. Do this especially after a timeout, because the next call can return the stale result of the timed-out one. The final report lists, per view:
- tags moved / unchanged
- remaining clashes, with ids
- lift tags added
- door tags centred / skipped

It also says what needs a manual touch.
