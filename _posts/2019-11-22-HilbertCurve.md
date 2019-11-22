---
layout: post
title: Hilbert 曲線をpythonで描く
tags: 木構造
---

本稿では，Hilbert 曲線を書くプログラムを作成します．

## 目次
- 背景
- 2次元での Hilbert 曲線
- 3次元での Hilbert 曲線 
- 参考文献
- 2次元のプログラム
- 3次元のプログラム

## 背景
Hilbert 曲線は，空間充填曲線のほとつです．四分木・八分木を用いて空間分割するとき，葉ノード（子ノードを持たないノード）を一次元的に結ぶために，Z-order 曲線や Hilbert 曲線といった，空間充填曲線を利用する場合があります．本稿では、四分木・八分木をベースに、Hilbert 曲線を書くプログラムを作成します．

## 2次元での Hilbert 曲線
### 同じ親を持つノードの位置関係
同じ親を持つノードの位置関係を下図のように定めます．つまり，  
1 : 左下  
2 : 右下  
3 : 左上  
4 : 右上  
に位置するノードです．（この子ノードを1から4まで順に結ぶと，Z-orderになります．）．ここで，子セルにつけられる番号を，order と定義します．

[<img src="{{ site.baseurl }}/images/2019-11-22-figure/fig1.png" width=250>]

### Hilbert 曲線の状態
二次元の Hilbert 曲線は，Fig. 2の4つの状態（state）から構成されます．例えばstate 1のHilbert曲線は，左下→左上→右上→右下の位置を順に通ります．これをZ-order 曲線で配置された子ノードの番号に対応させると，state 1のHilbert 曲線は  
state 1 : 1342  
と表現できます．state 2 ~ 4も同様に，  
state 2 : 1243  
state 3 : 4312  
state 4 : 4213  
と表現できます．

<img src="https://github.com/KeisukeSUGAYA/KeisukeSUGAYA.github.io/blob/master/images/2019-11-22-figure/fig2.png" width=250>


### Hilbert 曲線による再帰的な空間充填
子ノードの orderとHilbert 曲線のstate が決定すると，子ノードの子ノード（孫）に対するHilbert 曲線の state が決まります．例として，下図のようなstate 1 のHilbert 曲線を考えます．state 1のHilbert 曲線なので，子ノードは1342（左下→左上→右上→右下）の順に並にます．この4つの子ノードを分割し，16つの孫セルを生成します．すると，order 1の子セルを分割し得られる孫ノードは，state 2の順に並びます．order 2の子セル（右下）を分割し得られる孫セルは，state 3，order 3（左上）の子セルを分割し得られる孫セルは，state 2，order 4（右上）の子ノードを分割し得られる孫セルは，state 2になります．  
他のstate の場合も，子ノードのorderと，子ノードが従うHilbert曲線のstate が決定すると，孫ノードに対するHilbert曲線のstateが定まります．子ノードのorderと子ノードに対するstateから決定される孫セルのHilbert曲線のstateを，下表に示します．  

<img src="https://github.com/KeisukeSUGAYA/KeisukeSUGAYA.github.io/blob/master/images/2019-11-22-figure/fig3.png" width=250>  
<img src="https://github.com/KeisukeSUGAYA/KeisukeSUGAYA.github.io/blob/master/images/2019-11-22-figure/table1.png" width=550>


### 2次元 Hilbert 曲線の例
<img src="https://github.com/KeisukeSUGAYA/KeisukeSUGAYA.github.io/blob/master/images/2019-11-22-figure/fig4.gif" width=500>  

## 3次元での Hilbert 曲線 
### 同じ親を持つノードの位置関係
2次元と同様に考え，  
1 : 左　手前　下  
2 : 右　手前　下  
3 : 左　奥　　下  
4 : 右　奥　　下  
5 : 左　手前　上  
6 : 右　手前　上  
7 : 左　奥　　上  
8 : 右　奥　　上  
とします．

<img src="https://github.com/KeisukeSUGAYA/KeisukeSUGAYA.github.io/blob/master/images/2019-11-22-figure/fig5.png" width=250>  

### Hilbert 曲線の状態
三次元の Hilbert 曲線は，12つのstateから構成されます．Z-order 曲線で配置された子ノードの番号に対応させると，
state 1 : 1342685  
state 2 : 75684213  
state 3 : 12657843  
state 4 : 78431265  
state 5 : 15734862  
state 6 : 73156248  
state 7 : 68751342  
state 8 : 42137568  
state 9 : 65124378  
state 10 : 43786512  
state 11 : 62487315  
state 12 : 48621573  
と表現できます．  

<img src="https://github.com/KeisukeSUGAYA/KeisukeSUGAYA.github.io/blob/master/images/2019-11-22-figure/fig6.png" width=500>  


### Hilbert 曲線による再帰的な空間充填
order と state の関係は下表です．

<img src="https://github.com/KeisukeSUGAYA/KeisukeSUGAYA.github.io/blob/master/images/2019-11-22-figure/table2.png" width=550>


### 3次元 Hilbert 曲線の例

<img src="https://github.com/KeisukeSUGAYA/KeisukeSUGAYA.github.io/blob/master/images/2019-11-22-figure/fig7.svg" width=550>

## 参考文献
本稿を作成するために，以下を参考にしました．
Ref. １ : [フリー百科事典『ウィキペディア（Wikipedia）』](https://ja.wikipedia.org/wiki/%E3%83%92%E3%83%AB%E3%83%99%E3%83%AB%E3%83%88%E6%9B%B2%E7%B7%9A)  
Ref. 2 : Bader, M., “Space-Filling Curves An Introduction with Applications in Scientific Computing,” Springer  
doi : 10.1007/978-3-642-31046-1


## 2次元のプログラム
```
import os
import matplotlib.pyplot as plt
import matplotlib.cm as cm

class Position:
    def __init__(self):
        self.x = []
        self.y = []
        self.x.append(0)
        self.y.append(0)
    def R(self):
        self.x.append(self.x[len(self.x)-1]+1)
        self.y.append(self.y[len(self.y)-1]+0)
    def L(self):
        self.x.append(self.x[len(self.x)-1]-1)
        self.y.append(self.y[len(self.y)-1]+0)
    def U(self):
        self.x.append(self.x[len(self.x)-1]+0)
        self.y.append(self.y[len(self.y)-1]+1)
    def D(self):
        self.x.append(self.x[len(self.x)-1]+0)
        self.y.append(self.y[len(self.y)-1]-1)       

class Hilbert:
    def RUL(Position, n):
        if n <= 0: return
        Hilbert.URD(Position, n-1); Position.R()
        Hilbert.RUL(Position, n-1); Position.U()
        Hilbert.RUL(Position, n-1); Position.L()
        Hilbert.DLU(Position, n-1)
    def DLU(Position, n):
        if n <= 0: return
        Hilbert.LDR(Position, n-1); Position.D()
        Hilbert.DLU(Position, n-1); Position.L()
        Hilbert.DLU(Position, n-1); Position.U()
        Hilbert.RUL(Position, n-1)
    def LDR(Position, n):
        if n <= 0: return
        Hilbert.DLU(Position, n-1); Position.L()
        Hilbert.LDR(Position, n-1); Position.D()
        Hilbert.LDR(Position, n-1); Position.R()
        Hilbert.URD(Position, n-1)
    def URD(Position, n):
        if n <= 0: return
        Hilbert.RUL(Position, n-1); Position.U()
        Hilbert.URD(Position, n-1); Position.R()
        Hilbert.URD(Position, n-1); Position.D()
        Hilbert.LDR(Position, n-1)

pos = Position()
print("Hilbert Curve Program")
n_in = input("input the Order of Hilbert Curve : ")
n = int(n_in)
Hilbert.RUL(pos, n)

scatterX = []
scatterY = []
for j in range(2**n):
    for i in range(2**n):
        scatterX.append(i)
        scatterY.append(j)

# plt.xkcd()
# for i in range(len(pos.x)):
#     fig = plt.figure(figsize=(8,8))
#     plt.xlim(0-0.05, 2**n -0.95)
#     plt.ylim(0-0.05, 2**n -0.95)
#     plt.scatter(scatterX, scatterY, color="black")
#     plt.plot(pos.x[0:i+1],pos.y[0:i+1], color=cm.hsv(i/len(pos.x)))
#     plt.savefig('{0:04d}'.format(i)+".png")
#     plt.close()


# fig = plt.figure(figsize=(8,8))
# plt.xlim(0-0.05, 2**n -0.95)
# plt.ylim(0-0.05, 2**n -0.95)
# plt.scatter(scatterX, scatterY, color="black")
# for i in range(len(pos.x)-1):
#     plt.plot(pos.x[i:i+2],pos.y[i:i+2], color=cm.hsv(i/(len(pos.x)-1)))
#     plt.savefig('{0:04d}'.format(i)+".png")
# plt.show()
# plt.close()


fig = plt.figure(figsize=(8,8))
plt.xlim(0-0.05, 2**n -0.95)
plt.ylim(0-0.05, 2**n -0.95)
plt.scatter(scatterX, scatterY, color="black")
plt.plot(pos.x,pos.y, color="blue")
plt.savefig("Order_" +'{0:04d}'.format(n)+".svg")
plt.show()
plt.close()
```

## 3次元のプログラム
```
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D

class Position:
    def __init__(self):
        self.x = []
        self.y = []
        self.z = []
        self.x.append(0)
        self.y.append(0)
        self.z.append(0)
    def XP(self):
        self.x.append(self.x[len(self.x)-1]+1)
        self.y.append(self.y[len(self.y)-1]+0)
        self.z.append(self.z[len(self.z)-1]+0)
    def XM(self):
        self.x.append(self.x[len(self.x)-1]-1)
        self.y.append(self.y[len(self.y)-1]+0)
        self.z.append(self.z[len(self.z)-1]+0)
    def YP(self):
        self.x.append(self.x[len(self.x)-1]+0)
        self.y.append(self.y[len(self.y)-1]+1)
        self.z.append(self.z[len(self.z)-1]+0)
    def YM(self):
        self.x.append(self.x[len(self.x)-1]+0)
        self.y.append(self.y[len(self.y)-1]-1)
        self.z.append(self.z[len(self.z)-1]+0)
    def ZP(self):
        self.x.append(self.x[len(self.x)-1]+0)
        self.y.append(self.y[len(self.y)-1]+0)
        self.z.append(self.z[len(self.z)-1]+1)
    def ZM(self):
        self.x.append(self.x[len(self.x)-1]+0)
        self.y.append(self.y[len(self.y)-1]+0)
        self.z.append(self.z[len(self.z)-1]-1)

class Hilbert:
    def xpypzp(Position, n):
        if n <= 0: return
        Hilbert.ypzpxp(Position,n-1); Position.YP()
        Hilbert.zpxpyp(Position,n-1); Position.XP()
        Hilbert.zpxpyp(Position,n-1); Position.YM()
        Hilbert.xmymzp(Position,n-1); Position.ZP()
        Hilbert.xmymzp(Position,n-1); Position.YP()
        Hilbert.zmxpym(Position,n-1); Position.XM()
        Hilbert.zmxpym(Position,n-1); Position.YM()
        Hilbert.ypzmxm(Position,n-1) 

    def xpymzm(Position, n):
        if n <= 0: return
        Hilbert.ypzmxm(Position,n-1); Position.YM()
        Hilbert.zpxmym(Position,n-1); Position.XP()
        Hilbert.zpxmym(Position,n-1); Position.YP()
        Hilbert.xmypzm(Position,n-1); Position.ZM()
        Hilbert.xmypzm(Position,n-1); Position.YM()
        Hilbert.zmxmyp(Position,n-1); Position.XM()
        Hilbert.zmxmyp(Position,n-1); Position.YP()
        Hilbert.ypzpxp(Position,n-1)

    def ypzpxp(Position, n):
        if n <= 0: return
        Hilbert.zpxpyp(Position,n-1); Position.XP()
        Hilbert.xpypzp(Position,n-1); Position.ZP()
        Hilbert.xpypzp(Position,n-1); Position.XM()
        Hilbert.ymzpxm(Position,n-1); Position.YP()
        Hilbert.ymzpxm(Position,n-1); Position.XP()
        Hilbert.xpymzm(Position,n-1); Position.ZM()
        Hilbert.xpymzm(Position,n-1); Position.XM()
        Hilbert.zmxmyp(Position,n-1)
        
    def ypzmxm(Position, n):
        if n <= 0: return
        Hilbert.zpxmym(Position,n-1); Position.XP()
        Hilbert.xpymzm(Position,n-1); Position.ZM()
        Hilbert.xpymzm(Position,n-1); Position.XM()
        Hilbert.ymzmxp(Position,n-1); Position.YM()
        Hilbert.ymzmxp(Position,n-1); Position.XP()
        Hilbert.xpypzp(Position,n-1); Position.ZP()
        Hilbert.xpypzp(Position,n-1); Position.XM()
        Hilbert.zmxpym(Position,n-1)

    def zpxpyp(Position, n):
        if n <= 0: return
        Hilbert.xpypzp(Position,n-1); Position.ZP()
        Hilbert.ypzpxp(Position,n-1); Position.YP()
        Hilbert.ypzpxp(Position,n-1); Position.ZM()
        Hilbert.zpxmym(Position,n-1); Position.XP()
        Hilbert.zpxmym(Position,n-1); Position.ZP()
        Hilbert.ymzmxp(Position,n-1); Position.YM()
        Hilbert.ymzmxp(Position,n-1); Position.ZM()
        Hilbert.xmypzm(Position,n-1)

    def zpxmym(Position, n):
        if n <= 0: return
        Hilbert.xpymzm(Position,n-1); Position.ZM()
        Hilbert.ypzmxm(Position,n-1); Position.YM()
        Hilbert.ypzmxm(Position,n-1); Position.ZP()
        Hilbert.zpxpyp(Position,n-1); Position.XP()
        Hilbert.zpxpyp(Position,n-1); Position.ZM()
        Hilbert.ymzpxm(Position,n-1); Position.YP()
        Hilbert.ymzpxm(Position,n-1); Position.ZP()
        Hilbert.xmymzp(Position,n-1)

    def xmypzm(Position, n):
        if n <= 0: return
        Hilbert.ymzpxm(Position,n-1); Position.YP()
        Hilbert.zmxpym(Position,n-1); Position.XM()
        Hilbert.zmxpym(Position,n-1); Position.YM()
        Hilbert.xpymzm(Position,n-1); Position.ZM()
        Hilbert.xpymzm(Position,n-1); Position.YP()
        Hilbert.zpxpyp(Position,n-1); Position.XP()
        Hilbert.zpxpyp(Position,n-1); Position.YM()
        Hilbert.ymzmxp(Position,n-1) 

    def xmymzp(Position, n):
        if n <= 0: return
        Hilbert.ymzmxp(Position,n-1); Position.YM()
        Hilbert.zmxmyp(Position,n-1); Position.XM()
        Hilbert.zmxmyp(Position,n-1); Position.YP()
        Hilbert.xpypzp(Position,n-1); Position.ZP()
        Hilbert.xpypzp(Position,n-1); Position.YM()
        Hilbert.zpxmym(Position,n-1); Position.XP()
        Hilbert.zpxmym(Position,n-1); Position.YP()
        Hilbert.ymzpxm(Position,n-1) 

    def ymzpxm(Position, n):
        if n <= 0: return
        Hilbert.zmxpym(Position,n-1); Position.XM()
        Hilbert.xmypzm(Position,n-1); Position.ZM()
        Hilbert.xmypzm(Position,n-1); Position.XP()
        Hilbert.ypzpxp(Position,n-1); Position.YP()
        Hilbert.ypzpxp(Position,n-1); Position.XM()
        Hilbert.xmymzp(Position,n-1); Position.ZP()
        Hilbert.xmymzp(Position,n-1); Position.XP()
        Hilbert.zpxmym(Position,n-1)

    def ymzmxp(Position, n):
        if n <= 0: return
        Hilbert.zmxmyp(Position,n-1); Position.XM()
        Hilbert.xmymzp(Position,n-1); Position.ZP()
        Hilbert.xmymzp(Position,n-1); Position.XP()
        Hilbert.ypzmxm(Position,n-1); Position.YM()
        Hilbert.ypzmxm(Position,n-1); Position.XM()
        Hilbert.xmypzm(Position,n-1); Position.ZM()
        Hilbert.xmypzm(Position,n-1); Position.XP()
        Hilbert.zpxpyp(Position,n-1)

    def zmxpym(Position, n):
        if n <= 0: return
        Hilbert.xmypzm(Position,n-1); Position.ZM()
        Hilbert.ymzpxm(Position,n-1); Position.YP()
        Hilbert.ymzpxm(Position,n-1); Position.ZP()
        Hilbert.zmxmyp(Position,n-1); Position.XM()
        Hilbert.zmxmyp(Position,n-1); Position.ZM()
        Hilbert.ypzmxm(Position,n-1); Position.YM()
        Hilbert.ypzmxm(Position,n-1); Position.ZP()
        Hilbert.xpypzp(Position,n-1)

    def zmxmyp(Position, n):
        if n <= 0: return
        Hilbert.xmymzp(Position,n-1); Position.ZP()
        Hilbert.ymzmxp(Position,n-1); Position.YM()
        Hilbert.ymzmxp(Position,n-1); Position.ZM()
        Hilbert.zmxpym(Position,n-1); Position.XM()
        Hilbert.zmxpym(Position,n-1); Position.ZP()
        Hilbert.ypzpxp(Position,n-1); Position.YP()
        Hilbert.ypzpxp(Position,n-1); Position.ZM()
        Hilbert.xpymzm(Position,n-1)

pos = Position()
print("Hilbert Curve Program")
n_in = input("input the Order of Hilbert Curve : ")
n = int(n_in)
Hilbert.xpypzp(pos, n)

# print(pos.x,pos.y,pos.z)

fig = plt.figure()
ax = fig.gca(projection='3d')
ax.plot(pos.x, pos.y, pos.z)
plt.show()
```
