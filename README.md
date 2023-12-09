# Blog Operation

## Install

```
sudo apt-get install hugo
git clone git@github.com:wonderflow/wonderflow.github.io.git
git submodule update --init --recursive
```

## New Post
```
hugo new posts/<YYYY>-<MM>-<DD>-<title>.md
```

### Images

```
cp myimage.png ./static/images/<folder>/
```

Link it by `![undefined](/images/<folder>/myimage.jpg)`

## Preview

```
hugo server -D
```


## Build

```
hugo -e production
```

## Release

```
git add .
git commit -m "my new post"
git push origin master
```