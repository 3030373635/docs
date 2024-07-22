## 一、webpack安装

```html
npm npm i webpack webpack-cli -g
```

## 二、3.x安装

### 2.1 cli安装

```html
#安装
npm i -g @vue/cli
#初始化
vue create project
#可视化初始化
vue ui
```

## 三、2.x安装

### 2.1 cli安装

```html
#安装
npm i vue-cli -g
#初始化
vue init webpack project
```

## 四、2.x/3.x共存安装

### 4.1 安装3.x

### 4.3 安装桥接

```html
桥接共存,安装cli3.x同时让cli2.x不被覆盖
#安装桥接
npm i -g @vue/cli-init
#cli2.x的初始化命令依然是：
vue init webpack project