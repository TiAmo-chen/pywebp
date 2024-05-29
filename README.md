# WebP Python bindings

[![Build status](https://img.shields.io/endpoint.svg?url=https%3A%2F%2Factions-badge.atrox.dev%2Fanibali%2Fpywebp%2Fbadge&label=build&logo=none)](https://actions-badge.atrox.dev/anibali/pywebp/goto)
[![License](https://img.shields.io/github/license/anibali/pywebp.svg)](https://github.com/anibali/pywebp/blob/master/LICENSE)
[![PyPI](https://img.shields.io/pypi/v/webp)](https://pypi.org/project/webp/)
[![GitHub](https://img.shields.io/github/stars/anibali/pywebp?style=social)](https://github.com/anibali/pywebp)

## 安装

```sh
pip install webp
```

在 Windows 系统上，您可能会在安装过程中遇到以下错误:

```
conans.errors.ConanException: 'settings.compiler' value not defined
```

这意味着您需要安装一个 C 编译器，并配置 Conan 以便它知道使用哪个编译器。详见 https://github.com/anibali/pywebp/issues/20 for more details.

### 依赖

* Python 3.8+

## 使用

```python
import webp
```

### 示例 API

```python
# 保存图像
webp.save_image(img, 'image.webp', quality=80)

# 加载图像
img = webp.load_image('image.webp', 'RGBA')

# 保存动画
webp.save_images(imgs, 'anim.webp', fps=10, lossless=True)

# 加载动画
imgs = webp.load_images('anim.webp', 'RGB', fps=10)
```

如果您更喜欢使用numpy数组，请使用函数`imwrite`、`imread`、`mimwrite`和`mimread`。


### 高级 API

```python
# 将PIL图像编码为内存中的WebP，并添加编码器提示
pic = webp.WebPPicture.from_pil(img)
config = WebPConfig.new(preset=webp.WebPPreset.PHOTO, quality=70)
buf = pic.encode(config).buffer()
# 读取WebP文件并将其解码为BGR numpy数组
with open('image.webp', 'rb') as f:
  webp_data = webp.WebPData.from_buffer(f.read())
  arr = webp_data.decode(color_mode=WebPColorMode.BGR)
# 保存动画
enc = webp.WebPAnimEncoder.new(width, height)
timestamp_ms = 0
for img in imgs:
  pic = webp.WebPPicture.from_pil(img)
  enc.encode_frame(pic, timestamp_ms)
  timestamp_ms += 250
anim_data = enc.assemble(timestamp_ms)
with open('anim.webp', 'wb') as f:
  f.write(anim_data.buffer())
# 加载动画
with open('anim.webp', 'rb') as f:
  webp_data = webp.WebPData.from_buffer(f.read())
  dec = webp.WebPAnimDecoder.new(webp_data)
  for arr, timestamp_ms in dec.frames():
    # `arr` 包含了解码后的帧像素
    # `timestamp_ms` 包含了帧的结束时间
    pass
```


## 功能特点

* 图像编码/解码
* 动画编码/解码
* 自动内存管理
* 用于操作`PIL.Image`对象的简单API

### 未实现的功能

* 在YUV色彩模式下编码/解码静态图像
* 高级复用/解复用（色彩配置文件等）
* 暴露所有有用的字段

## 开发者笔记

### 设置

1. Install `mamba` and `conda-lock`. The easiest way to do this is by installing
   [Mambaforge](https://github.com/conda-forge/miniforge#mambaforge) and then
   running `mamba install conda-lock`.
   安装`mamba`和`conda-lock`。最简单的方法是安装Mambaforge](https://github.com/conda-forge/miniforge#mambaforge)，然后运行`mamba install conda-lock`。
3. 创建并激活Conda环境:
   ```console
   $ conda-lock install -n webp
   $ mamba activate webp
   ```
4. 安装PyPI依赖项:
   ```console
   $ pdm install -G:all
   ```

### 运行测试

```console
$ pytest tests/
```

### Cutting a new release

1. Ensure that tests are passing and everything is ready for release.
2. Create and push a Git tag:
   ```console
   $ git tag v0.1.6
   $ git push --tags
   ```
3. Download the artifacts from GitHub Actions, which will include the source distribution tarball and binary wheels.
4. Create a new release on GitHub from the tagged commit and upload the packages as attachments to the release.
5. Also upload the packages to PyPI using Twine:
   ```console
   $ twine upload webp-*.tar.gz webp-*.whl
   ```
6. Bump the version number in `pyproject.toml` and create a commit, signalling the start of development on the next version.

These files should also be added to a GitHub release.

## Known issues

* An animation where all frames are identical will "collapse" in on itself,
  resulting in a single frame. Unfortunately, WebP seems to discard timestamp
  information in this case, which breaks `webp.load_images` when the FPS
  is specified.
* There are currently no 32-bit binaries of libwebp uploaded to Conan Center. If you are running
  32-bit Python, libwebp will be built from source.
