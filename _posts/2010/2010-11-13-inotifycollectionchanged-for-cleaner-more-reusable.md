---
layout: post
title:  "INotifyCollectionChanged for cleaner more reusable code"
date:   2010-11-13 20:22:32 GMT
redirect_from:
  - /post/1564455979
  - /post/1564455979/inotifycollectionchanged-for-cleaner-more-reusable
---



My last post briefly described the work I did to port [iCircuit](http://icircuitapp.com) to Windows Phone 7. In it, I described how the UI was broken into two large chunks: the graphical circuit editor and the "chrome".

The chrome is all the text input fields, buttons, selectable lists - it's really quite a monstrous amount of code. But this code is not easy to port to different platforms. Yes, I communicate between the business logic and the UI through interfaces, but those interfaces have very few implementation details. When I move to a new platform, I have to rewrite all of the chrome. This is not a technology limitation as I could write a cross platform UI library like Swing, or even port WinForms to the iPhone, but those are bad ideas. Different platforms have different interaction models that must be obeyed.

That was my thinking ever since I saw [Miguel de Icaza present at Lang.net 2006](http://langnetsymposium.com/2006/speakers.aspx). And it still is. But now having worked on simultaneously porting iCircuit to 5 other platforms (Mac, Windows via Silverlight, Silverlight, WP7, Android), I am in real need of more code reuse even on the Chrome.

My first step in that direction is to embrace [INotifyCollectionChanged](http://msdn.microsoft.com/en-us/library/system.collections.specialized.inotifycollectionchanged.aspx). This interface is part of the magic sauce that makes reactive UIs easier to code. It is used heavily throughout SIlverlight to great affect.

Imagine a mail program with a list messages in one pane, and a single message in another. Imagine that the user deletes the selected message. A bunch of things have to happen:

1. The UI has to close the selected message and probably show the next one in the list
2. The app has to delete the message from its local database
3. It then has to transmit the delete request to the server
4. And then it needs to delete the message from the message list UI

This isn't hard to code, but gets really tedious. Now imagine moving the message from one folder to another. It has to be deleted from one list and placed onto another. All I want to do in code is say:

```csharp
var folder = ...;
var message = folder.GetSelectedMessage();
var otherFolder = AskUserWhere();
folder.Remove(message);
otherFolder.Add(message);
```


But I also have to write this code:

```csharp
var uis = FindAllUIsThatShow(folder);
foreach (var ui in uis) ui.Refresh();
uis = FindAllUIsThatShow(otherFolder);
foreach (var ui in uis) ui.Refresh();
```


It's stupid to write that code. First it means I have to have a way of tracking which UI elements are looking at a folder, second it means I have to perform potentially costly Refresh operations. Those refreshes are complicated by having to manage selection states, finding minimal changes in lists, requerying, and on and on.

With INotifyCollectionChanged, we have a way for the UI to react to changes in the data model. When the data changes, the UI intelligently updates. This relieves a mental burden when programming model changes, and an architectual burden when designing the UI code.

It's not the greatest interface - I wish it was actually more capable - but I have decided to adopt it for all my apps since it's pretty well baked into Silverlight and, by extension, .NET.

To that end, I have implemented a new UITableViewController in MonoTouch that knows how to listen to these collection change events and updates its data accordingly. It's a very small extension to UITableViewController, but I'm already excited by its power and ease of use.

To use it, simply assign the Collection property to an IList. The DecorateCell function is called each time the table needs to update a cell with a data item from the list. If the IList also implements INotifyCollectionChanged, then a little bit of magic happens and you get a table view that reacts intelligently to model changes. Even if your IList doesn't implement the interface, you still get a very easy way to present data using a UITableView.

Enjoy!

```csharp

using System;
using MonoTouch.UIKit;
using System.Collections.ObjectModel;
using System.Collections.Specialized;
using MonoTouch.Foundation;
using System.Collections;

namespace ObservableTableView
{
	public class RowSelectedEventArgs : EventArgs {
		public object Item { get; set; }
		public NSIndexPath Path { get; set; }		
	}
	
	public delegate void RowSelectedEventHandler(object sender, RowSelectedEventArgs e);
	
	public class ObservableTableViewController : UITableViewController
	{		
		public UITableViewRowAnimation AddAnimation { get; set; }
		public UITableViewRowAnimation DeleteAnimation { get; set; }
		public UITableViewCellStyle CellStyle { get; set; }
		
		public event RowSelectedEventHandler RowSelected;
		
		Del _del;
		Data _data;
		
		IList _collection;
		INotifyCollectionChanged _not;
		
		public IList Collection {
			get {
				return _collection;
			}
			set {
				if (_not != null) {
					_not.CollectionChanged -= HandleCollectionChanged;					
				}
				_collection = value;
				_not = _collection as INotifyCollectionChanged;
				if (_not != null) {
					_not.CollectionChanged += HandleCollectionChanged;
					TableView.ReloadData();
				}
			}
		}		
		
		public ObservableTableViewController (UITableViewStyle withStyle)
			: base(withStyle)
		{
			AddAnimation = UITableViewRowAnimation.Top;
			DeleteAnimation = UITableViewRowAnimation.Fade;
			CellStyle = UITableViewCellStyle.Default;
			
			_del = new Del(this);
			_data = new Data(this);
			
			TableView.Delegate = _del;
			TableView.DataSource = _data;
		}
		
		protected virtual void DecorateCell(UITableViewCell cell, object item, NSIndexPath path) {
		}
		
		protected virtual void OnRowSelected(object item, NSIndexPath path) {
			if (RowSelected != null) {
				RowSelected(this, new RowSelectedEventArgs() {
					Item = item,
					Path = path
				});
			}
		}

		void HandleCollectionChanged (object sender, NotifyCollectionChangedEventArgs e)
		{
			TableView.BeginUpdates();
			
			if (e.Action == NotifyCollectionChangedAction.Add) {
				var paths = new NSIndexPath[e.NewItems.Count];
				for (var i = 0; i < paths.Length; i++) {
					paths[i] = NSIndexPath.FromRowSection(e.NewStartingIndex + i, 0);
				}
				TableView.InsertRows(paths, AddAnimation);
			}
			else if (e.Action == NotifyCollectionChangedAction.Remove) {
				var paths = new NSIndexPath[e.OldItems.Count];
				for (var i = 0; i < paths.Length; i++) {
					paths[i] = NSIndexPath.FromRowSection(e.OldStartingIndex + i, 0);
				}
				TableView.DeleteRows(paths, DeleteAnimation);
			}
			
			TableView.EndUpdates();
		}
		
		class Del : UITableViewDelegate {
			ObservableTableViewController _c;
			public Del(ObservableTableViewController c) {
				_c = c;
			}
			
			public override void RowSelected (UITableView tableView, NSIndexPath indexPath)
			{
				var item = _c.Collection != null ? _c.Collection[indexPath.Row] : null;
				_c.OnRowSelected(item, indexPath);
			}
		}
		
		class Data : UITableViewDataSource {
			ObservableTableViewController _c;
			NSString _ident;
			public Data(ObservableTableViewController c) {
				_c = c;
				_ident = new NSString("C");
			}
			public override int NumberOfSections (UITableView tableView)
			{
				return 1;
			}
			public override int RowsInSection (UITableView tableview, int section)
			{
				var coll = _c.Collection;
				return coll != null ? coll.Count : 0;
			}
			public override UITableViewCell GetCell (UITableView tableView, NSIndexPath indexPath)
			{
				var cell = tableView.DequeueReusableCell(_ident);
				if (cell == null) {
					cell = new UITableViewCell(_c.CellStyle, _ident);
				}
				
				try {
					var coll = _c.Collection;
					if (coll != null) {
						var obj = coll[indexPath.Row];
						_c.DecorateCell(cell, obj, indexPath);
					}
					
					return cell;
				}
				catch {
				}
				
				return cell;
			}
		}
	}
}
```

