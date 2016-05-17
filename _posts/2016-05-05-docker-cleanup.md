---
layout:   post
title:    'Docker: Delete your unused containers'
date:     2016-05-05
summary:  Just what the title said.
category: cli
---

### Remove unused instances

```
docker rm `docker ps -aq -f status=paused -f status=exited -f status=dead`
```

### Remove images with name "\<none\>"

```
docker rmi -f `docker images -q --filter "dangling=true"
```
