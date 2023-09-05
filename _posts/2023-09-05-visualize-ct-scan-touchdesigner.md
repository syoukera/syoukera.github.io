---
layout: post
title: CTスキャンデータをTouchDesingerで描画する
subtitle: Visualise CT scan data using TouchDesigner
author: syoukera
categories: touchdesigner
banner:
  video: https://github.com/syoukera/syoukera.github.io/assets/39158849/4d146767-4e51-4ea8-8797-36b83a00bc42
  loop: true
  volume: 0.8
  start_at: 8.5
  image: https://bit.ly/3xTmdUP
  opacity: 0.618
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  subheading_style: "color: gold"
tags: touchdesigner pydicom
sidebar: []
---
CTスキャンデータはdcmという拡張子であらわされており，病院でCTやMRIのスキャンを行った際に受け取るDVDを開けるとこのデータが保存されています．
dicomデータはpydicomというpythonパッケージで読み込むことで3次元のndarrayとして利用することができます．このデータをTouchDesigner上でTOPに変換することによってCTスキャンデータなどの医療用データの可視化が可能です．
この記事では，pydicomを用いて読み込んだCTスキャンのデータをTouchDesigner上で描画します．

# 環境構築

## pydicomのインストール

anaconda環境でpycicomをインストールするためには，以下のコマンドを実行します．

```bash
conda install -c conda-forge pydicom
```
## pydicomをTouchDesignerで使用できるようにする

Edit >> PreferencesからPythonのパスを指定します．

![68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f3236373138332f35396161386161322d636665392d323061622d613531662d3232663462633237623232322e706e67](https://github.com/syoukera/syoukera.github.io/assets/39158849/a98a7059-1571-4b77-9bb2-a8e77aac3b44)

私の環境では以下のパスを登録しました．これでTouchDesignerで実行するPythonでpydicomを使用できるようになります．

```
C:/Users/[ユーザー名]/Miniconda3/envs/dicom/Lib/site-packages
```

# TouchDesignerでの実装

あとはTouchDesigner上でPythonスクリプトを動かしていきます．
今回のプロジェクトでは以下のように，①dicomという名前のText DATでDCMクラスを作成し，②作成したDCMクラスをScript CHOPで呼び出してCHOPに書き出します．最後にCHOP to TOPを使用して変換することによって毎フレームの断面図を取得しています．

![68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f3236373138332f30616137383865332d316333632d343866372d336439322d6336323633373765333336302e706e67](https://github.com/syoukera/syoukera.github.io/assets/39158849/852243d7-4f87-43cd-81b7-dee257671a54)

## dicomクラスを作成

Base COMPを作成して，右クリックからCustomize ComponentをクリックしてExtensionを追加することで，TouchDesigner上で定義したPythonクラスを利用できるようになります．（詳しくは[比嘉 了さんの解説記事](http://satoruhiga.com/post/extending-touchdesigner/)の「COMP Extensionを使う」を参考にしてください．）ここでBase COMPの中に作成されたText DATに以下を記述して，dicomというクラスを作成しています．

`root_dir = 'PATH_TO_DCM_FOLDER'`には自分のdcmファイルへのパスに変更してください．

dicomクラスでは初期化の際に`root_dir`で指定したパスに存在しているdcmファイルを読み込んで三次元のndarrayである`self.D_array`を作成しています．このクラスを使用するときには`self.D_array`を直接参照することによって任意の断面を取得します．


```python
from TDStoreTools import StorageManager
import TDFunctions as TDF

import pydicom
import os
import numpy as np
import math

root_dir = 'PATH_TO_DCM_FOLDER'

class dicom:
	"""
	dicom description
	"""
	def __init__(self, ownerComp):
		
		# The component to which this extension is attached
		self.ownerComp = ownerComp
		
		# list for dcm data 
		dcms = []
		# walck through files in root_dir
		for d, s, fl in os.walk(root_dir):
			# each file in files
			for fn in fl:
				# only append DCM files
				if ".dcm" in fn.lower():
					# append path of DCM files to list
					dcms.append(os.path.join(d, fn))

		# to know size of a dcm image, read dicom data usign pydicom package
		ref_dicom = pydicom.read_file(dcms[0])
		# save number of rows
		self.rows = ref_dicom.Rows
		# save number of columns
		self.columns = ref_dicom.Columns
		# save number of dcm files
		self.Num_dcm = len(dcms)

		# initialize ndarray for save 3-dimentional dcm data
		self.D_array = np.zeros((self.rows, self.columns, len(dcms)), dtype=ref_dicom.pixel_array.dtype)
		# save each dcm file to 3-dimentional ndarray
		for dcm in dcms:
			# load a dcm file using pydicom
			d = pydicom.read_file(dcm)
			# assign data of a dcm file to 3-dimentional ndarray
			self.D_array[:, :, dcms.index(dcm)] = d.pixel_array
```

## dicomクラスからndarrayをTOPに書き出す

Script CHOPを作成して，紐づいているScript Callbacksをから上で定義したdicomクラスを使用します．
depthとして定義した値を変更することによって，スライスする深さを変更することが可能です．
また，`scriptOp.copyNumpyArray(op_dicom.D_array[:, index, :].astype(np.float32))`の`index`の配置を変更することにより任意のxyz断面に変更することが可能です．

```python
import numpy as np
import math

op_dicom = op('..')

def onCook(scriptOp):

	# depth is a value range from [0, 1]
	depth = 0.5

	# normalize depth by number of dcm
	index = math.floor(depth*(op_dicom.Num_dcm-1))

    # clear channels at previous cook
	scriptOp.clear()
    
    # slice 3d array at y=index
	scriptOp.copyNumpyArray(op_dicom.D_array[:, index, :].astype(np.float32))

	return
```

## 3断面を同時に描画する

depthをnoiseで指定し，xyzにスライスした３断面を同時に表示すると以下のようにCTスキャンデータをTouchDesigner上で可視化することができました．

![2023-09-05_mov1](https://github.com/syoukera/syoukera.github.io/assets/39158849/268247fa-188b-46b5-9377-d445ff8205a0)


