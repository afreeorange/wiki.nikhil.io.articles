
	<([a-z][a-z0-9]*)[^>]*?(/?)>

```
/              # Start Pattern
 <             # Match '<' at beginning of tags
 (             # Start Capture Group $1 - Tag Name
  [a-z]        # Match 'a' through 'z'
  [a-z0-9]*    # Match 'a' through 'z' or '0' through '9' zero or more times
 )             # End Capture Group
 [^>]*?        # Match anything other than '>', Zero or More times, not-greedy (wont eat the /)
 (\/?)         # Capture Group $2 - '/' if it is there
 >             # Match '>'
/is            # End Pattern - Case Insensitive & Multi-line ability
```

[Source](https://stackoverflow.com/a/3026235)
