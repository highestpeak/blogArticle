# 执行命令

1. `vue create <project-name>` 创建 VUE 项目
2. `vue add electron-builder` 安装 electron 项目

# `vue.config.js`

``` javascript
const path = require('path')

function resolve(dir) {
    return path.join(__dirname, dir)
}

module.exports = {
    publicPath: './',
    devServer: {
        // can be overwritten by process.env.HOST
        host: 'localhost',
        port: 8080
    },
    // 重新设定 主进程和 渲染进程的目录
    // 对于vue的渲染进程的每个vue文件的引入意义更大一些
    chainWebpack: config => {
        config.resolve.alias
            .set('@', resolve('src/renderprocess'))
            .set('src', resolve('src/renderprocess'))
            .set('common', resolve('src/renderprocess/common'))
            .set('components', resolve('src/renderprocess/components'))
            .set('main', resolve('src/mainprocess'))
    },
    // 打包时的配置
    pluginOptions: {
        electronBuilder: {
            builderOptions: {
                win: {
                    icon: './public/logo.png'
                },
                mac: {
                    icon: './public/logo.png'
                }
            }
        }
    }
}
```

# 更改项目结构如下

较为重要的目录结构如下

``` shell
- node_modules
- public
- src
  - mainprocess
  - renderprocess
    - assets
    - compontents
    - router
    - store
    - views
  App.vue
  background.js
  main.js
.editorconfig
.eslintrc.js
package.json
vue.config.js
```

