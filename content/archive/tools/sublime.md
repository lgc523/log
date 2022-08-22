---
title: "Sublime 配置"
date: 2022-08-19T17:29:11+08:00
type: post
tags: ['tools']
---

## sublime config

一个项目目录包含 ``java`` 和 ``go`` 两种语言，``idea`` 加载 java 项目直接就不显示 ``go`` 项目的文件夹了，因为idea go 插件偶尔出问题，懒得找idea的解决方法，就用 ``sublime`` 打开了，记录配置一下 ``sublime``。

顺带记录一下在 mac 上显示 鼠标操作的app，``KeyCastr``。

### plugin
- **gosublime**
  
  ```
  git clone https://margo.sh/GoSublime
  git clone https://github.com/DisposaBoy/GoSublime
  git clone https://hub.fastgit.xyz/DisposaBoy/GoSublime
  ctrl . & ctrl &
  ```

  - 修改默认配置
  
    ```
    {
        "env": {
            "GOPATH": "/",
            "GOROOT": "/usr/local/go",
            "PATH": "$GOROOT:$GOPATH:$GOROOT/bin",
        },
        "gscomplete_enabled": true,
        "fmt_enabled": true,
        "fmt_tab_indent": false,
        "fmt_tab_width": 4,
    
        "autocomplete_snippets": true,
        "autocomplete_tests": true,
        "autocomplete_builtins": true,
        "autocomplete_closures": true,
        "autocomplete_suggest_imports": true,
        "calltips": true,
        "use_named_imports": true,
        "autoinst": true,
        "ipc_timeout": 1,
        "fmt_cmd": ["goimports"],
        "on_save": [
            {"cmd": "gs_comp_lint"},
            {"cmd": "goimports"}
        ],
        "lint_enabled": true,
        "linters": [
            {"cmd": ["go", "run"]}
        ],
        "comp_lint_enabled": true,
        "comp_lint_commands": [
            {"cmd": ["go", "install"]}
        ],
    }
    
    ```
  
    ```
    keybings
    {
    		"keys": ["ctrl+q"],
    		"command": "gs_doc",
    		"args": {"mode": "hint"},
    		"context": [{ "key": "selector", "operator": "equal", "operand": "source.go" }]
    }
    ```
  
    ```
    ~/Library/Application Support/Sublime Text/Packages/GoSublime 新建 margo 文件夹
    cp src/margo.sh/extension-example/extension-example.go margo
    cp -R margo src/margo.sh/vendor
    ```
  
- **terminus**
  
  ```
  [
  	{ "keys": ["control+command+t"], "command": "toggle_terminus_panel" }
  ]
  ```
  
- **wakatime**

- **Side-bar keybind**

  ```
  { "keys": ["command+b"],"command": "toggle_side_bar" }
  ```


