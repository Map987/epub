# epub
https://colab.research.google.com/gist/Map97png/b15bbaa3a0d05163db52c3c837e4e6db/untitled11.ipynb

```python
!git clone https://github.com/Map987/linovelib2epub --depth=1

%cd /content/linovelib2epub

!python -m venv venv
!source /venv/bin/activate.fish # 我用的 fish
!pip install -r requirement-dev.txt # 他们的 requirement.txt 很久没更新过了，直接用 dev
!pip install -e . #https://blog.sh1mar.in/post/linux/scrape-bili-novel/

!wget https://storage.googleapis.com/chrome-for-testing-public/129.0.6668.58/linux64/chrome-linux64.zip -N -P /tmp
!unzip -o /tmp/chrome-linux64.zip -d /tmp/
!rm -rf /tmp/chrome-linux64.zip
!cp -rf /tmp/chrome-linux64 /usr/local/bin/chrome-linux64
!rm -rf /tmp/chrome-linux64
!sudo ln -s /usr/local/bin/chrome-linux64/chrome /usr/local/bin/google-chrome
print() #https://qiita.com/watsony/items/f1b04f99599247b6cce6或者https://github.com/jpjacobpadilla/Google-Colab-Selenium
#src/linovelib2epub/spider/linovelib_spider.py文件https://github.com/lightnovel-center/linovelib2epub/blob/f648dc7af3f50bea4df2be8c8075bd5758f31387/src/linovelib2epub/spider/linovelib_spider.py#L324处添加co.set_argument("--headless")，以及 co.set_argument("--no-sandbox")
#https://github.com/lightnovel-center/linovelib2epub/blob/main/src/linovelib2epub/spider/linovelib_spider.py
```

```python
from linovelib2epub import Linovelib2Epub, TargetSite

import nest_asyncio
nest_asyncio.apply() #避免 RuntimeError: asyncio.run() cannot be called from a running event loop.
if __name__ == "__main__":
    linovelib_epub = Linovelib2Epub(
        book_id=3114,
        divide_volume=True,
        target_site=TargetSite.LINOVELIB_PC,
        browser_path="/usr/local/bin/chrome-linux64",
        headless=True
    )#src/linovelib2epub/linovel.py 中 class Linovelib2Epub: 类
     #https://github.com/lightnovel-center/linovelib2epub/blob/main/src/linovelib2epub/linovel.py
    linovelib_epub.run()#为么有run这个方法
```

## epub 添加图片，插图到前面几页，首页

```
import zipfile
import os
import tempfile
import shutil

def find_oebps_folder(first_dir):
   for root, dirs, files in os.walk(first_dir):
        for file in files:
            print(file)
            if file.lower().endswith('.opf'):
                return root
   return None

def find_opf_file(directory):
    print(directory)
    for root, dirs, files in os.walk(directory):
        for file in files:
            print(file)
            if file.endswith('.opf'):
                return os.path.join(root, file)
        return None

def find_image_folder(directory):
    for root, dirs, files in os.walk(directory):
        for file in files:
            if file.lower().endswith(('.jpg', '.jpeg', '.png', '.gif')):
                return root
    return None

def find_xhtml_folder(directory):
    for root, dirs, files in os.walk(directory):
        for file in files:
            if file.endswith('.xhtml'):
                return root
    return None

import os

def print_file_tree(directory, prefix=''):
    if not os.path.isdir(directory):
        print("提供的路径不是一个文件夹")
        return

    files = os.listdir(directory)
    for i, file in enumerate(files):
        print(prefix + '├── ' + file)
        path = os.path.join(directory, file)
        if os.path.isdir(path):
            if i == len(files) - 1:
                # 最后一个文件
                new_prefix = prefix + '    '
            else:
                new_prefix = prefix + '|   '
            print_file_tree(path, new_prefix)

# 假设temp_dir是之前解压EPUB文件到的临时文件夹的路径


import os
import tempfile
import shutil

def modify_epub(epub_path):
    # 创建一个临时文件夹
    temp_dir = tempfile.mkdtemp()
    
    # 解压EPUB文件到临时文件夹
    with zipfile.ZipFile(epub_path, 'r') as epub_zip:
        epub_zip.extractall(temp_dir)
    print_file_tree(temp_dir)
    # 获取OEBPS文件夹路径
   # oebps_folder = os.path.join(temp_dir, 'OEBPS')
    
    oebps_folder = find_oebps_folder(temp_dir)
    if not oebps_folder:
        print("没有找到oebps文件夹。")
        return

    # 查找opf文件
    content_opf_path = find_opf_file(oebps_folder)
    if not content_opf_path:
        print("没有找到opf文件。")
        return
    
    # 查找图片文件夹
    images_folder = find_image_folder(oebps_folder)
    if not images_folder:
        print("没有找到图片文件夹。")
        return
    
    # 查找XHTML文件夹
    xhtml_folder = find_xhtml_folder(oebps_folder)
    if not xhtml_folder:
        print("没有找到XHTML文件夹。")
        return
    
    # 获取Images文件夹中的所有图片
    images = [f for f in os.listdir(images_folder) if f.lower().endswith(('.jpg', '.jpeg', '.png', '.gif'))]
    
    # 创建一个新的XHTML文件内容
    xhtml_content = f"""<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <title>Image Gallery</title>
</head>
<body>"""
    
    # 添加图片到XHTML内容
    for image in images:
        xhtml_content += f'<div class="duokan-image-single illus"><img alt="{os.path.splitext(image)[0]}" src="{os.path.relpath(os.path.join(images_folder, image), xhtml_folder)}" /></div>\n'
    
    xhtml_content += """</body>
</html>"""
    
    # 新的XHTML文件路径
    new_xhtml_path = os.path.join(xhtml_folder, 'aaa.xhtml')
    
    # 写入新的XHTML文件
    with open(new_xhtml_path, 'w', encoding='utf-8') as xhtml_file:
        xhtml_file.write(xhtml_content)
    
    # 读取并更新content.opf文件
    with open(content_opf_path, 'r', encoding='utf-8') as opf_file:
        content_opf = opf_file.read()
    
    # 更新manifest
    manifest_start = content_opf.find('<manifest>')
    manifest_end = content_opf.find('</manifest>')
    manifest = content_opf[manifest_start:manifest_end + len('</manifest>')]
    new_item = f'<item id="aaa" href="{os.path.relpath(new_xhtml_path, oebps_folder)}" media-type="application/xhtml+xml"/>'
    updated_manifest = manifest.replace('</manifest>', new_item + '</manifest>')
    
    # 更新spine
    spine_start = content_opf.find('<spine toc="ncx">')
    spine_end = content_opf.find('</spine>')
    spine = content_opf[spine_start:spine_end + len('</spine>')]
    new_itemref = '<itemref idref="aaa" />'
    updated_spine = spine.replace('<spine toc="ncx">', f'<spine toc="ncx">{new_itemref}')
    
    # 更新content.opf文件
    updated_content_opf = content_opf.replace(manifest, updated_manifest)
    updated_content_opf = updated_content_opf.replace(spine, updated_spine)
    
    # 写入更新后的content.opf文件
    with open(content_opf_path, 'w', encoding='utf-8') as opf_file:
        opf_file.write(updated_content_opf)
    
    # 重新压缩修改后的文件回EPUB
    with zipfile.ZipFile(epub_path, 'w', zipfile.ZIP_DEFLATED) as epub_zip:
        for root, dirs, files in os.walk(temp_dir):
            for file in files:
                file_path = os.path.join(root, file)
                epub_zip.write(file_path, file_path[len(temp_dir):])
    print("ok")
    # 清理临时文件夹
    shutil.rmtree(temp_dir)

# 使用示例
epub_path = '/content/01和班.epub'
modify_epub(epub_path)
```

