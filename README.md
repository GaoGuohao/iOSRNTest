#iOS通过Pod快速集成ReactNative环境

因为我们项目近期接入RN，目前介入的版本是 ReactNaitve 0.63版本，在介入过程中，发现RN的项目结构是 RN工程包含 iOS 和 Android工程的目录。

但是我们对RN的引入为模块化引入，而非全项目。这样的目录结构可能对git管理或者现用工程目录管理都是一个问题。由此考虑能不能直接在iOS的现有项目目录下直接集成RN。

看了一下网上上很多人的方式是直接将现用项目 Copy 一份到 RN工程下的 iOS目录，如果项目人员多的话，不一定每个人很好的操作，所以产生一个想法：
在iOS现有工程执行 ```pod install```的时候，自动快速集成ReactNative环境。

#### 关于RN的环境安装
这里不做太多陈述，官网说的一的非常详细：[中文安装文档](https://reactnative.cn/docs/environment-setup)；

总结就是需要安装 node 环境，和一些辅助插件工具。

#### 目录结构分析
首先我们创建空的RN项目查看RN项目的默认目录和引用设置。

``` react-native init <项目名称>```

 如：``` react-native init RNTest```，这个过程会等待一段时间，执行成功后我们查看目录：

```
RNTest
├── App.js
├── __tests__
├── android
├── app.json
├── babel.config.js
├── index.js
├── ios
├── metro.config.js
├── node_modules
├── package.json
└── yarn.lock
```

可以发现 node所有的依赖配置为 package.json文件控制，依赖全部下载在 node_modules目录下，我们打开默认自带的iOS工程，查看Podfile文件的RN pod依赖配置：
```ruby
require_relative '../node_modules/react-native/scripts/react_native_pods'
require_relative '../node_modules/@react-native-community/cli-platform-ios/native_modules'

platform :ios, '10.0'

target 'RNTest' do
  config = use_native_modules!

  use_react_native!(:path => config["reactNativePath"])

  target 'RNTestTests' do
    inherit! :complete
    # Pods for testing
  end

  # Enables Flipper.
  #
  # Note that if you have use_frameworks! enabled, Flipper will not work and
  # you should disable these next few lines.
  use_flipper!
  post_install do |installer|
    flipper_post_install(installer)
  end
end

target 'RNTest-tvOS' do
  # Pods for RNTest-tvOS

  target 'RNTest-tvOSTests' do
    inherit! :search_paths
    # Pods for testing
  end
end
```

首先可以看到， ReactNative 0.63版本支持的iOS最低系统版本为 iOS 10.0。

依赖了两个文件：
```
require_relative '../node_modules/react-native/scripts/react_native_pods'
require_relative '../node_modules/@react-native-community/cli-platform-ios/native_modules'
```
其他通过 pod path的方式依赖的本地资源：
```ruby 
config = use_native_modules!
use_react_native!(:path => config["reactNativePath"]) 
```

我们查看一些 node_modules/react-native/scripts/react_native_pods的这个文，如下：

```ruby
# Copyright (c) Facebook, Inc. and its affiliates.
#
# This source code is licensed under the MIT license found in the
# LICENSE file in the root directory of this source tree.

def use_react_native! (options={})
  # The prefix to the react-native
  prefix = options[:path] ||= "../node_modules/react-native"

  # Include Fabric dependencies
  fabric_enabled = options[:fabric_enabled] ||= false

  # Include DevSupport dependency
  production = options[:production] ||= false

  # The Pods which should be included in all projects
  pod 'FBLazyVector', :path => "#{prefix}/Libraries/FBLazyVector"
  pod 'FBReactNativeSpec', :path => "#{prefix}/Libraries/FBReactNativeSpec"
  pod 'RCTRequired', :path => "#{prefix}/Libraries/RCTRequired"
  pod 'RCTTypeSafety', :path => "#{prefix}/Libraries/TypeSafety"
  pod 'React', :path => "#{prefix}/"
  pod 'React-Core', :path => "#{prefix}/"
  pod 'React-CoreModules', :path => "#{prefix}/React/CoreModules"
  pod 'React-RCTActionSheet', :path => "#{prefix}/Libraries/ActionSheetIOS"
  pod 'React-RCTAnimation', :path => "#{prefix}/Libraries/NativeAnimation"
  pod 'React-RCTBlob', :path => "#{prefix}/Libraries/Blob"
  pod 'React-RCTImage', :path => "#{prefix}/Libraries/Image"
  pod 'React-RCTLinking', :path => "#{prefix}/Libraries/LinkingIOS"
  pod 'React-RCTNetwork', :path => "#{prefix}/Libraries/Network"
  pod 'React-RCTSettings', :path => "#{prefix}/Libraries/Settings"
  pod 'React-RCTText', :path => "#{prefix}/Libraries/Text"
  pod 'React-RCTVibration', :path => "#{prefix}/Libraries/Vibration"
  pod 'React-Core/RCTWebSocket', :path => "#{prefix}/"

  unless production
    pod 'React-Core/DevSupport', :path => "#{prefix}/"
  end

  pod 'React-cxxreact', :path => "#{prefix}/ReactCommon/cxxreact"
  pod 'React-jsi', :path => "#{prefix}/ReactCommon/jsi"
  pod 'React-jsiexecutor', :path => "#{prefix}/ReactCommon/jsiexecutor"
  pod 'React-jsinspector', :path => "#{prefix}/ReactCommon/jsinspector"
  pod 'React-callinvoker', :path => "#{prefix}/ReactCommon/callinvoker"
  pod 'ReactCommon/turbomodule/core', :path => "#{prefix}/ReactCommon"
  pod 'Yoga', :path => "#{prefix}/ReactCommon/yoga", :modular_headers => true

  pod 'DoubleConversion', :podspec => "#{prefix}/third-party-podspecs/DoubleConversion.podspec"
  pod 'glog', :podspec => "#{prefix}/third-party-podspecs/glog.podspec"
  pod 'Folly', :podspec => "#{prefix}/third-party-podspecs/Folly.podspec"

  if fabric_enabled
    pod 'React-Fabric', :path => "#{prefix}/ReactCommon"
    pod 'React-graphics', :path => "#{prefix}/ReactCommon/fabric/graphics"
    pod 'React-jsi/Fabric', :path => "#{prefix}/ReactCommon/jsi"
    pod 'React-RCTFabric', :path => "#{prefix}/React"
    pod 'Folly/Fabric', :podspec => "#{prefix}/third-party-podspecs/Folly.podspec"
  end
end

def use_flipper!(versions = {}, configurations: ['Debug'])
  versions['Flipper'] ||= '~> 0.54.0'
  versions['Flipper-DoubleConversion'] ||= '1.1.7'
  versions['Flipper-Folly'] ||= '~> 2.2'
  versions['Flipper-Glog'] ||= '0.3.6'
  versions['Flipper-PeerTalk'] ||= '~> 0.0.4'
  versions['Flipper-RSocket'] ||= '~> 1.1'
  pod 'FlipperKit', versions['Flipper'], :configurations => configurations
  pod 'FlipperKit/FlipperKitLayoutPlugin', versions['Flipper'], :configurations => configurations
  pod 'FlipperKit/SKIOSNetworkPlugin', versions['Flipper'], :configurations => configurations
  pod 'FlipperKit/FlipperKitUserDefaultsPlugin', versions['Flipper'], :configurations => configurations
  pod 'FlipperKit/FlipperKitReactPlugin', versions['Flipper'], :configurations => configurations
  # List all transitive dependencies for FlipperKit pods
  # to avoid them being linked in Release builds
  pod 'Flipper', versions['Flipper'], :configurations => configurations
  pod 'Flipper-DoubleConversion', versions['Flipper-DoubleConversion'], :configurations => configurations
  pod 'Flipper-Folly', versions['Flipper-Folly'], :configurations => configurations
  pod 'Flipper-Glog', versions['Flipper-Glog'], :configurations => configurations
  pod 'Flipper-PeerTalk', versions['Flipper-PeerTalk'], :configurations => configurations
  pod 'Flipper-RSocket', versions['Flipper-RSocket'], :configurations => configurations
  pod 'FlipperKit/Core', versions['Flipper'], :configurations => configurations
  pod 'FlipperKit/CppBridge', versions['Flipper'], :configurations => configurations
  pod 'FlipperKit/FBCxxFollyDynamicConvert', versions['Flipper'], :configurations => configurations
  pod 'FlipperKit/FBDefines', versions['Flipper'], :configurations => configurations
  pod 'FlipperKit/FKPortForwarding', versions['Flipper'], :configurations => configurations
  pod 'FlipperKit/FlipperKitHighlightOverlay', versions['Flipper'], :configurations => configurations
  pod 'FlipperKit/FlipperKitLayoutTextSearchable', versions['Flipper'], :configurations => configurations
  pod 'FlipperKit/FlipperKitNetworkPlugin', versions['Flipper'], :configurations => configurations
end

# Post Install processing for Flipper
def flipper_post_install(installer)
  installer.pods_project.targets.each do |target|
    if target.name == 'YogaKit'
      target.build_configurations.each do |config|
        config.build_settings['SWIFT_VERSION'] = '4.1'
      end
    end
  end
end

```

第二行代码就是路径设置：
```ruby
# The prefix to the react-native
prefix = options[:path] ||= "../node_modules/react-native"
```
综上述可以发现，ReactNative的所有本地的path依赖都是 ```prefix```的设置，所以我们只要控制好这个文件的目录的设置即可，但是我们人工修改也不太合适，所以可以考虑脚本自动实现修改。

#### 集成到现有iOS工程下

如果我们想实现在现有iOS工程下，集成RN环境，不保留上一层RN工程目录，其实就是把 node_modules 目录保留到 iOS当前项目下一份，修改对应的路径配置即可，也就是 所有的 ```../node_modules``` 改为 ```./node_modules```，切不要每个人手动操作，而是通过脚本自动完成。

##### 1.创建一个iOS Demo.
这里我们创建一个Demo演示，直接用Xcode创建iOS项目 iOSRNTest。
##### 2.创建 package.json node依赖配置。
在刚创建的iOS项目目录下，创建文件 package.json node依赖管理文件：

```shell touch package.json ```

文件配置：
```json
{
  "name": "<项目名>",
  "version": "<项目版本>",
  "private": true,
  "dependencies": {
    "react-native": "0.63.4"
  }
}
```
其他的不需要依赖，在iOS工程中，只依赖 ```"react-native": "0.63.4"```即可。
注意：这个文件需要和iOS项目一起提交到git仓库。
##### 3.编写 pod 脚本
我们的目标是在执行 ``` pod install```的时候 自动检测node环境，且自动将ReactNative环境依赖到iOS工程中。
pod工具就是通过ruby语言编写的，所以我们可以插入ruby脚本来做一些自动化的操作。在iOS工程目录下创建ruby脚本文件 Podfile_ReactNative.rb，开始编写脚本：
```ruby
# 定义一个函数，在 Podfile文件中调用此函数即可
def installReactNativeSdk()

    # 设置 react_native_pods.rb 文件路径
    node_mudle_pod_file = "node_modules/react-native/scripts/react_native_pods.rb"

    # 判断该文件是否存在，如果已经存在，表示RN环境已经配置，如果没有存在表示RN环境还未集成到项目
    if File.exist?(node_mudle_pod_file)
        Pod::UI.puts "\nReactNative 环境已存在！\n\n"
    else
        Pod::UI.puts "ReactNative 环境不存在，准备下载···"
        # 判断是否安装 node环境
        if system "node -v > /dev/null"
            # 使用 yarn 或 npm 下载依赖
            if system "yarn install || npm install"
                Pod::UI.puts "ReactNative 环境安装成功！\n\n"
            else
                Pod::UI.puts "ReactNative 环境安装失败！请安装yarn，在命令行执行：npm install -g yarn"
                Kernel.exit(false)
            end
        else
            #如果没有安装，提示自行安装node环境
            Pod::UI.puts "环境下载失败！请先安装node环境，详细见：https://reactnative.cn/docs/environment-setup"
            Kernel.exit(false)
        end
    end
end
```
此时ruby的脚本已将编写完成，接下来让我们去配置 Podfile文件的设置。

##### 4.配置 Podfile 环境
如果项目下还没有 Podfile文件，可以通过 ``` pod init ```创建，如果已经存在直接设置。

这里还是用上面创建的空项目 ```iOSRNTest```演示，编辑 Podfile:
``` ruby
# Uncomment the next line to define a global platform for your project

# 设置下载源
source 'https://github.com/CocoaPods/Specs.git'

# 导入我们自定义的脚本
require_relative './Podfile_ReactNative'

# 执行我们编写的RN环境检测代码
installReactNativeSdk()

# 设置RN配置 依赖，这里需要注意，不要使用 ../node_modules/,而是直接node_modules/
require_relative 'node_modules/react-native/scripts/react_native_pods'

# 这里需要注意，RN 0.63版本必须iOS10.0以上版本才支持
platform :ios, '10.0'

target 'iOSRNTest' do
  # Comment the next line if you don't want to use dynamic frameworks
  use_frameworks!

  # 设置RN Path 依赖
  use_react_native!(:path => "node_modules/react-native")
end

```

到这里环境全部配置完成，我们可以直接执行 ``` pod install ```来尝试是否可以快速集成RN环境。
如果不出问题，控制台将会显示下载ReactNative环境日志。
```shell
➜  iOSRNTest git:(main) ✗ pod install
ReactNative 环境不存在，准备下载···
yarn install v1.22.10
warning ../../../package.json: No license field
[1/4] 🔍  Resolving packages...
```
等执行结束后，打开 iOSRNTest.xcworkspace 工程，可以看到 ReactNative环境已经全部集成进去！

##### 5.过滤目录
这里需要注意的是，node_modules目录为RN依赖的资源，没必要提交到git工程，可以在```.gitignore```文件中过滤掉。

目前我们项目通过这中方式集成RN，其他开发者不太需要关心RN的配置，只要会执行 ```pod install```即可！

因为刚开始做ReactNative，如果您有更好的建议 欢迎评论！

项目Demo提交到[Github](https://github.com/GaoGuohao/iOSRNTest.git)，可以参考！

注意：
git clone后，需要保证本地已经安装node的环境！
直接执行 ```pod install``` ！
