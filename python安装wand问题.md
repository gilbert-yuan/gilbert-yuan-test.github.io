## python安装wand问题

python 中安装wand 的时候总有会报错的情况所以特别说明

```shell
pip install wand # 安装wand 没有问题。只是在使用的时候出了问题。

brew uninstall --force imagemagick
brew install imagemagick@6 
echo 'export PATH="/usr/local/opt/imagemagick@6/bin:$PATH"' >> ~/.bash_profile 
brew link imagemagick@6 --force
```
