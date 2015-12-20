The following grant access to *all* users. This is useful if you've
mistakenly restricted access to a few and have locked yourself out.

### For ARD versions *below* 3.2

` sudo /System/Library/CoreServices/RemoteManagement/ARDAgent.app/Contents/Resources/kickstart \`  
` `**`-activate` `-configure` `-access` `-on` `-restart` `-agent`
`-privs` `-all`**

### For ARD versions *above* 3.2

` sudo /System/Library/CoreServices/RemoteManagement/ARDAgent.app/Contents/Resources/kickstart \`  
` `**`-configure` `-allowAccessFor` `-allUsers` `-privs` `-all`**

[Category:Nikhil's Notes](Category:Nikhil's_Notes "wikilink")
[Category:From a past sysadmin
life](Category:From_a_past_sysadmin_life "wikilink")
