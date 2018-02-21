---
layout: default
title: Working with Docker toolbox and VS Code
categories: [docker, vscode]
category: docker
date:   2015-11-17 16:16:01 -0600
---
# Working with Docker toolbox and VS Code

1. Set Default cmd and default terminal

```json
"terminal.integrated.shell.windows": "C:\\Windows\\System32\\cmd.exe"
```

2. Execute in docker terminal

```bash
docker-machine env
```

3. Setting env variables and running vscode

```bash
 @FOR /f "tokens=*" %i IN ('docker-machine env') DO @%i && code .
```

---

## Tips

- If you want to do this every time you start VS Code, it would probably be easier to create define all env variables as `Globals` windows env variables