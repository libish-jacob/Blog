---
layout: post
modified: '2014-04-13 11:43 +0530'
published: true
comments: true
title: >-
  Implement Multiselect in WPF DataGrid in proper WPF way with Multiselect
  Behavior and without any code behind
description: >-
  In this article you will see how to implement multiselect in WPF DataGrid in
  proper way when you are following MVVM pattern. In this article you will see
  how to implement a behavior to your DataGrid so that we can achieve
  multiselect.
tags:
  - WPF
categories:
  - .Net
date: '2014-04-13 11:43 +0530'
---
### Introduction
In this article you will see how to implement multiselect in WPF DataGrid in proper way when you are following MVVM pattern. In this article you will see how to implement a behavior to your DataGrid so that we can achieve multiselect.

Behaviors are introduced with Expression Blend, to encapsulate pieces of functionality into a reusable component. These components can then be attached to controls to give them an additional behavior.

### Background
Multiselect is something which is like a must have feature in any control which displays multiple items, but unfortunately it is not straight forward in WPF when you are an MVVM developer. We need to attach a multiselect behavior to controls like DataGrid in order to get multiselect working.

### Problem it addresses
In DataGrid, the property "SelectedItems” does not have an accessible setter. So we need alternate ways to get the list of selected items.

### Requirements
Here you need to add reference to System.Windows.Interactivity. If you have installed blend, then it is available in C:\Program Files (x86)\Microsoft SDKs\Expression\Blend 3\Interactivity\Libraries\WPF.

### Using the code

1. Add reference to System.Windows.Interactivity
2. Add DataGrid to your xaml.

{% highlight csharp %}
<window […]="" xmlns:i="http://schemas.microsoft.com/expression/2010/interactivity">
    <grid>
        <datagrid itemssource="{Binding Values}">
        </datagrid>
    </grid>
</window>
{% endhighlight %}

3. create view model with a public property called SelectedItems.
{% highlight csharp %}
namespace WpfApplicationTest
{
    using System.Collections;
    using System.Collections.Specialized;

    internal class MainWindowViewModel
    {
        public IList Values { get; set; }

        public MainWindowViewModel()
        {
            this.Values = new StringCollection { "test1", "test2", "test3", "test4", "test5" };
        }

        public IList SelectedItems { get; set; }
    }
}
{% endhighlight %}

4. Create multiselect behavior as shown below.
{% highlight csharp %}
namespace WpfApplicationTest
{
    using System;
    using System.Windows.Controls;
    using System.Windows.Interactivity;

    public class MultiSelectBehavior : Behavior<datagrid>
    {
        protected override void OnAttached()
        {
            base.OnAttached();

            ((MainWindowViewModel)this.AssociatedObject.DataContext).SelectedItems = this.AssociatedObject.SelectedItems;
        }
    }
}
{% endhighlight %}

5. Now add this behavior to DataGrid. Also don't forget to make SelectionMode="Extended". This will help you to select row by pressing Ctrl key.
{% highlight csharp %}
<window […]="">
    <grid>
        <datagrid itemssource="{Binding Values}" SelectionMode="Extended">
            <i:interaction.behaviors>
                <wpfapplicationtest:multiselectbehavior></wpfapplicationtest:multiselectbehavior>
            </i:interaction.behaviors>
        </datagrid>
    </grid>
</window>
{% endhighlight %}

### Point of interest
If you have multiple DataGrid, then you need multiple Multiselect behaviors, one way to avoid this is by creating switch in MultiselectBehavior.
{% highlight csharp %}
namespace WpfApplicationTest
{
    using System;
    using System.Windows.Controls;
    using System.Windows.Interactivity;

    public class MultiSelectBehavior : Behavior<datagrid>
    {
        protected override void OnAttached()
        {
            base.OnAttached();
            switch (this.AssociatedObject.Name)
            {
                case "DataGrid1":
                    ((MainWindowViewModel)this.AssociatedObject.DataContext).SelectedItemsForDataGrid1 = this.AssociatedObject.SelectedItems;

                    break;

                case "DataGrid2":
                    ((MainWindowViewModel)this.AssociatedObject.DataContext).SelectedItemsForDataGrid1 = this.AssociatedObject.SelectedItems;

                    break;

                default:
                    break;
            }
        }
    }
}
{% endhighlight %}