<!--
title:   異なるサイズ（A4・A3）と角度を持ったPDFをA3サイズのPDFに2アップしよう
tags:    PDF,PyPDF2,Python
id:      00bbde6ea8d52174d96c
private: false
-->
# やりたいこと
2つの異なるページサイズ（A3とA4）を持ち、ページの向きも縦横混在するPDFを処理して、A4用紙が2アップされたA3用紙を作成します。

処理対象であるPDFの各ページのサイズと向きによって、処理を変えます。
処理は以下の通りです。

- A4タテの場合：A4サイズで貼り付ける。
- A4ヨコの場合：反時計回りに90度回転し、A4サイズで貼り付ける。
- A3タテの場合：反時計回りに90度回転し、追加する。
- A3ヨコの場合：無処理で追加する。

作成していた基本A4サイズの資料をA3サイズにとりまとめる必要があったので、下記コードを作成しました。

## 苦労した点
A4用紙を回転させつつ、A3用紙に貼り付ける方法が難しかったです。

`.rotateCounterClockwise(90)` で回転させたA4用紙を、そのまま`.mergeTranslatedPage()`でA3用紙に貼り付けられたら簡単だったのですが、回転が反映されませんでした。

最終的には`.mergeRotatedTranslatedPage()`の引数に回転角度と位置を指定することで上手くいきました。

# プログラム

```python:qiita.python
import PyPDF2
from PyPDF2 import PdfFileReader, PdfFileWriter
from pathlib import Path

input_file = "input.pdf"
ouput_file = input_file.replace(".pdf", "_2up.pdf")

# 入力PDFファイルを開く
reader = PdfFileReader(input_file)
# 出力PDFファイルを開く
output_pdf = PdfFileWriter()

# ページサイズを取得し、ページの向きを把握する（A3用紙なら回転させる）。
def organaize_page_rotate(page):
    page_width, page_height = page.mediaBox.getWidth(), page.mediaBox.getHeight()
    rotated_angle = 0
    # A3用紙の場合
    if (page_width > 1000) or  (page_height > 1000):
        page_size = "A3"
        if page_width < page_height:
            page = page.rotateCounterClockwise(90)
            rotated_angle = 90
    # A4用紙の場合
    else:
        page_size = "A4"
        if page_width > page_height:
            rotated_angle = 90
    return page, page_size, rotated_angle

# A3用紙にA4用紙を貼り付ける。左右の位置と、回転の有無で処理を変える。
def rotated_and_add_page(a3_page, page, tx, rotated_angle):
    # 回転が必要ない場合
    if rotated_angle == 0:
        a3_page.mergeTranslatedPage(page, tx=tx, ty=0)
    # 左側で回転が必要な場合
    elif tx == 0:
        a3_page.mergeRotatedTranslatedPage(page, tx=595.32/2, ty=595.32/2, rotation=rotated_angle)
    # 右側で回転が必要な場合
    else:
        a3_page.mergeRotatedTranslatedPage(page, tx=tx, ty=595.32, rotation=rotated_angle)

# 各ページを処理する
old_right_page_num = -1
for page_num in range(reader.getNumPages()):
    # A3用紙の右側へ既に追加していればスキップする
    if page_num == old_right_page_num:
        continue
    page = reader.getPage(page_num)
    page, page_size, rotated_angle = organaize_page_rotate(page)

    # A3用紙の場合
    if page_size == "A3":
        output_pdf.addPage(page)

    # A4用紙の場合
    else:
        # A3用紙に2アップでページを貼り付ける
        a3_page = PyPDF2.pdf.PageObject.createBlankPage(None, 1190.52, 841.92)
        rotated_and_add_page(a3_page, page, tx=0, rotated_angle=rotated_angle)

        right_page_num = page_num + 1
        if right_page_num < reader.getNumPages():
            print(right_page_num)
            right_page = reader.getPage(right_page_num)
            right_page, page_size, rotated_angle = organaize_page_rotate(right_page)

            # 右側に貼り付けたい次のページがA3用紙の場合は処理をスキップする
            if page_size == "A3":
                pass
            # 右側に貼り付けたい次のページがA4用紙ならそのまま張り付ける
            else:
                rotated_and_add_page(a3_page, right_page, tx=595.32, rotated_angle=rotated_angle)
                old_right_page_num = right_page_num

        output_pdf.addPage(a3_page)

# 出力PDFファイルに保存する
with open(ouput_file, 'wb') as f:
    output_pdf.write(f)

print("Completed")
```