---
title: shells
layout: command
version: 0.59.1
usage: |
  Lists all open shells.
---

# `{{ $frontmatter.title }}`

<div style='white-space: pre-wrap;'>{{ $frontmatter.usage }}</div>

## Signature

```> shells ```

## Examples

Enter a new shell at parent path '..' and show all opened shells
```shell
> enter ..; shells
```

Show currently active shell
```shell
> shells | where active == true
```
