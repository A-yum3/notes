# 私的学習メモ

## 前提条件

- Obsidianを利用する
- すべてMarkdown形式で書く

## 使い方

- Zettelkastenをベースに構築する
- LYT Frameworkを利用して999_Notes/下にEvergreen Noteをストックしていく
- MOCを作成し、作成したMOCに対して100_HOMEからたどり着けるようにLinking

## ディレクトリ解説

- 000_Private
  - 非公開情報を含むディレクトリ。**Git上ではignoreに指定し、管理しない**
- 100_HOME
  - Home Note。原則ではこのノートから全てのノートにたどり着く事ができる
- 200_Bin
  - 人に関する情報。自己に関係するものを多く含む
- 300_Sources
  - 本や動画から学習した内容をノートにまとめている。リファクタリング推奨だが、ノートを小さく切って999_Notesに保存している場合はその限りではない
- 997_outputs
  - はてなブログやSlideのアウトプット用ディレクトリ。SlideにはMarpを利用し、テンプレートには中にあるcssを用いること
- 998_images
  - Obsidian Publishを使っていた時の画像保存用ディレクトリ。昔の名残で残しているため非推奨
- 999_Notes
  - Zettelkasten方式にて貯蔵されたNote郡。ここにあるNoteはLYT Frameworkによってすべて他のなにかのNoteと結び付けられていることが保証される必要がある