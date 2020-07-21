---
layout: post
modified: '2014-04-13 11:37 +0530'
published: true
comments: true
title: How to Implement Multiselect in WPF RadGridView without any codebehind
description: >-
  You can achieve this with the help of Behavior in WPF. Create one multiselect
  behavior and attach it to RadGridView. This way you can avoid any code behind
  code altogether and can implement multiselect in proper MVVM way.
tags:
  - WPF
categories:
  - .Net
date: '2014-04-13 11:37 +0530'
---
### Issue
You can achieve this with the help of Behavior in WPF. Create one multiselect behavior and attach it to RadGridView. This way you can avoid any code behind code altogether and can implement multiselect in proper MVVM way.

### How did I fix it
Create a property of type ObservableCollection in your ViewModel like as shown below.

{% highlight csharp %}
public ObservableCollection<object> SelectedItems { get; set; }
{% endhighlight %}

Now create a MultiselectBehavior as shown below. You have to add reference to **System.Windows.Interactivity.dll**
  
{% highlight csharp %}
public class MultiSelectBehavior : Behavior<radgridview>
    {
        protected override void OnAttached()
        {
            base.OnAttached();
            ((YourViewModel)this.AssociatedObject.DataContext).SelectedItems = this.AssociatedObject.SelectedItems;
        }
    }
{% endhighlight %}

Now add this behavior to RadGridView as shown below. Also don't forget to make selectionmode="Extended". This will help you to select row by pressing ctrl key.
{% highlight csharp %}
<telerik:radgridview itemssource="{Binding AvailableSignals, Mode=TwoWay}"  SelectionMode="Extended">
      <telerik:radgridview.columns>
         <telerik:gridviewdatacolumn>
      </telerik:gridviewdatacolumn></telerik:radgridview.columns>
      <i:interaction.behaviors>
          <multiselect:multiselectbehavior>
      </multiselect:multiselectbehavior></i:interaction.behaviors>
 </telerik:radgridview>
{% endhighlight %}
