+++
author = "toukakoukan"
categories = ["Active Directory", "Server 2003", "Server 2008"]
date = 0001-01-01T00:00:00Z
description = ""
draft = false
slug = "windows-server-2008-domain-controllers-unable-to-authenticate-login-requests"
tags = ["Active Directory", "Server 2003", "Server 2008"]
title = "Windows Server 2008 Domain Controllers unable to authenticate login requests."

+++

Recently I was faced with a problem. My Window Server 2003 DCs could authenticate logins fine, but my Windows Server 2008 DCs didn’t work at all.

Naturally, in this situation you run good old DCDIAG against the servers in question.  
 In my case, it came back with:
```
dsbindwithspnex() failed with error 1753
```

Which, with a brief google, leads you to [Troubleshooting RPC Endpoint Mapper errors](http://support.microsoft.com/kb/839880) (KB839880).

As two servers worked (2k3) and two didn’t (2k8), it seemed likely to be a simple firewall misconfiguration. So I asked for our firewall config from our firewall crew, and it all came back identical. The functioning servers had exactly the same ports open as did our non-functioning servers.

I was perplexed. Eventually a bit of “endpoint mapping troubleshooting” Googling led me to [How to configure RPC to use certain ports](http://support.microsoft.com/kb/908472) (KB908472).

Which, eventually lead me to [the realisation](http://www.windowsnetworking.com/kbase/WindowsTips/WindowsServer2008/AdminTips/Network/DynamicPortRangeinWindowsServer2008.html) that Windows Server 2003 and Windows Server 2008 use *different dynamic port ranges* for RPC.

Enable these extra ports on the firewall (49152 to 65535) or redefine them ([using netsh](http://www.windowsnetworking.com/kbase/WindowsTips/WindowsServer2008/AdminTips/Network/DynamicPortRangeinWindowsServer2008.html)), and bob’s your uncle.

Good luck diagnosing this as the issue though! The 2k3, 2k8 association isn’t necessarily the first thing that springs to mind.

