﻿# 机器学习实践指南  
	案例应用解析,第二版,机械工业出版社,ISBN:978-7-111-54021-2 
***
# 以下是R语言基础
***
* **R语言基础语法和操作:**
**[机器学习第4章第2节 : R语言科学计算](http://blog.csdn.net/u013733326/article/details/78318624)**
**[机器学习第4章第1节 : R语言基本语法及操作](http://blog.csdn.net/u013733326/article/details/78278636)**
***
# 以下是Python声音基础
***

* **将文字加密至音频,并绘制出音频的图谱:**
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
"""
hideMessagesToWav.py

这里的隐藏策略是:
    首先产生一段正弦波的噪声,
    然后在这一段噪声中隐藏一段文本文件的内容.
    这里使用简单的加密算法:
        字符的ASCII码乘以自动的整数后,
        减去64与该整数的乘积
        信息隐藏格式:
            ----------------------------------------------------------------------------
            |  区域  |  随机噪声区  |  加密字符区  | 随机噪声区  | 加密字符区  |  ......  |
            ----------------------------------------------------------------------------
            |  长度  |   interval   |     1       |  interval  |    1       |          |
            ----------------------------------------------------------------------------
@author: Oscar
"""
import wave
import pylab as pl
import numpy as np

print(u"正在将文件编码到声音.....")
print(u"正在打开文件.....")
fileOpen = wave.open(r"wav/pltest.wav","wb")
messageFile  = open("messages.txt")
try:
    #读取文件内容
    text = messageFile.read()
finally:
    #读取完毕关闭文件
    messageFile.close()
# map()函数接收两个参数，一个是函数，一个是序列，map将传入的函数依次作用到序列的每个元素，并把结果作为新的list返回
# ord()函数主要用来返回对应字符的ascii码
# map(func,*iterables)
wordDataAsAscii = list(map(ord,text))#这里我发现直接使用map是有问题的,所以使用list进行转换
# 这里是以list类型的变量为参数产生数组,这里wordDataAsASCII成为了一维数组
wordDataAsAscii = np.array(wordDataAsAscii)
wordDataAsAsciiLength = len(wordDataAsAscii.tolist())
#设置波形参数
#采样率
framerate = 44100
#声道数
nchannels = 2
#每位宽度
sampwidth = 2
#长度
nframes = framerate *4
#振幅
base_amplitude = 200;
max_amplitude = base_amplitude * 128
#每个字符的间隔
interval = (nframes -10 ) / wordDataAsAsciiLength;
#每周期样本数
wave_data = np.zeros((nframes),dtype=np.short)

count = 0

#写噪声数据和隐藏文字的字符
randomNum = np.random.rand(nframes)#rand(d0, d1, ..., dn)	Random values in a given shape.
for currentPos in range(0,nframes):
    if currentPos % interval ==0 and count < wordDataAsAsciiLength:
        #将隐藏文字的字符通过一定的变化写入噪声数据
        posSamp = (wordDataAsAscii[count] * base_amplitude) - (64*base_amplitude)
        count += 1
    elif currentPos % 60 == 0:
        #生成随机噪声数据
        posSamp = int((randomNum[currentPos] * max_amplitude) - (max_amplitude / 2))
    else:
        posSamp = 0
    wave_data[currentPos] = posSamp

#写波形数据参数
print("wiriting wav file......")
str_data = wave_data.tostring()
fileOpen.setnchannels(nchannels)
fileOpen.setframerate(framerate)
fileOpen.setsampwidth(sampwidth)
fileOpen.setnframes(nframes)
fileOpen.writeframes(str_data)
print("file has been saved!")

#绘制图形
wave_data.shape = -1,2
wave_data = wave_data.T
time = np.arange(0,nframes / 2)
pl.subplot(211)
pl.plot(time,wave_data[0],c="r")
pl.xlabel("time (seconds) L")
pl.subplot(212)
pl.plot(time,wave_data[0],c="g")
pl.xlabel("time (seconds) R")

```
-	**运行后如下图:**  <br />
![LowerVolume](/python%E5%9F%BA%E7%A1%80/%E5%A3%B0%E9%9F%B3%E5%9F%BA%E7%A1%80/saveImg/hideMessagesToWav.png)  

***
* **降低音量,并绘制出原音频和降低后音频的图谱:**
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
"""
Created on Wed Aug  9 08:55:09 2017

LowerVolume.py

减低音量的算法
音量降低可通过将采样波形变小来实现,具体来说,就是把每个采样数据按一定比例缩小
同时将缩小的幅度控制在合理的范围内,保证音量降低后声音仍然清晰
@author: Oscar
"""

import wave 
import numpy as np
import pylab as pl

print("working.....")
def waveChange(x,dwmax,dwmin):
    if x != 0:
        if abs(x) < dwmax and abs(x) > dwmin:
            x *= 0.5
        else:
            x *= 0.2
    return x

#打开wav文件
file = wave.open(r"wav/speak.wav","rb")
fileOUT = wave.open(r"wav/lowerVolumeSpeak.wav","wb")

#读取波形数据
"""
笔记:
    getparams：一次性返回所有的WAV文件的格式信息，它返回的是一个组元(tuple)：
    声道数,      量化位数（byte单位）, 采样频率,  采样点数,  压缩类型,  压缩类型的描述。
    nchannels   sampwidth            framerate  nframes  comptype   compname
    wave模块只支持非压缩的数据，因此可以忽略最后两个信息：
"""
#(nchannels,sampwidth,framerate,nframes,comptype,compname)
params = file.getparams()
nchannels,sampwidth,framerate,nframes = params[:4]
str_data = file.readframes(nframes)

#将波形数据转换为数组,并更改:
print("changing wav data......")
wave_data = np.fromstring(str_data,dtype=np.short)
#------start----
#我不知道为什么要重复下面这几句话,但是是书上的例子,我还是写下来了
params = file.getparams()
nchannels,sampwidth,framerate,nframes = params[:4]
str_data = file.readframes(nframes)
#------end------

#降低音量
"""
与放大音量相似,以原声音波形的最大值为基准,计算上限和下限,以waveChange为回调函数,降低音量
"""
change_dwmax = wave_data.max() / 100 * 1
change_dwmin = wave_data.max() / 100 * 0.5
"""
frompyfunc，把Python里的函数（可以是自写的）转化成ufunc，
用法是frompyfunc(func, nin, nout)，其中func是需要转换的函数，nin是函数的输入参数的个数，nout是此函数的返回值的个数。
注意frompyfunc函数无法保证返回的数据类型都完全一致,
因此返回一个中间类型object，需要再次obj.astype(np.float64)之类将其元素类型强制调齐。
"""
wave_change = np.frompyfunc(waveChange,3,1)
out_wave_data = wave_change(wave_data,change_dwmax,change_dwmin)
out_wave_data = out_wave_data.astype(wave_data.dtype)
out_str_data = out_wave_data.tostring()

#把波形参数写入磁盘
print("saving new wav file......")
fileOUT.setnchannels(nchannels)
fileOUT.setframerate(framerate)
fileOUT.setsampwidth(sampwidth)
fileOUT.writeframes(out_str_data)
print("new wav file saved!")

#绘制原声音波形图
wave_data.shape = -1,2
wave_data = wave_data.T
time = np.arange(0,nframes) * (1.0 / framerate)

pl.subplot(2,2,1)#亦可写作pl.subplot(221)
pl.plot(time,wave_data[0],c="b")
pl.xlabel("time (seconds)")
pl.ylabel("Original volume")
pl.subplot(222)#亦可写作pl.subplot(2,2,2)
pl.plot(time,wave_data[1],c="b")
pl.xlabel("time (seconds)")

#绘制降低后音量的波形图
out_wave_data.shape = -1,2
out_wave_data = out_wave_data.T
time = np.arange(0,nframes) * (1.0 / framerate)

pl.subplot(2,2,3)#亦可写作pl.subplot(223)
pl.plot(time,out_wave_data[0],c="r")
pl.xlabel("time (seconds)")
pl.ylabel("Lower volume")
pl.subplot(224)#亦可写作pl.subplot(2,2,4)
pl.plot(time,out_wave_data[1],c="r")
pl.xlabel("time (seconds)")

#关闭打开的文件
file.close()
fileOUT.close()
#显示图形
pl.show()

```
-	**运行后如下图:**  <br />
![LowerVolume](/python%E5%9F%BA%E7%A1%80/%E5%A3%B0%E9%9F%B3%E5%9F%BA%E7%A1%80/saveImg/LowerVolume.png)  



***
* **增加音量,并绘制出原音频和新音频的图谱:**
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
"""
放大音量
LouderVolume.py.
为包证声音质量,需要对音量调节范围,设置上限和下限(以原声音为基准,计算上限和下限),
为此编写waveChange(x,dwmax,dwmin)函数,计算调整后的数据,x为每次采样的波形数据,
dwmax为上限,dwmin为下限,该函数仅仅会将上限和下限之间的区域内的数据放大为原来的1.5倍,
在此区域外的数据设置为上限和下限
@author: Oscar
"""
import wave
import pylab as pl
import numpy as np
def waveChange(x,dwmax,dwmin):
    """
        x -- 每次采样的波形数据
        dwmax -- 上限
        dwmin -- 下限
    """
    if x != 0:
        if abs(x) > dwmax:
            x = x / abs(x) * dwmax
        elif abs(x) < dwmin:
            x = x / abs(x) * dwmin
        else:
            x = x * 1.5
    return x
#打开wav文档
file = wave.open(r"wav/speak.wav","rb")
fileOUT = wave.open(r"wav/OUT.wav","wb")
#(nchannels,sampwidth,framerate,nframes,comptype,compname)
params = file.getparams()
nchannels,sampwidth,framerate,nframes = params[:4]
print("reading wav file......")
str_data = file.readframes(nframes)
#将波形数据转换为数组,并更改
print("changing wav file......")
wave_data = np.fromstring(str_data,dtype=np.short)
#---start---
#我不知道为什么要重复这几句话,但是是书上的例子,我还是写下来了
params = file.getparams()
nchannels,sampwidth,framerate,nframes = params[:4]
str_data = file.readframes(nframes)
#---end---

#放大音量
"""
为保证放大后不失真,可采用以原声音为基准的放大策略,声音波形图像内室正弦函数图像,
再以时间轴为X轴,采样数据为Y轴的坐标系中,波形数据可正可负,上下波动
因此,以原声音数据的最大值为依据计算上下限,上限为原声音数据最大值的88%,下限为原声音数据最大值的14%
"""
change_dwmax = wave_data.max() / 100 * 88
change_dmin = wave_data.max() / 100 * 14
"""
frompyfunc，把Python里的函数（可以是自写的）转化成ufunc，
用法是frompyfunc(func, nin, nout)，其中func是需要转换的函数，nin是函数的输入参数的个数，nout是此函数的返回值的个数。
注意frompyfunc函数无法保证返回的数据类型都完全一致,
因此返回一个中间类型object，需要再次obj.astype(np.float64)之类将其元素类型强制调齐。
"""
wave_change = np.frompyfunc(waveChange,3,1)
new_wave_data = wave_change(wave_data,change_dwmax,change_dmin)
new_wave_data = new_wave_data.astype(wave_data.dtype)
new_str_data  = new_wave_data.tostring()

#写波形数据参数,配置声道数、量化位数和取样频率
print("saving new wav file ...")
fileOUT.setnchannels(nchannels)
fileOUT.setframerate(framerate)
fileOUT.setsampwidth(sampwidth)

#将new_str_data转换为二进制数据写入文件
fileOUT.writeframes(new_str_data)

#绘制原声音声音波形
wave_data.shape = -1,2
wave_data = wave_data.T
time = np.arange(0,nframes) * (1.0 / framerate)
pl.subplot(2,2,1)
pl.plot(time,wave_data[0])
pl.subplot(2,2,2)
pl.plot(time,wave_data[1],c="g")
pl.xlabel("time (seconds)")

#绘制放大声音声音波形
new_wave_data.shape = -1,2
new_wave_data = new_wave_data.T
new_time = np.arange(0,nframes) * (1.0 / framerate)
pl.subplot(2,2,3)
pl.plot(new_time,new_wave_data[0])
pl.subplot(2,2,4)
pl.plot(new_time,new_wave_data[1],c="g")
pl.xlabel("time (seconds)")

file.close()
fileOUT.close()

pl.show()
```
-	**运行后如下图:**  <br />
![LouderVolume](/python%E5%9F%BA%E7%A1%80/%E5%A3%B0%E9%9F%B3%E5%9F%BA%E7%A1%80/saveImg/LouderVolume.png)  


***
* **绘制声音的图谱:**  
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
"""
绘制波形图

plottingWaveform.py

This is a temporary script file.
"""

import wave
import pylab as pl
import numpy as np
print('working')
#打开wav文档
file = wave.open(r"wav/back.wav","rb")

#读取格式信息
#(nchannels,sampwidth,framerate,nframes,comptype,compname)
params = file.getparams()
nchannels,sampwidth,framerate,nframes = params[:4]

#读取波形数据
str_data = file.readframes(nframes)
file.close()

#将波形数据装换成数组gt
wave_data = np.fromstring(str_data,dtype=np.short)
wave_data.shape = -1,2

wave_data = wave_data.T
time = np.arange(0,nframes) * (1.0 / framerate)
#h绘制波形
"""
subplot(mnp) / (m,n,p)是将多个图画到一个平面上的工具。
其中，m表示是图排成m行，n表示图排成n列，也就是整个figure中有n个图是排成一行的，一共m行，
如果m=2就是表示2行图。p表示图所在的位置，p=1表示从左到右从上到下的第一个位置。
"""
pl.subplot(2,1,1)
pl.plot(time,wave_data[0])
pl.subplot(2,1,2)

pl.plot(time,wave_data[1],c="g")
pl.xlabel("time (seconds)")
```
-	**运行后如下图:**  <br />
![plottingWaveform](/python%E5%9F%BA%E7%A1%80/%E5%A3%B0%E9%9F%B3%E5%9F%BA%E7%A1%80/saveImg/plottingWaveform.png)  

***
# 以下是python图像基础
***
* **图像文字信息隐藏的解密:**
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
"""
showHideMessage.py

在之前的信息隐藏后,肉眼观察载体图像,仍无法察觉与之前相比有任何变化.
解密信息与隐藏信息相反,是隐藏信息的逆过程
图像隐藏信息解密原理:
    提取载体文件中蓝色像素值为奇数的像素点
@author: Oscar
"""

import cv2
import numpy as np

fn = "saveImg/secretImg.png"
if __name__ == "__main__":
    print(u"正在读取文件......")
    img = cv2.imread(fn)
    height = img.shape[0]
    width = img.shape[1]
    imgInfo = np.zeros((height,width,3),np.uint8)
    for j in range(0,height):
        for i in range(0,width):
            if (img[j,i,0] % 2) == 1:
                #如果蓝色值为奇数,则该像素点为文字
                imgInfo[j,i,1] = 255
        print(u"正在解密......",round(100*j / height),"%")
    cv2.imshow('imgInfo',imgInfo)
    #cv2.imwrite(fn,imgInfo)
    cv2.waitKey()
    cv2.destroyAllWindows()
```
-	**运行后如下图(隐藏信息后的文件):**  <br />
![showHideMessage](/python%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%83%8F%E5%9F%BA%E7%A1%80/saveImg/showHideMessage.png)  

***
* **图像的文字信息隐藏:**
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
"""
hideMessages.py

图像隐藏信息的原理:
    首先从源图中提取文字图像信息
    记录这个文字图像信息像素点在图像矩阵的位置
    对载体文件进行预处理,将蓝色像素值全部设为偶数
    最后将记录的文字信息像素点在载体文件对应位置的蓝色像素值设为奇数
图像隐藏信息解密原理:
    提取载体文件中蓝色像素值为奇数的像素点
    
@author: Oscar
"""

import cv2

#含有文字的图像
fnMessageImg = "img/test1.jpg"
#载体文件
fnBaseImg = "img/test2.jpg"
#包含隐藏信息的载体文件
fnSecretImg = "img/secret.jpg"

#注意是(b,g,r),而不是(r,g,b)
redColor=(0,0,255)
if __name__ == "__main__":
    print(u"正在处理中......")
    #图像大小
    messageImg = cv2.imread(fnMessageImg)
    baseImg = cv2.imread(fnBaseImg)
    height = messageImg.shape[0]
    width = messageImg.shape[1]
    
    #加上要隐藏的信息
    cv2.putText(messageImg,"Hello World!",(20,300),cv2.FONT_HERSHEY_PLAIN,2.0,redColor,thickness = 2)
    
    cv2.namedWindow('messageImg')
    cv2.imshow('messageImg',messageImg)
    cv2.namedWindow('baseImg')
    cv2.imshow('baseImg',baseImg)
    
    #处理隐藏信息载体图
    #将所有蓝色值变成偶数
    for j in range(0,height):
        for i in range(0,width):
            if (baseImg[j,i,0] % 2) == 1:
                baseImg[j,i,0] -= 1
        print("changing......",round(50*j / height),"%")
        mirror_w = width /2
    
    #读取源图,将信息写人目标图,将有信息的像素点的蓝色值设为奇数
    for j in range(0,height):
        for i in range(0,width):
            if(messageImg[j,i,0],messageImg[j,i,1],messageImg[j,i,2]) == redColor:
                baseImg[j,i,0] += 1
        print("writing.......",round(50*j / height)+50,"%")
    
    #保存修改后的目标图,并显示
    cv2.namedWindow('secretImg')
    cv2.imshow('secretImg',baseImg)
    cv2.imwrite(fnSecretImg,baseImg)
    cv2.waitKey()
    cv2.destroyAllWindows()
```
-	**运行后如下图(源文件):**  <br />
![messageImg](/python%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%83%8F%E5%9F%BA%E7%A1%80/saveImg/messageImg.png)    
-	**运行后如下图(载体文件):**  <br />
![baseImg](/python%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%83%8F%E5%9F%BA%E7%A1%80/saveImg/baseImg.png)  
-	**运行后如下图(隐藏信息后的文件):**  <br />
![secretImg](/python%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%83%8F%E5%9F%BA%E7%A1%80/saveImg/secretImg.png)  


***
* **平铺图片:**
```python
#!/user/bin/env python
# -*- coding: utf-8 -*-
"""
平铺图片

@author: Oscar
"""
import cv2
import numpy as np

fn = "img/6.1.jpg"
if __name__ == '__main__':
    print('loading %s...' %fn)
    img = cv2.imread(fn)
    height = img.shape[0]
    width = img.shape[1]
    
    #横向平铺5个像素
    sz1 = width * 5
    #纵向平铺2像素
    sz0 = height * 2
    #创建空白图像,然后接将图片排列
    myimg = np.zeros((sz0,sz1,3),np.uint8)
    #逐个图像复制
    img_x = 0
    img_y = 0
    for now_y in range(0,sz0):
        #增加行数
        for now_x in range(0,sz1):
            #复制对应位置的图像像素点
            myimg[now_y,now_x,0] = img[img_y,img_x,0]
            myimg[now_y,now_x,1] = img[img_y,img_x,1]
            myimg[now_y,now_x,2] = img[img_y,img_x,2]
            #增加列数
            img_x += 1
            if img_x >= width:
                img_x = 0
        img_y += 1
        if img_y >= height:
            img_y = 0
        print(round((now_y * 100) / (height * 2)),'%')
    cv2.namedWindow('img1')
    cv2.imshow('img', myimg)
    cv2.waitKey()
    cv2.destroyAllWindows()
```
-	**运行后如下图:**  <br />
![TileImgpy](/python%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%83%8F%E5%9F%BA%E7%A1%80/saveImg/TileImgpy.png)  

***
* **图片的负片+水印效果:**
```python
#!/user/bin/env python
# -*- coding: utf-8 -*-
"""
负片效果

@author: Oscar
"""
import cv2
import numpy as np

fn = "img/test1.jpg"
if __name__ == '__main__':
    print('loading %s...' %fn)
    img = cv2.imread(fn)
    height = img.shape[0]
    width = img.shape[1]
    ii = 0
    #生成负片
    #通道分离，顺序BGR不是RGB
    b,g,r = cv2.split(img)
    b = 255 - b
    g = 255 - g
    r = 255 - r
    #直接通过索引改变色彩分量
    img[:,:,0] = b
    img[:,:,1] = g
    img[:,:,2] = r
    #加上水印
    cv2.putText(img,"第一个水印效果",(20,20),cv2.FONT_HERSHEY_PLAIN,2.0,(0,0,0),thickness=2)
    cv2.namedWindow('负片+水印')
    cv2.imshow('负片+水印',img)
    cv2.waitKey()
    cv2.destroyAllWindows()
```
-	**运行后如下图:**  <br />
![negativeImg](/python%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%83%8F%E5%9F%BA%E7%A1%80/saveImg/negativeImg.png)  


***
* **图像的日落效果:**
```python
# -*- coding: utf-8 -*-
"""
图像的日落效果
将蓝色值和绿色值设为原来的70%,红色值不变
"""
import cv2
import numpy as np #在python3中,xrange被移除,我这里使用的是range,这句话便没有用到
fn="img/test1.jpg"

if __name__ == '__main__':
    img = cv2.imread(fn)
    w=img.shape[1]
    h=img.shape[0]    
    ii=0
    #生成日落效果
    #b[:,:] = img[:,:,0]  
    #g[:,:] = img[:,:,1]  
    #r[:,:] = img[:,:,2]  
    for xi in range(0,w):
        for xj in range (0,h):
            img[xj,xi,0]= int(img[xj,xi,0]*0.7)
            img[xj,xi,1]= int(img[xj,xi,1]*0.7)
        if  xi%10==0 :print('.'),          
    cv2.namedWindow('img')     
    cv2.imshow('img', img) 
    cv2.waitKey()
    cv2.destroyAllWindows()
```
-	**运行后如下图:**  <br />
![sunset](/python%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%83%8F%E5%9F%BA%E7%A1%80/saveImg/sunset.png)  

***
* **调整图像亮度:**  
```python
# -*- coding: utf-8 -*-
"""
Adjust brightness
调整图像亮度
"""

import cv2
fn = 'img/test1.jpg'
if __name__ == '__main__':
    print('loading %s ...' %fn)
    print(u'处理中....')
    status = 'dark'
    if status == 'light':
        value = 1.2
    elif status == 'dark' :
        value = 0.2
    else:
        value = 1
    img = cv2.imread(fn)
    #第二维度长度
    w = img.shape[1]
    #第一维度长度
    h = img.shape[0]
    ii = 0
    #将全部色彩变暗
    for xi in range(0,w):
        for xj in range(0,h):
            #将像素值整体减少,设为原像素的20%
            img[xj,xi,0] = int(img[xj,xi,0] * value)
            img[xj,xi,1] = int(img[xj,xi,1] * value)
            img[xj,xi,2] = int(img[xj,xi,2] * value)
        #显示进度条
        if xi % 10 == 0:
            print('当前状态:',status,'当前进度:' , round(xi * 100 / w) ,'%')
    cv2.namedWindow('img')
    cv2.imshow('img' + status,img)
    cv2.waitKey()
    cv2.destroyAllWindows()
```
-	**运行后如下图(dark):**  <br />
![AdjustBrightness_dark](/python%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%83%8F%E5%9F%BA%E7%A1%80/saveImg/AdjustBrightness_dark.png)  
-	**运行后如下图(light):**  <br />
![AdjustBrightness_light](/python%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%83%8F%E5%9F%BA%E7%A1%80/saveImg/AdjustBrightness_light.png)  

***
* **图像基础,读取图像:**
```python
# -*- coding: utf-8 -*-

import cv2 as cv2
fn = "img/test1.jpg"

if __name__ == '__main__':
    print ('loading %s... ' % fn)
    img = cv2.imread(fn)
    cv2.imshow('preview...',img)
    cv2.waitKey()
    cv2.destroyAllWindows()
```
-	**运行后如下图:**  <br />
![imageBase](/python%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%83%8F%E5%9F%BA%E7%A1%80/saveImg/imageBase.png)  


***
* **随机生成像素:**
```python
# -*- coding: utf-8 -*-
# 随机生成像素
"""
    首先产生空图像矩阵,
    然后确定矩阵的2000个随机位置
    最后将随机位置处的像素值设置为随机数组
"""
import numpy as np
import cv2

if __name__ == '__main__':
    #行数
    sz1 = 200
    #列数
    sz2 = 300
    print('产生的空图像矩阵为(%s,%s)' % (sz1,sz2))
    img = np.zeros((sz1,sz2,3),np.uint8)
    pos1 = np.random.randint(200,size = (5000,1)) #行位置随机数组
    pos2 = np.random.randint(300,size = (5000,1)) #列位置随机数组
    #在随机位置设置像素点值
    for i in range(5000):
        img[pos1[i],pos2[i],[0]] = np.random.randint(0,255)
        img[pos1[i],pos2[i],[1]] = np.random.randint(0,255)
        img[pos1[i],pos2[i],[2]] = np.random.randint(0,255)
    #显示图像
    cv2.imshow('preview...',img)
    #等待按键
    cv2.waitKey()
    #销毁窗口
    cv2.destroyAllWindows()
```
-	**运行后如下图:**  <br />
![randomXY](/python%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%83%8F%E5%9F%BA%E7%A1%80/saveImg/randomXY.png)  

***
* **正余弦图像的绘制:**   
```python  
#coding:utf-8
"""
Numpy_pylab_matplotlib.py
"""
import numpy as np
import matplotlib.pyplot as plt

#sin函数绘制
x = np.arange(-8,8,0.01)
y = np.sin(x)

plt.plot(x,y)
plt.show()

#余弦函数绘制
y = np.cos(x)
plt.plot(x,y)
plt.show()
```
-	**运行后如下图(sin):**  <br />
![Numpy_pylab_matplotlib_sin](/python%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%83%8F%E5%9F%BA%E7%A1%80/saveImg/Numpy_pylab_matplotlib_sin.png)  

-	**运行后如下图(cos):**  <br />
![Numpy_pylab_matplotlib_cos](/python%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%83%8F%E5%9F%BA%E7%A1%80/saveImg/Numpy_pylab_matplotlib_cos.png)

***
* **异常的主动抛出:**
```python
#coding:utf-8
try:
	raise NameError("主动抛出异常:NameError")
except NameError:
	print("异常被抛出!原因是:NameError")
	raise
```
-	**运行后如下图:**  <br />
![tryExceptRaise](/python%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%83%8F%E5%9F%BA%E7%A1%80/saveImg/tryExceptRaise.png)

***
* **异常的捕获:**
```python
#coding:utf-8
while True:
	try:
		x = int(input("请输入一个数字:"))
		break
	except ValueError:
		print("异常被抛出!")
		break
```  
-	**运行后如下图:**  <br />
![tryExcept](/python%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%83%8F%E5%9F%BA%E7%A1%80/saveImg/tryExcept.png)

***
* **自定义class的学习:**  
```python
class Complex:
	"""docstring for Complex"""
	r = 0 
	i = 0
	def __init__(self, realPart,imagePart):
		self.r,self.i = realPart,imagePart
x= Complex(3.0,-4.5)
print(x.r,x.i)
```
-	**运行后如下图:**  <br />
![class](/python%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%83%8F%E5%9F%BA%E7%A1%80/saveImg/class.png)

***

* **自定义函数的学习:**  
```python
#coding:utf-8
def fib(n):
	"""fibonacci series"""
	a,b = 0,1
	while a < n:
		print(a)
		a,b = b, a+b
fib(2000)
```
- 	**运行后如下图:**  <br />
![def](/python%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%83%8F%E5%9F%BA%E7%A1%80/saveImg/def.png)

***

