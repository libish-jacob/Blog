---
layout: post
modified: '2014-10-7 13:54 +0530'
published: true
comments: true
title: How to set width or height to RadSplitContainer wpf
tags:
  - WPF
categories:
  - UI Controls
  - .Net
description: >-
  It is often a requirement when we use RadSplitContainer and we want to set
  width or height to RadSplitContainer. Here in this post we will see how to
  achieve this.
date: '2014-10-7 13:54 +0530'
---
## Introduction
  It is often a requirement when we use RadSplitContainer and we want to set width or height to RadSplitContainer. Here in this post we will see how to achieve this. When we use RadSplitContainer, we would like it to occupy a small space initially and if required we can drag it to take more space. This is normally achieved by setting the width for the control. If we are using two RadSplitContainer and if we wish to set these RadSplitContainer widths in 1:3 ratios, we may not be able to achieve this easily if you are not an expert in it. So here is a post to relieve your effort to achieve this.

If you can set the RadSplitContainer inside a Docking control, then we can set the width of each RadSplitContainer seperately. This is as shown below.

{% highlight csharp %}
<telerik:RadDocking>
  <telerik:RadSplitContainer Width="300">
    <telerik:RadPaneGroup>
      <telerik:RadPane></telerik:RadPane>
    </telerik:RadPaneGroup>
  </telerik:RadSplitContainer>
  <telerik:RadSplitContainer>
      <telerik:RadPaneGroup>
        <telerik:RadPane></telerik:RadPane>   
      </telerik:RadPaneGroup>
  </telerik:RadSplitContainer>
</telerik:RadDocking>
{% endhighlight %}

If you try the above code, then the first RadSplitContainers will take a width of 300 and the other will take a width as required by its child controls. In most of the scenario we would like the second RadSplitContainers to take up the rest of the space. In order for the RadSplitContainers to take up the whole space you need to set the HasDocumentHost property of the Docking control to False. This is as shown below.

{% highlight csharp %}
<telerik:RadDocking HasDocumentHost="False">
  <telerik:RadSplitContainer Width="300">
    <telerik:RadPaneGroup>
      <telerik:RadPane></telerik:RadPane>
    </telerik:RadPaneGroup>
  </telerik:RadSplitContainer>
  <telerik:RadSplitContainer>
    <telerik:RadPaneGroup>
      <telerik:RadPane></telerik:RadPane>   
    </telerik:RadPaneGroup>
  </telerik:RadSplitContainer>
</telerik:RadDocking>
{% endhighlight %}

Now the RadSplitContainer's together will fill the rest of the space.
