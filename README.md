# Unit-Test-for-Mini-Program

## 文件配置
下载好`miniprogram-simulate`
```js
npm i —save-dev miniprogram-simulate
```
以及`jest`
```js
npm install —save-dev jest
```

在根目录下创建`jest.config.js`
配置jest
```js
{
    // jest 是直接在 nodejs 环境进行测试，使用 jsdom 进行 dom 环境的模拟。在使用时需要将 jest 的 `testEnvironment` 配置为 `jsdom`。
    // jest 内置 jsdom，所以不需要额外引入。
    “testEnvironment”: “jsdom”,
    // 配置 jest-snapshot-plugin 从而在使用 jest 的 snapshot 功能时获得更加适合肉眼阅读的结构
    “snapshotSerializers”: [“miniprogram-simulate/jest-snapshot-plugin”]
}
```

在package.json中
```js
scripts: {
  “test”: “jest”
}
```

## 创建测试文件
在要测试的文件的同级目录中创建
`filename.test.js`

### 引入测试工具
```js
const simulate = require(‘miniprogram-simulate’)
```

### 引入路径
```js
const path = require('path')
```

### 创建单元测试
[官方使用指南文档](https://github.com/wechat-miniprogram/miniprogram-simulate/blob/master/docs/tutorial.md#%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97)

**示例**
```js
describe('main description', () => {
  let id
  beforeAll(() => {
    id = simulate.load(path.join(__dirname, './comp')) //加载自定义组件，返回组件的id
  })
  test('basic displaysheet', () => {
    const container = simulate.render(id) //使用id渲染自定义组件，返回组件封装实例
    const parent = document.createElement('parent-wrapper') //创建容器节点
    container.attach(parent) //将组件插入到容器节点中，会触发attached生命周期
    expect(container.toJSON()).toMatchSnapshot() //判断组件渲染结果
    
    comp.detach() //将组件从容器节点中移除，会触发detached生命周期
  })
  //测试点击事件
  test('close container', async() => {
    const container = simulate.render(id)
    container.attach(document.createElement('parent-wrapper'))
    const closeBtn = container.querySelector('CLOSE_BUTTON_CLASS_NAME')
    closeButton.dispatchEvent('tap')
    await simulate.sleep(10)
    expect(container.data).toEqual({show: false})
    })
})
```

**避坑指南**
* `dispatchEvent`是触发事件，需要在组件中执行，由于目前的源码是异步执行，涉及到`dispatchEvent`事件的测试，需要在function中前加async，在`dispatchEvent`事件后加上`await simulate.sleep(10)`才能保证`expect`成功
* `dispatchEvent`对应的事件名要去掉`bind`，如`bindtap`事件触发时就是`dispatchEvent(‘tap’)` ; `bindgetuserinfo`触发时就是`getuserinfo`

**关于测试组件中某个函数是否执行**
```js
const spy = jest.spyOn(container.instance, 'funcName')
//执行一些触发func的操作
expect(spy).toHaveBeenCalled()
//expect(spy).toHaveBeenCalledTimes(1) //2,3,4,5....
//expect(spy).toHaveBeenCalledWith({...})
```

**关于测试api接口**
```js
import api from ‘../../models/api’
jest.mock(‘../../models/api’, () => ({
  getInfomation: jest.fn()
}))
test(‘test api’, async () => {
//前略
await api.getInfomation.mockResolvedValue({ code: 0 }) //mock一定要写在 simulate.sleep(10)之前
await simulate.sleep(10)
expect(...)
})
```


## 运行单测
`npm run test 'filepath'` / `npm run -- --watch`(保存时即时返回结果)

如果出现类似Unexpected identifier的报错，尝试配置Babel
`yarn add --dev babel-jest @babel/core @babel/preset-env`
可以在工程的根目录下创建一个babel.config.js文件用于配置与你当前Node版本兼容的Babel：
```js
// babel.config.js
module.exports = {
  presets: [
    [
      '@babel/preset-env',
      {
        targets: {
          node: 'current',
        },
      },
    ],
  ],
};
```

如果因为`__wxConfig`等预设api出现 `is not defined`的报错
在`jest.config.js`里添加
```js
module.exports = {
  setupFilesAfterEnv: [‘./jest.setup.js’],
};
```
在根目录创建`jest.setup.js`文件，并添加
```js
//根据自己实际用到的方法添加
global.__wxConfig = { 
	envVersion: ‘develop’ 
},
global.wx = {
  navigateToMiniProgram: jest.fn(),
  showLoading: jest.fn(),
  hideLoading: jest.fn(),
  showModal: jest.fn(),
  request: jest.fn(),
  getStorageSync: jest.fn(),
  showShareMenu: jest.fn(),
  ...
}
```

## 检查覆盖率
```node
npm test -- --coverage
```
