+++
author = "toukakoukan"
categories = ["Azure"]
date = 2016-04-16T00:00:00Z
description = ""
draft = false
image = "/images/2016/04/2016-04-16-PowerShell-Example-Function-Trigger-Output.png"
slug = "creating-http-invoked-powershell-azure-functions"
tags = ["Azure"]
title = "Creating HTTP Invoked PowerShell Azure Functions"

+++

Azure Functions ([introduced on the 31st of March 2016](https://azure.microsoft.com/en-us/blog/introducing-azure-functions/)) is Azure’s answer to AWS Lambda, but boasts a *much* wider range of supported languages, particularly PowerShell which piqued my interest.

It’s currently in Preview, and the [documentation is a little light](https://azure.microsoft.com/en-us/documentation/articles/functions-reference/) at the time of writing. In fact, it’s basically non-existent for PowerShell save for a single *QueueTrigger – PowerShell* example function for PS scripts triggered by a queue. So let’s work from this as our base script!

[![2016-04-16 - PowerShell Example Function (Queue)](/images/2016/04/2016-04-16-PowerShell-Example-Function-Queue.png)](/images/2016/04/2016-04-16-PowerShell-Example-Function-Queue.png)

(See [Create your first Azure Function](https://azure.microsoft.com/en-us/documentation/articles/functions-create-first-azure-function/) for a guide on how to get to this point. Just make sure you select the *QueueTrigger – PowerShell *function instead of the NodeJS function he suggests.)

Now you’re setup, go to the **Integrate** tab and change the Trigger and Output to HTTP (note that the parameter names are req and res now instead of input and output).

[![2016-04-16 - PowerShell Example Function Trigger - Output](/images/2016/04/2016-04-16-PowerShell-Example-Function-Trigger-Output.png)](/images/2016/04/2016-04-16-PowerShell-Example-Function-Trigger-Output.png)

Now go back to the script and change *input* to *req* and *output* to *res* wherever you see them.

[![2016-04-16 - PowerShell Example Function InputOutput](/images/2016/04/2016-04-16-PowerShell-Example-Function-InputOutput.png)](/images/2016/04/2016-04-16-PowerShell-Example-Function-InputOutput.png)

Now change the word “queue” on line three to “HTTP” (‘cos we’re not reading from a queue anymore!), add a string of some description into the test field and click “Save and Run” at the bottom of the page.

[![2016-04-16 - PowerShell Example Function Test Run](/images/2016/04/2016-04-16-PowerShell-Example-Function-Test-Run-1.png)](/images/2016/04/2016-04-16-PowerShell-Example-Function-Test-Run-1.png)

Voila! Our writeline is logging our input!! But wait, nothing’s appeared in our output? What gives?

If you look at lines six to eight, you can see our *$entity* variable which contains our output is expecting to be able to pull an ID property from a JSON formatted input. This doesn’t exist because we’re not listening to a queue anymore!

So let’s change the *$entity* on line seven to be something a bit simpler like…

```
$entity = "Hello {0} {1}" -f $json.FirstName, $json.LastName
```

We also need to fix the JSON conversion on line 6 so it’ll work with JSON on multiple lines!
```
$json = $in | Out-String | ConvertFrom-Json
```
Now let’s update our test input to be a JSON string.
```
{ "FirstName": "Sam", "LastName": "Martin" }
```
Hit “Save and Run” at the bottom of the screen…

[![2016-04-16 - PowerShell JSON Input](/images/2016/04/2016-04-16-PowerShell-JSON-Input.png)](/images/2016/04/2016-04-16-PowerShell-JSON-Input.png)

 

Hurrah! Output and everything!

Our code now looks like this:
```$in = Get-Content $Env:req [Console]::WriteLine("Powershell script processed HTTP message '$in'")  
$output = $Env:res $json = $in | out-string | ConvertFrom-Json $entity = "Hello {0} {1}" -f $json.FirstName, $json.LastName $entity | Out-File -Encoding Ascii $output
```

So how do we call this from our laptops? Easy! Grab the Function URL from the top of the page, open a PowerShell Window and execute *Invoke-RestMethod* like so:
```
Invoke-RestMethod "<yourfunctionurlhere>" -Method POST -Body '{"FirstName":"Sam","LastName":"Martin"}'
```

[![2016-04-16 - PowerShell JSON invoke-restmethod](/images/2016/04/2016-04-16-PowerShell-JSON-invoke-restmethod.png)](/images/2016/04/2016-04-16-PowerShell-JSON-invoke-restmethod.png)

Beautiful! We get our response back into our PowerShell Window!

I’m really excited to see what people end up creating with PowerShell Azure Functions! I can see a whole world of ChatOps rolling out in front of us!


## Additional Reading

[Azure Functions Developer Reference](https://azure.microsoft.com/en-us/documentation/articles/functions-reference/)

[Invoke-RestMethod](https://www.google.co.uk/webhp?sourceid=chrome-instant&ion=1&espv=2&ie=UTF-8#q=invoke-restmethod)

