---
title: "RLE Image compression"
description: 
date: 2021-10-14T18:05:40+07:00
image: 
math: 
license:
categories: 
    - image processing
    - image compression
tags:
    - image
    - processing
    - compression
    - python
    - rle
hidden: false
comment: true
draft: false
---
## Apa itu image compression?

<div style="text-align: justify">
Sering kali dalam kehidupan sehari-hari kita menggunakan citra/gambar baik itu untuk keperluan akademik maupun sosial media namun apakah kita tahu bahwa sebenarnya citra digital yang kita gunakan sebenarnya merupakan interpretasi dari matriks, ya matriks yang sering kita pelajari dulu ketika di SMA. Sebuah citra digital merupakan matriks dua hingga tiga dimensi. Untuk citra hitam putih & grayscale sendiri tersusun dari matriks dua dimensi sedangkan citra berwarna (RGB) tersusun dari matriks tiga dimensi. Tahukan kamu bahwa hasil citra pada kamera sebenarnya berformat RAW?, citra berformat RAW ini memiliki ukuran yang besar sehingga perlu dimampatkan (kompres). Pemampatan ini bertujuan untuk mengurangi ukuran file agar lebih efisien. Secara garis besar ada dua teknik kompresi pada citra, yaitu lossless dan lossy. Pada teknik kompresi lossless citra akan direkonstruksi secara sempurna tanpa mengurangi kualitas citra sedangkan teknik kompresi lossy compression atau juga disebut irreversible compression merupakan teknik kompresi yang menggunakan perkiraan yang tidak tepak dan membuang sebagian data untuk mewakili konten. 
</div>

![Ilustrasi citra digital](img-num-ilustration.png "Ilustrasi citra digital")

## Konsep RLE (Run Length Encoding)
<div style="text-align: justify">
RLE (Run Length Encoding) merupakan salah satu teknik kompresi lossless yang paling mudah dipraktikkan. Konsep RLE Compression adalah dengan memeriksa pengulangan nilai pixel yang terjadi berturut-turut. Algoritma ini dapat bekerja dengan efisien pada citra biner.

Mari kita ambil contoh citra [ 0 1 1 1 1 1 0 0 0 0 0 0 1 1 1 0 0 0 1 1 1 ] memiiki 21 pixel dan terdiri dari binary. Untuk menggunakan algoritma RLE langkah - langkah yang perlu kita lakukan sebagai berikut.
1. Langkah awal, pointer saat ini ada pada pixel pertama yaitu  0 . Kemudian kita kompresi menjadi [01] yang artinya 0 diulang sebanyak 1 kali.
2. Selanjutnya bernilai 1 yang diulang sebanyak 5 kali setelah itu kita gabungkan pada hasil kompresi sebelumnya menjadi [01 15].
3. Kemudian bernilai 0 yang diulang sebanyak 6 kali dan kita gabungkan dengan hasil kompresi sebelumnya menjadi [01 15 06].
4. Ulang langkah sebelumnya sampai dengan pixel terakhir.
5. Hasil RLE [01 15 06 13 03 13]
Citra sebelumnya memiliki 21 nilai sekarang setelah dikompress menjadi 6 nilai.

Sekarang untuk mendekompresi atau mengembalikan nilai semula yaitu,
1. Nilai awal yaitu 01, yang berarti 0 diulang sebanyak satu kali. Menghasilkan [1].
2. Selanjutnya yaitu 15. Kita lakukan seperti sebelumnya, dan digabung dengan hasil sebelumnya menghasilkan [0 1 1 1 1 1].
3. Dan seterusnya.
</div>

![Ilustrasi RLE](rle-illustration.webp "Ilustrasi RLE")

## Implementasi pada Python

### Persiapan Awal

Import dependensi dan deklarasi fungsi pendukung yang akan digunakan.
```python
import numpy as np
import cv2
import matplotlib.pyplot as plt
import sys
import os

# Fungsi untuk menampilkan citra
def show(img, figsize=(10, 10), title="Image", cmap='RGB'):
    figure=plt.figure(figsize=figsize)
    plt.imshow(img,cmap=cmap)
    plt.axis('off')
    plt.show()
```
![contoh citra](lena.jpg "contoh citra lena")

Mari kita cari tahu ukuran citra diatas dengan python.
```python
# fungsi untuk menampilkan ukuran file
def get_size(filename="lena.jpg"):
    stat = os.stat(filename)
    size=stat.st_size
    print(size/1024,'KB')
    return size
get_size()
```
```terminal
7.818359375 KB
```
Diketahui bahwa contoh citra memiliki ukuran 7,8KB

### Fungsi Encoding

Sekarang kita lakukan RLE compression pada contah citra lena dan kita save kedalam file.

```python
# read graysclae img
def RLE_encoding(img, bits=8,  binary=True):
    """
    img: Grayscale img.
    bits: what will be the maximum run length? 2^bits       
    """
    if binary:
        ret,img = cv.threshold(img,127,255,cv.THRESH_BINARY+cv.THRESH_OTSU)

    encoded = []
    shape=img.shape
    count = 0
    prev = None
    fimg = img.flatten()
    th=127
    for pixel in fimg:
        if binary:
            if pixel<th:
                pixel=0
            else:
                pixel=1
        if prev==None:
            prev = pixel
            count+=1
        else:
            if prev!=pixel:
                encoded.append((count, prev))
                prev=pixel
                count=1
            else:
                if count<(2**bits)-1:
                    count+=1
                else:
                    encoded.append((count, prev))
                    prev=pixel
                    count=1
    encoded.append((count, prev))
    return np.array(encoded)
# read image as grayscale
img = cv.imread('lena.jpg',0)
encoded = RLE_encoding(img, binary=False)
cv.imwrite('encode.tif',encoded)
```
Setelah itu kita cek ukuran file setelah di encoding
```python
get_size('encoded.tif')
```
```terminal
57.708984375 KB
```
Hasilnya citra grayscale tersebut justru memiliki ukuran yang lebih besar daripada sebelum dilakukan encoding. Mari kita coba kita encoding citra tersebut kedalam citra binary.
```python
encoded_bin = RLE_encoding(img,binary=True)
cv.imwrite('encoded_bin.tif',encoded_bin)
get_size('encoded_bin.tif')
```
```terminal
4.322265625 KB
```
Hasilnya ketika dilakukan RLE pada citra binary ukuran file hasil kompresi jauh lebih kecil daripada citra asli.
### Fungsi Decoding
Untuk menampilkan kembali citra yang sudah diencoding perlu dilakukan decoding terlebih dahulu.
```python
# Fungsi untuk melakukan decoding
def RLE_decode(encoded, shape):
    decoded=[]
    for rl in encoded:
        r,p = rl[0], rl[1]
        decoded.extend([p]*r)
    dimg = np.array(decoded).reshape(shape)
    return dimg
decode = RLE_decode(encoded,img.shape)
decode_bin = RLE_decode(encoded_bin,img.shape)
show(decode,cmap="gray")
show(decode_bin,cmap="gray")
```
![Lena grayscale](lena-grayscale.png)  ![Lena binary](lena-binary.png) 

## Simpulan
Percobaan yang dilakukan ini mungkin memakan banyak waktu dan hasil yang diberikan tidak mengejutkan. Tapi kesimpulan yang dapat diambil dari percobaan ini adalah RLE (Run Length Encoding) dapat bekerja dengan baik dengan ketika:
1. Citra yang akan diproses merupakan citra binary
2. Frekuensi piksel pada citra tidak terlalu besar.
3. Menyimpan dengan format TIFF.

**Referensi**<br>
&emsp;&emsp;&emsp;Gonzalez, Rafael C. 2001. *Digital Image Processing Second Edition*. Tom Robbins.<br>
&emsp;&emsp;&emsp;Eddine ALAA, Nour, dan Ismail Zine El Abidne. 2021. *Introduction to Image Processing with Python*. Cadi Ayyad University.<br>
&emsp;&emsp;&emsp;Viper, Quassarian. 2021. *Image Compression In Python: Run Length Encoding*. https://q-viper.github.io/2021/05/24/coding-run-length-encoding-in-python/ (diakses tanggal 15 Oktober 2021)<br>
&emsp;&emsp;&emsp;Hanifah, Riska. 2017. *Image Compression*. https://dosen.perbanas.id/image-compression/ (diakses tanggal 15 Oktober 2021)<br>