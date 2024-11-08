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
