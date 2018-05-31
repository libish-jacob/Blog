---
layout: post
modified: '2014-04-13 12:28 +0530'
published: true
comments: true
title: How to suppress PRISM module initialisation errors and continues execution
tags:
  - 'C#'
  - Prism
categories:
  - .Net
description: >-
  In the process of module initialization, if there is any error in a module,
  then PRISM will throw exception and will stop loading other modules until you
  fix the error in the module. But if your module is not really important in
  your application, you would like to suppress this module initialization error
  and continue loading other modules.
date: '2014-04-13 12:28 +0530'
---
## Introduction
While in the process of module initialization, if there is any error in a module, then PRISM will throw exception and will stop loading other modules until you fix the error in the module. But if your module is not really important in your application, you would like to suppress this module initialization error and continue loading other modules.

## How did I do it:
In such situation you have to handle the ModuleInitializationError by your own.
This is achieved by a Custom module initializer that silently logs module initialization errors but continues execution without throwing exceptions further up the call stack.

You will need to inherit ModuleInitializer and override HandleModuleInitializationError and then handle the exception by your own. ModuleInitializer is available in Microsoft.Practices.Prism.Modularity;

{% highlight csharp %}
public class CustomModuleInitializer : ModuleInitializer  {
        public CustomModuleInitializer(IServiceLocator serviceLocator, ILoggerFacade loggerFacade)
            : base(serviceLocator, loggerFacade) {
        }

        public override void HandleModuleInitializationError(ModuleInfo moduleInfo, string assemblyName, Exception exception) {
            try {
                base.HandleModuleInitializationError(moduleInfo, assemblyName, exception);
            }
            catch (Exception ex) {
//log errors.
            }
        }
    }
{% endhighlight %}


And from your bootstrapper, override ConfigureContainer and register this module initializer like as shown below. 

{% highlight csharp %}
protected override void ConfigureContainer()        
{
// Register custom module initializer that does not stop initializing modules on first error.            
Container.RegisterType<IModuleInitializer, CustomModuleInitializer>();
}
{% endhighlight %}
