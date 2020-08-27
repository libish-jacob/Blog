---
layout: post
modified: '2020-08-27 10:40 +0530'
published: true
comments: true
title: Delphi how to check if you have read permission on a directory
description: >-
  It seems to be difficult to check the read permission on a directory in Delphi
  2009. Here we will see how to find the read permission in an alternate way.
tags:
  - Delphi
date: '2020-08-27 10:40 +0530'
---
It seems to be difficult to check the read permission on a directory in Delphi 2009. Here we will see how to find the read permission in an alternate way. In later version of Delphi we have more straight way of achieving this.

Here we will try to list the files in the directory and will check the status of this to decide whether the user has read permission to this drive. We must also consider whether the directory exists as well since we are doing this in an alternate manner. Below is the code which will help us decide on whether the user has permission on the given folder.

### How does it work:
We will try to list the files and folders in the directory. If the folder exists, then the FindFirst operation will return 0. If the folder does not exist, then it will return a different result. Once it returns 0, we will check the first items Attr property, this will tell us whether it is a folder or a file or something else. The first item in the result will always be the current folder. If we check the Attr then it should return a status **faDirectory** which has a value 16 which says it is a directory. now lets see how to check whether the folder has read permission. To do this we have to check if the Attr property is returning code 1040 or not. If the first item Attr says it is 1040 then that means we don’t have read permission on that drive.


{% highlight csharp %}
function HavePermissionToDrive(directory : string) : Boolean;
var
SR: TSearchRec;
faNotAllowed : Integer;
begin
faNotAllowed := 1040; //custom code for none.
Result := true;

if FindFirst(directory + '\*.*', faAnyFile, SR) = 0 then
begin
  if(SR.Attr = faNotAllowed) then
  begin
   Result := false;
  end;
  end
else
 Result := false;
end;

{% endhighlight %}

