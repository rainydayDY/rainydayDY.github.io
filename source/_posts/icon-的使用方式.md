---
title: icon 的使用方式
date: 2019-01-25 11:11:26
tags:
- svg
categories: 前端
---
> 使用svg-sprite-loader和svgo处理icon
<p hidden><!--more--></p>

### 为什么不推荐使用iconfont

1. 引入的字体文件如果下载到本地，每次新增或者修改icon，我们都需要重新下载一遍，如果通过外链的方式，每次都要改一遍地址，不方便
2. 不同的项目的UI规范是不一样的，每次项目中的icon在我们看来，没什么区别，实际上线条的宽度圆角什么的是有差别的

### 为什么使用svg

1. 支持多色图标了，不再受单色限制。
2. 支持像字体那样通过font-size,color来调整样式。
3. 支持 ie9+
4. 可利用CSS实现动画。
5. 减少HTTP请求。
6. 矢量，缩放不失真
7. 可以很精细的控制SVG图标的每一部分

### svg-sprite-loader
将svg文件打包成一个雪碧图，并且在页面加载的时候自动注入到document.body下，每个需要使用的地方通过use复制模板

use：在SVG文档内取得目标节点，并在别的地方复制它们



### 使用方式

1. 下载svg格式的icon（如果同一icon在项目中有多个颜色的显示，下载完成后，将fill属性去掉，不然自己写的颜色的是无效的）；
2. 统一放置在目录assets/icon下
3. 安装依赖： svg-sprite-loader
4. 配置webpack

```
    {
        test: /\.svg$/,
        loader: 'svg-sprite-loader',
        include: resolve('src/assets/svg'),
        options: {
            symbolId: 'icon-[name]'
        }
    },
    {
        test: /\.(woff2?|eot|ttf|otf|svg)(\?.*)?$/,
            loader: 'url-loader',
            exclude: resolve('src/assets/svg'),
            options: {
                limit: 10240,
                name: 'static/fonts/[name].[hash:8].[ext]'
            }
    },

```
5. index.jsx中对icon进行自动导入

```javascript
    // utils/svgIcon.js
    const requireAll = requireContext => requireContext.keys().map(requireContext);
    const req = require.context('../assets/svg', false, /\.svg$/);
    requireAll(req);

```
6. SvgIcon组件
```javascript

import React from 'react';
import PropTypes from 'prop-types';
import './index.less';

export default class SvgIcon extends React.Component {

    static propTypes = {
        iconClass: PropTypes.string.isRequired,
        className: PropTypes.string,
    }

    render() {
        const { iconClass, className } = this.props;
        const svgClass = className ? 'svg-icon ' + className : 'svg-icon';
        return (
            <svg className={svgClass} aria-hidden="true">
                <use xlinkHref={`#icon-${iconClass}`}></use>
            </svg>
        );
    }
}
```
7. 引入SvgIcon
```javascript
    <SvgIcon iconClass="svg的名称" className=""/>
```

### svg压缩工具——————svgo

我们从UI那里拿到的svg带有很多冗余信息，比方说下面这个：

```html
<?xml version="1.0" encoding="UTF-8"?>
<svg width="54px" height="45px" viewBox="0 0 54 45" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
    <!-- Generator: Sketch 52.2 (67145) - http://www.bohemiancoding.com/sketch -->
    <title>换</title>
    <desc>Created with Sketch.</desc>
    <g id="接单发货" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
        <g id="采购单详情-修改采购单" transform="translate(-60.000000, -1554.000000)">
            <g id="换" transform="translate(60.000000, 1554.000000)">
                <path d="M3,0 L51,0 C52.6568542,-3.04359188e-16 54,1.34314575 54,3 L54,42 C54,43.6568542 52.6568542,45 51,45 L0,45 L0,3 C-2.02906125e-16,1.34314575 1.34314575,3.04359188e-16 3,0 Z" id="Rectangle-7-Copy-4" fill="#ECF8F1"></path>
                <text font-family="PingFangSC-Regular, PingFang SC" font-size="30" font-weight="normal" line-spacing="36" fill="#43C177">
                    <tspan x="12" y="33">换</tspan>
                </text>
            </g>
        </g>
    </g>
</svg>
```
里面包含了不必要的xml信息、编辑器信息、title、描述信息等，这些信息并不会影响svg的展示

经过压缩之后：

```html
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 54 45"><g fill="none" fill-rule="evenodd"><path fill="#ECF8F1" d="M3 0h48a3 3 0 013 3v39a3 3 0 01-3 3H0V3a3 3 0 013-3z"/><text fill="#43C177" font-family="PingFangSC-Regular, PingFang SC" font-size="30"><tspan x="12" y="33">换</tspan></text></g></svg>
```

压缩脚本：
```javascript
// 压缩svg，去除冗余信息
const fs = require('fs');
const path = require('path');
const SVGO = require('svgo');

const fileDir = path.join(__dirname, 'src/assets/svg');


const svgo = new SVGO({
    plugins: [{
        cleanupAttrs: true, // 从换行符，尾随空格和重复空格中清除属性
    }, {
        removeDoctype: true, // 删除文档类型声明
    }, {
        removeXMLProcInst: true, // 删除XML处理指令
    }, {
        removeComments: true, // 删除评论
    }, {
        removeMetadata: true, 
    }, {
        removeTitle: true,
    }, {
        removeDesc: true,
    }, {
        removeUselessDefs: true, // 删除没有ID的<defs>元素
    }, {
        removeEditorsNSData: true, // 删除编辑器的命名空间，元素和属性
    }, {
        removeEmptyAttrs: true,
    }, {
        removeHiddenElems: true,
    }, {
        removeEmptyText: true,
    }, {
        removeEmptyContainers: true,
    }, {
        removeViewBox: false,
    }, {
        cleanupEnableBackground: true,
    }, {
        convertStyleToAttrs: true, // 将样式转换为属性
    }, {
        convertColors: true, // 将rgba转换为16进制颜色
    }, {
        convertPathData: true, // 将Path数据转换为相对或绝对数据（以较短者为准），将一个段转换为另一段，修剪无用的定界符，智能舍入等
    }, {
        convertTransform: true, // 将多个转换折叠为一个，将矩阵转换为短别名，等等
    }, {
        removeUnknownsAndDefaults: true, // 删除未知元素的内容和属性，使用默认值删除属性
    }, {
        removeNonInheritableGroupAttrs: true, // 删除不可继承组的“表示”属性
    }, {
        removeUselessStrokeAndFill: true, // 删除无用的stroke和fill属性
    }, {
        removeUnusedNS: true, // 删除未使用的名称空间声明
    }, {
        cleanupIDs: true, // 删除未使用的并最小化使用的ID
    }, {
        cleanupNumericValues: true, // 将数值四舍五入到固定精度，删除默认的px单位
    }, {
        moveElemsAttrsToGroup: true, // 将元素的属性移到其所属的组中
    }, {
        moveGroupAttrsToElems: true, // 将一些组属性移动到所包含的元素
    }, {
        collapseGroups: true, // 折叠无用的组
    }, {
        removeRasterImages: false, // 删除光栅图像
    }, {
        mergePaths: true, // 将多个路径合并为一个
    }, {
        convertShapeToPath: true, // 将一些基本形状转换为<path>
    }, {
        sortAttrs: true, // 排序元素属性以实现史诗般的可读性
    }, {
        removeDimensions: true, // 删除宽度/高度并添加viewBox（如果缺少）（与removeViewBox相反，请先禁用它）
    }],
});

fs.readdir(fileDir, (err, files) => {
    if (!err) {
        files.forEach((fileName) => {
            // 对大于1kb的svg文件采取压缩
            if (fileName.endsWith('.svg')) {
                const filePath = `${fileDir}/${fileName}`;
                fs.readFile(filePath, { encoding: 'utf-8' }, (err, data) => {
                    if (!err) {
                        svgo.optimize(data, {}).then((result) => {
                            if (result.data) {
                                fs.writeFileSync(filePath, result.data);
                            }
                        });
                    }
                });
                // fs.stat(filePath, (err, stats) => {
                //     if (!err) {
                //         if (stats.size > 1000) {
                //             console.log(fileName, stats.size);
                            
                //         }
                //     }
                // });
            }
        });
    }
});


```

参考文章：
1. [手摸手，带你优雅的使用 icon](https://juejin.im/post/59bb864b5188257e7a427c09#heading-6)



