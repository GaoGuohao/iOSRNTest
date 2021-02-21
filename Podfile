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