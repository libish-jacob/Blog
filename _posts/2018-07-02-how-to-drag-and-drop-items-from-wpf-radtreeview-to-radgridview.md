---
layout: post
modified: '2014-09-02 11:01 +0530'
published: true
comments: true
title: How to drag and drop items from WPF RadTreeView to RadGridView
description: >-
  This could be one of the required features if you are working on WPF. But this
  won’t work without some additional work. Here in this post we will see how we
  can drag and drop items from RadTreeView to RadGridView. This is achieved with
  the help of behaviors.
tags:
  - WPF
categories:
  - UI Controls
date: '2014-09-02 11:01 +0530'
---
## Introduction

  This could be one of the required features if you are working on WPF. But this won’t work without some additional work. Here in this post we will see how we can drag and drop items from RadTreeView to RadGridView. This is achieved with the help of behaviors.

Below shown is the snippet of xaml for tree view. Here we have to set IsDragDropEnabled="True". This will allow you to drag and drop items from treeview to itself. I.e. you can drag a tree node to another parent node. We will have to do additional work to get it dragged and dropped to another control. Now we have to attach a custom behavior to this tree view with the help of interactivity. If you don’t want to use interactivity, then have a look at the bottom to find an alternative for this.

{% highlight csharp %}
<telerik:radtreeview isdragdropenabled="True" itemssource="{Binding IoSystemTree}" tag="{Binding }">
  <telerik:radtreeview.itemtemplate>
   [… …]
  </telerik:radtreeview.itemtemplate>
        <i:interaction.behaviors>
        <ioview:treedragdropbehavior>
        </ioview:treedragdropbehavior></i:interaction>
</telerik:radtreeview>
{% endhighlight %}

Now in the GridView we will have to set AllowDrop="True". This will allow you to drop items to this control. Now we will have to add a custom behavior for this control that will accept the dropped item. We will be using interactivity to add this custom binding to this control. This is as shown below.

{% highlight csharp %}
<telerik:RadGridView Name="IoSignals" 
AllowDrop="True" 
IsReadOnly="True" 
AutoGenerateColumns="False" 
ItemsSource="{Binding SignalsToSetAndSubscribe}" 
RowIndicatorVisibility="Collapsed" 
SelectedItem="{Binding SignalsToSetSelectedItem}" 
SelectionMode="Extended" 
ShowGroupPanel="False" 
VirtualizingStackPanel.VirtualizationMode="Recycling">
[… …]
 <i:Interaction.Behaviors>
 <ioView:DragDropBehavior />
 </i:Interaction.Behaviors>
</telerik:RadGridView>
{% endhighlight %}

The custom behavior for RadGridView is as shown below.

{% highlight csharp %}
using System.Collections;
    using System.Collections.Generic;
    using System.Collections.ObjectModel;
    using System.Linq;
    using System.Windows;
    using System.Windows.Interactivity;
    using Telerik.Windows.Controls;
    using Telerik.Windows.Controls.GridView;
    using Telerik.Windows.Controls.TreeView;
    using Telerik.Windows.DragDrop;
    using Vestas.Firecrest.Advanced.Presentation.ViewModels.IO;
   using DragEventArgs = Telerik.Windows.DragDrop.DragEventArgs;

  public class DragDropBehavior : Behavior<RadGridView> {
        public ObservableCollection<object> Collection { get; set; }
        public int DropIndex { get; set; }

   protected override void OnAttached()  {
            base.OnAttached();
            this.AssociatedObject.RowLoaded += this.AssociatedObjectRowLoaded;
            DragDropManager.AddDropHandler(this.AssociatedObject, AssociatedObjectDragEnter);
            DragDropManager.AddDragOverHandler(this.AssociatedObject, this.OnRowDragOver, true);
        }       

   private void AssociatedObjectRowLoaded(object sender, RowLoadedEventArgs e) {
            if (e.Row is GridViewHeaderRow || e.Row is GridViewNewRow || e.Row is GridViewFooterRow) {
                return;
            }

   var row = e.Row as GridViewRow;
         this.InitializeRowDragAndDrop(row);
        }

   private void InitializeRowDragAndDrop(GridViewRow row) {
            if (row == null) {
                return;
            }

   DragDropManager.RemoveDragOverHandler(row, this.OnRowDragOver);
   DragDropManager.AddDragOverHandler(row, this.OnRowDragOver);
        }

  private void AssociatedObjectDragEnter(object sender, DragEventArgs e) {
            var dataFromObject = DragDropPayloadManager.GetDataFromObject(e.Data, TreeViewDragDropOptions.Key) as TreeViewDragDropOptions;

   if (dataFromObject != null) {
                IEnumerable<object> draggedItems = dataFromObject.DraggedItems.ToList();
/*
Forach(var item in draggedItems )
this.AssociatedObject.Items.AddNewItem(item);
*/
AddItemToList();// Add your custom code to do the injection of dragged items to grid.
            }
        }

private void OnRowDragOver(object sender, DragEventArgs e) {
// the below line of codes will force the control to accept even the disallowed objects.

var options = DragDropPayloadManager.GetDataFromObject(e.Data, TreeViewDragDropOptions.Key) as TreeViewDragDropOptions;
       if (options != null)
         {
             options.DropAction = DropAction.Move;
             var dragVisual = options.DragVisual as TreeViewDragVisual;
             if (dragVisual != null) {
                 dragVisual.IsDropPossible = true;
             }
            }

//Now we will find the and update the index of dropped item.
            var row = sender as GridViewRow;
            if (row == null) {
                return;
            }

int dropIndex = (this.AssociatedObject.Items as IList).IndexOf(row.DataContext);
DropPosition dropPositionFromPoint = this.GetDropPositionFromPoint(e.GetPosition(row), row);

if (dropIndex >= row.GridViewDataControl.Items.Count - 1 &&
          dropPositionFromPoint == DropPosition.After) {
                return;
            }
            
   DropIndex = dropIndex;
    }

  private DropPosition GetDropPositionFromPoint(Point absoluteMousePosition, GridViewRow row)
     {
        if (row != null)
         {
         return absoluteMousePosition.Y < row.ActualHeight / 2 ? DropPosition.Before : DropPosition.After;
         }

return DropPosition.Inside;
      }
    }
{% endhighlight %}

Below is the complete code for custom behavior which we have to attach to TreeView.

{% highlight csharp %}
using System.Collections.ObjectModel;
    using System.Windows.Interactivity;
    using Telerik.Windows.Controls;
    using Telerik.Windows.Controls.TreeView;
    using Telerik.Windows.DragDrop;

   public class TreeDragDropBehavior : Behavior<RadTreeView> {
        public ObservableCollection<object> Collection { get; set; } 
        public int DropIndex { get; set; }

   protected override void OnAttached() {
            base.OnAttached();
            AssociatedObject.IsDragTooltipEnabled = false;
            TreeViewSettings.SetDragDropExecutionMode(AssociatedObject, TreeViewDragDropExecutionMode.New);
            DragDropManager.AddDropHandler(AssociatedObject, OnDrop, true);
            AssociatedObject.PreviewDragLeave += OnDragLeave;
        }

   private void OnDrop(object sender, DragEventArgs e) {
/* this will block items to be dragged and added to the treeview itself. If your application wants this, then comment out this line of code.*/
            e.Handled = true;
        }

   private void OnDragLeave(object sender, System.Windows.DragEventArgs e) {
            var options = DragDropPayloadManager.GetDataFromObject(e.Data, TreeViewDragDropOptions.Key) as TreeViewDragDropOptions;

   if (options != null) {
// here we are forcing the drop status to say valid.
                options.DropAction = DropAction.Move;
                var dragVisual = options.DragVisual as TreeViewDragVisual;
                if (dragVisual != null)
                {
                    dragVisual.IsDropPossible = true;
                }
            }
        }
    }
    
{% endhighlight %}

Now after following the steps as explained below, the drag and drop functionality will start working for you. Here we are using WPF interactivity to bind the behavior to control. But if you don’t like interactivity, then create a bool property in the behavior class and set that property from xaml like as shown below.

{% highlight csharp %}
public class TreeDragDropBehavior : Behavior<RadTreeView>
    {
Private bool isEnabled;
public bool IsEnabled
{
 get { return isEnabled;}
 set{ isEnabled = value;
 OnAttached();
 }
}

// Rest of the code goes here.
}
{% endhighlight %}

And now from XAML you have to attach to this behavior like as shown below.

{% highlight csharp %}
<telerik:RadTreeView IsDragDropEnabled="True" ItemsSource="{Binding IoSystemTree}" Tag="{Binding }" TreeDragDropBehavior.IsEnabled=”true”>
{% endhighlight %}

You can repeat the same for DragDropBehavior and for RadGridView. This will attach your behaviors to the control without the help of interactivity. Once you have done as explained, you should be able to drag and drop items from tree view to grid.
