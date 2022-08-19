---
title: "Sublime 配置"
date: 2022-08-19T17:29:11+08:00
type: post
tags: ['tools']
---

## sublime config

一个项目目录包含 ``java`` 和 ``go`` 两种语言，``idea`` 加载 java 项目直接就不显示 ``go`` 项目的文件夹了，就用 ``sublime`` 打开了，顺便记录配置一下 ``sublime``。

顺带记录一下在 mac 上显示 鼠标操作的app，``KeyCastr``。

### plugin
- **gosublime**
  
  1. git clone https://margo.sh/GoSublim
  2. ctrl . & ctrl x
  
- **terminus**
	keyBinds
	
	```
	[
		{ "keys": ["command+shift+t"], "command": "toggle_terminus_panel" }
	]
	```
- **GitGutter**

- **Side-bar keybind**

  ```
  { "keys": ["command+shift+t"], "command": "toggle_terminus_panel" },
  { "keys": ["command+b"],"command": "toggle_side_bar" }
  ```

  

- **CodeTimeTracker**
     - Use command (Ctrl+Shift+p) and type 'CodeTimeTracker' or 'tracker' and choose the option "CondeTimeTracker: Open Dashboard".
  
  -  Open menu 'Tools'->'CodeTimeTracker'->'CodeTimeTracker:Open Dashboard' Dashboard will appear in you default browser with the address http://localhost:10123/CodeTimeTracker/
  

