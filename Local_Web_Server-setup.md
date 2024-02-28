Setup Web Server at local
--

### Set up web server with python
```commandline
pythom -m http.server
```

### Set up web server with nodejs

#### launch with `http-server`
```commandline
npm install -g http-server
http-server
```

#### launch with `serve`
```commandline
npm install -g serve
serve ./dist
```

### Issues
#### cannot found `serve`
本来使用`npm -g`进行安装的可执行文件，比如`http-server`, `serve`是可以全局执行的，但有时安装后系统找不到，这时可以使用`npx`执行，比如
> npx需要在某一个node项目下执行

```commandline
npm install -g serve
npx serve ./dist -l 8101
```
