---
layout: post
title: Grep archived logs and sort the results
enki_id: 11
tags: grep logrotate bash
---

Recently I needed to search through our production logs for a request that was a few days old and had been archived by logrotate.
I found this command line script to easily search all the logs in the log directory and view the most recent results first:

```bash
$ cd path/to/log/dir
$ ls production* |sort -nr -t . -k 3,3 | xargs zgrep pattern -A 2 -B 2
```

where `production` is the name of your log and `pattern` is the pattern to grep for. This script will return 2 lines before and after the result to give a bit of context.
