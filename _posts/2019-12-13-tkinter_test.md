---
layout: post
title: Tkinter による単純な GUI アプリ作成
tags: GUI, python
---

本稿では，Tkinter を利用して簡単な GUI アプリ（degree を入力すると radian が帰ってくる）を作ります．言語はpythonです．

## 目次
- 参考文献
- プログラムのGUI
- プログラムの全体

## 参考文献
Ref. 1, TkDocs (https://tkdocs.com/tutorial/firstexample.html)  
Ref. 1 の Feet を meter に変換するプログラムを参考に，degree を radian に変換するプログラムを作成します．

## プログラムのGUI
<img src="{{ site.baseurl }}/images/2019-11-13-figure/tkinter.PNG" style="width: 200px;"/>  

## プログラムの全体
```
import sys
import math
import tkinter as tk
from tkinter import ttk

def deg2rad(*args):
    try:
        value = float(degree.get())
        radian.set(value*math.pi/180.0)
    except ValueError:
        pass

root = tk.Tk()

# window title
root.title("degree to radian")

# window size, grid arrangement
mainframe = ttk.Frame(root, padding="3 3 12 12")
mainframe.grid(column=0, row=0, sticky=(tk.N, tk.W, tk.E, tk.S))
root.columnconfigure(0, weight=1)
root.rowconfigure(0, weight=1)

# variables
degree = tk.StringVar()
radian = tk.StringVar()

###                        column
###           |    1    |    2    |    3    |
###       | 1 | (empty) | textbox | "degree"|
###  row  | 2 |"is eq~" |  result | "radian"|
###       | 3 |         |         |  Button |

degree_entry = ttk.Entry(mainframe, width=7, textvariable=degree)                  # degree input
degree_entry.grid(row=1, column=2, sticky=(tk.W, tk.E))                            # 1-2
ttk.Label(mainframe, text="degree").grid(row=1, column=3, sticky=tk.W)             # 1-3
ttk.Label(mainframe, text="is equivalent to").grid(row=2, column=1, sticky=tk.E)   # 2-1
radian_output = ttk.Label(mainframe, textvariable=radian)                          # radian output
radian_output.grid(row=2, column=2, sticky=(tk.W, tk.E))                           # 2-2
ttk.Label(mainframe, text="radian").grid(row=2, column=3, sticky=tk.W)             # 2-3

ttk.Button(mainframe, text="Calculate", command=deg2rad).grid(column=3, row=3, sticky=tk.W) # 3-3


for child in mainframe.winfo_children():
    child.grid_configure(padx=5, pady=5)

degree_entry.focus()
# press <Return> and calculates radian
root.bind('<Return>', deg2rad)

###
root.mainloop()
###
```
