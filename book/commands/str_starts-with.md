---
title: str starts-with
layout: command
version: 0.59.1
usage: |
  checks if string starts with pattern
---

# `{{ $frontmatter.title }}`

<div style='white-space: pre-wrap;'>{{ $frontmatter.usage }}</div>

## Signature

```> str starts-with (pattern) ...rest```

## Parameters

 -  `pattern`: the pattern to match
 -  `...rest`: optionally matches prefix of text by column paths

## Examples

Checks if string starts with 'my' pattern
```shell
> 'my_library.rb' | str starts-with 'my'
```

Checks if string starts with 'my' pattern
```shell
> 'Cargo.toml' | str starts-with 'Car'
```

Checks if string starts with 'my' pattern
```shell
> 'Cargo.toml' | str starts-with '.toml'
```
