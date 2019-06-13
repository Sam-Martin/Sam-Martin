+++
author = "toukakoukan"
categories = ["kanban", "leankit", "powershell"]
date = 0001-01-01T00:00:00Z
description = ""
draft = false
slug = "using-the-leankit-api-with-powershell"
tags = ["kanban", "leankit", "powershell"]
title = "Using the LeanKit API with PowerShell"

+++

As I alluded to [in an earlier post](http://samuelmartin.wordpress.com/2014/04/20/leankit-integration-with-ticketing-system-using-powershell/ "LeanKit integration with ticketing system (using PowerShell)"), I’ve been using PowerShell to interact with the[ LeanKit API](https://support.leankit.com/forums/20153741-LeanKit-API-Application-Programming-Interface- "LeanKit API"). You can find the rationale and overarching methodology in the post linked above. Here we’ll be dealing with the nuts and bolts.


# **Approach 1 – Using the .NET framework provided by LeanKit (FIXED by John Mathias)**

Initially I attempted to perform this task by importing the [LeanKit API Client Library for .NET](https://support.leankit.com/entries/28685527-LeanKit-API-Client-Library-for-NET "LeanKit API Client Library for .NET") into PowerShell using* [System.Reflection.Assembly]::LoadFile()*, but ultimately couldn’t get it to authenticate successfully.

The code snippet in question is below, if anyone can point out where I went wrong, I would be most grateful.

**UPDATED:** Fixed! John Mathias from LeanKit was kind enough to point out that I was mistakenly using the entire URL to populate the ‘hostname’ field. Change it to using just the subdomain, and it works a treat!

```
$scriptWorkingDirectory = Split-Path -Path $MyInvocation.MyCommand.Definition -Parent

# Define variables
$boardID = 01234;
$boardAddress = "subdomain"; # Your leankit subdomain, e.g. JUST the 'subdomain' part of https://subdomain.leankit.com
$boardUser = "user@example.com";

# Load LeanKit API Library dependencies
[System.Reflection.Assembly]::LoadFile("$scriptWorkingDirectoryLeanKit.API.Client.LibraryLeanKit.API.Client.Library.dll") | out-null
[System.Reflection.Assembly]::LoadFile("$scriptWorkingDirectoryLeanKit.API.Client.LibraryNewtonsoft.Json.dll") | out-null
[System.Reflection.Assembly]::LoadFile("$scriptWorkingDirectoryLeanKit.API.Client.LibraryRestSharp.dll") | out-null

# Create Authentication object so we can feed it into our factory object's creation
$leanKitAuth = New-Object LeanKit.API.Client.Library.TransferObjects.LeanKitAccountAuth -Property @{
    Hostname = $boardAddress;
    Username = $boardUser;
    Password = $([Runtime.InteropServices.Marshal]::PtrToStringAuto([Runtime.InteropServices.Marshal]::SecureStringToBSTR($(read-host "Pass" -assecurestring))));
}

# Connect and authenticate
Write-Verbose "Connecting to $($leanKitAuth.Hostname)";
$leankitApi = $(New-Object LeanKit.API.Client.Library.LeanKitClientFactory).Create($leanKitAuth);

# Create a new card
$newCard = @{
    Title = "New card!!!";
    Description = "Oh my yes, it's a new card!";
    TypeID=01234;
}
$newCard = New-Object LeanKit.API.Client.Library.TransferObjects.Card -Property $newCard;
$leankitApi.AddCards($boardID,[LeanKit.API.Client.Library.TransferObjects.Card[]]@($newCard), "Automation!!!")

# Get the board
$leankitBoard = $leankitAPI.GetBoard($boardID);

# Get the card we just added
$card = $leankitBoard.alllanes().cards[0];

# Convert it to a card rather than a view
$card = $card.toCard()

# Change it slightly
$card.Title = "That's no card!!!"

# Update it!
$leankitApi.UpdateCards($boardID,[LeanKit.API.Client.Library.TransferObjects.Card[]]$card,"Automation");
```

<del>The above code would result in a very long wait at the final step where it would (according to [Fiddler2](http://fiddler2.com "Fiddler2")) make several calls to a blank HTTPS address. So I can only assume that the $leanKitAuth object isn’t getting properly passed to the .Create() method.</del>

The above code now works properly! Thanks John!  
 It also uses the plural versions of UpdateCards with the appropriate typing so you can pass an array of card objects when you have multiple cards to update.


# **Method 2 – Invoke-RestMethod**

Ultimately PowerShell’s [Invoke-RestMethod](http://technet.microsoft.com/en-us/library/hh849971.aspx "Invoke-RestMethod"), is absolutely perfect for the job anyway, so I decided to <del>use that in lieu of getting the framework working</del> leave it here as an example even though the code above now works.


## **Step 1) Getting your board**

I created two very basic functions in order to get a board.

### Set-LeanKitAuth

```
function Set-LeanKitAuth{
    param(
        [parameter(mandatory=$true)]
        [string]$url,

        [parameter(mandatory=$true)]
        [System.Management.Automation.PSCredential]$credentials
    )
    $script:leanKitURL = $url;
    $script:leankitCreds = $credentials
    return true;
}
```

### Get-LeanKitBoard

```
function Get-LeanKitBoard{
    param(
        [parameter(mandatory=$true)]
        [int]$BoardID
    )

    if(!($script:leanKitURL -and $script:LeanKitCreds)){
        throw "You must run set-leankitauth first"
    }

    [string]$uri = $script:leanKitURL + "/Kanban/Api/Boards/$boardID/"
    return Invoke-RestMethod -Uri $uri -Credential $script:leankitCreds
}
```

The idea here is that you only have to call *Set-LeanKitAuth* once at the beginning of the script, then your credentials are pervasive throughout the subsequent calls.

So to use the above functions, you would have a snippet like so:

```
Set-LeanKitAuth -url "https://myteam.leankit.com" -credentials (Get-Credential) $leankitBoard = Get-LeanKitBoard -BoardID 1234567890
```

(Obviously replacing the URL and BoardID as appropriate.)  
 This will prompt you for your username and password (email address and password namely), and then put the resulting board in $leanKitBoard.

### ***Data to get you started***

- **Lanes:** `$leanKitBoard.ReplyData.Lanes`
- **Backlog:** `$leanKitBoard.ReplyData.BackLog`
- **Lane Cards:** `$leanKitBoard.ReplyData.Lanes.Cards`
- **Backlog Cards:** `$leanKitBoard.ReplyData.BackLog.Cards`
- **CardTypes:** `$leanKitBoard.ReplyData.cardtypes`


## **Step 2) Adding Cards**

In order to add cards using PowerShell, I whipped up another function, similar to the first.

### *** Add-LeanKitCards***

```
function Add-LeanKitCards{

    param(
        [parameter(mandatory=$true)]
        [int]$boardID,

        [parameter(mandatory=$true)]
        [ValidateScript({
            if($_.length -gt 100){
                #"You cannot pass greater than 100 cards at a time to add-LeankitCards"
                return $false;
            }
            if(
                ($_ |?{$_.UserWipOverrideComment}).length -lt $_.length
            ){
                # "All cards must have UserWipOverrideComment when passing to Update-LeankitCards";
                return $false;
            }
            return $true;
        })]
        [hashtable[]]$cards
    )

    [string]$uri = $script:leanKitURL + "/Kanban/Api/Board/$boardID/AddCards?wipOverrideComment=Automation"
    return Invoke-RestMethod -Uri $uri -Credential $script:leankitCreds -Method Post -Body $(ConvertTo-Json $cards ) -ContentType "application/json"

}
```

This requires you to pass a hashtable (or an array of hashtables) with the appropriate values to the parameter -cards.

Here’s an example:

```
$cardArray = @();
$cardArray += @{
    Title = "This is a new card";
    Description = "Oh my, so it is. A fresh card!";
    TypeID=1234567890;
    laneID=1234567890
    UserWipOverrideComment = "Automation! Yeah!"
}

Add-LeanKitCards -boardID 1234567890 -cards $cardArray
```

Again, obviously, replacing the numbers with IDs meaningful to your environment.

(use the examples in *Data to get you started* above to help you find what these IDs would be in your environment).


## Step 3) Updating Cards

Updating cards is a little trickier, as you must provide more data. But before we get into that, here’s the function we’ll use (almost identical to the one above).

### Update-LeankitCards

```
function Update-LeankitCards{

    param(
        [parameter(mandatory=$true)]
        [int]$boardID,

        [parameter(mandatory=$true)]
        [ValidateScript({
            if($_.length -gt 100){
                # "You cannot pass greater than 100 cards at a time to Update-LeankitCards"
                return $false;
            }
            if(
                ($_ |?{$_.UserWipOverrideComment}).length -lt $_.length
            ){
                # "All cards must have UserWipOverrideComment when passing to Update-LeankitCards";
                return $false;
            }
            if(
                ($_ |?{$_.ID}).length -lt $_.length
            ){
                # "All cards must have an ID when passing to Update-LeankitCards";
                return $false;
            }
            return $true;
        })]
        [hashtable[]]$cards
    )

    [string]$uri = $script:leanKitURL + "/Kanban/Api/Board/$boardID/UpdateCards?wipOverrideComment=Automation"
    return Invoke-RestMethod -Uri $uri -Credential $script:leankitCreds -Method Post -Body $(ConvertTo-Json $cards) -ContentType "application/json"
}
```

Obviously we’ll have to pass the card ID in the array, but while playing around with this, it seemed like you needed more than that. In the end I just created a new hashtable of all the properties of the card I’m updating and then changed the ones I wanted to update. Like so:

```
# Get a card from a previous board response (I'm getting the first one here with '[0]', you'll probably want to choose your card more carefully)
$card = $leanKitBoard.ReplyData.lanes.cards[0]

# Create the hashtable
$updatedCard = @{UserWipOverrideComment = "No override"};

# Populate the hashtable with all the properties of the card we selected previously.
$card | gm | ?{$_.membertype -eq "NoteProperty"} | %{$updatedCard.add($_.name, $card.$($_.name))}

# Change the parameters you want to change
$updatedCard.LaneID = 01234567890;

# Add the card to an array
$cardArray = @($updatedCard);

# Submit it!
Update-LeankitCards -boardID 1234567890 -cards $cardArray
```

I shouldn’t need to say it again, but obviously, change the board ID etc to reflect your environment.

And that’s it! Easy really. There’s a lot more that you can do, for which I suggest you see the additional reading section.  
 Any questions, just leave them in the comments.


# Additional Reading

[https://support.leankit.com/forums/20153741-LeanKit-API-Application-Programming-Interface-](https://support.leankit.com/forums/20153741-LeanKit-API-Application-Programming-Interface-)

[https://support.leankit.com/entries/20265038-API-Basics](https://support.leankit.com/entries/20265038-API-Basics)

[http://technet.microsoft.com/en-us/library/hh849898.aspx](http://technet.microsoft.com/en-us/library/hh849898.aspx)

