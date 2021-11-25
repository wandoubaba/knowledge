# npm切换源

---

## 查看源来的镜像地址

```bash
npm get registry
```

## 设置成淘宝源

```bash
npm config set registry http://registry.npm.taobao.org
```

## 设置成初始的源

```bash
npm config set registry https://registry.npmjs.org/
```

## 将npm替换成cnpm

```bash
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

## 全局安装yarn

```bash
cnpm install yarn -g
```
