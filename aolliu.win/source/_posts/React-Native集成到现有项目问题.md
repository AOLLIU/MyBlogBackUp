---
title: React Native集成到现有项目问题
date: 2016-8-24 21:39:09
tags:
    - 跨平台开发
---

 - 最近将写写的一个React-Native界面,集成到现有的项目中,在这个过程中遇到了问题,但是从网上没有看到答案,经过尝试,最后解决了,感觉大家在今后的开发中可能会也遇到同样的方法

## 一,React-Native集成到现有项目

### 1.参考:
 网上有很多集成的步骤,而且写的很详细,我参考的是[这篇文章](http://www.lcode.org/react-native-integrating/),其他文章都和这篇文章大同小异

<!--more-->

### 2.问题:
 从本人集成过程中发现了问题,也不知道其他同学有没有遇见,所以...
 - 1.文中



![Snip20161114_1.png](http://upload-images.jianshu.io/upload_images/1494773-f82b7f090cf91439.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 到这一步确实会报很多错误,但是我按文中说把对应的库`JavaScriptCore.framework`引入,还是报错的,报错如下

```
Ld /Users/liuwei/Library/Developer/Xcode/DerivedData/Project-ayrfjucpjfizeidykzmsvsmbblkn/Build/Products/Debug-iphonesimulator/Project.app/Project normal x86_64
    cd /Users/liuwei/Desktop/Demo/Project
    export IPHONEOS_DEPLOYMENT_TARGET=9.3
    export PATH="/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/usr/bin:/Applications/Xcode.app/Contents/Developer/usr/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin"
    /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -arch x86_64 -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator9.3.sdk -L/Users/liuwei/Library/Developer/Xcode/DerivedData/Project-ayrfjucpjfizeidykzmsvsmbblkn/Build/Products/Debug-iphonesimulator -F/Users/liuwei/Library/Developer/Xcode/DerivedData/Project-ayrfjucpjfizeidykzmsvsmbblkn/Build/Products/Debug-iphonesimulator -filelist /Users/liuwei/Library/Developer/Xcode/DerivedData/Project-ayrfjucpjfizeidykzmsvsmbblkn/Build/Intermediates/Project.build/Debug-iphonesimulator/Project.build/Objects-normal/x86_64/Project.LinkFileList -Xlinker -rpath -Xlinker @executable_path/Frameworks -mios-simulator-version-min=9.3 -Xlinker -no_deduplicate -Xlinker -objc_abi_version -Xlinker 2 -fobjc-arc -fobjc-link-runtime /Users/liuwei/Library/Developer/Xcode/DerivedData/Project-ayrfjucpjfizeidykzmsvsmbblkn/Build/Products/Debug-iphonesimulator/libReact.a /Users/liuwei/Library/Developer/Xcode/DerivedData/Project-ayrfjucpjfizeidykzmsvsmbblkn/Build/Products/Debug-iphonesimulator/libRCTActionSheet.a /Users/liuwei/Library/Developer/Xcode/DerivedData/Project-ayrfjucpjfizeidykzmsvsmbblkn/Build/Products/Debug-iphonesimulator/libRCTGeolocation.a /Users/liuwei/Library/Developer/Xcode/DerivedData/Project-ayrfjucpjfizeidykzmsvsmbblkn/Build/Products/Debug-iphonesimulator/libRCTImage.a /Users/liuwei/Library/Developer/Xcode/DerivedData/Project-ayrfjucpjfizeidykzmsvsmbblkn/Build/Products/Debug-iphonesimulator/libRCTLinking.a /Users/liuwei/Library/Developer/Xcode/DerivedData/Project-ayrfjucpjfizeidykzmsvsmbblkn/Build/Products/Debug-iphonesimulator/libRCTNetwork.a /Users/liuwei/Library/Developer/Xcode/DerivedData/Project-ayrfjucpjfizeidykzmsvsmbblkn/Build/Products/Debug-iphonesimulator/libRCTSettings.a /Users/liuwei/Library/Developer/Xcode/DerivedData/Project-ayrfjucpjfizeidykzmsvsmbblkn/Build/Products/Debug-iphonesimulator/libRCTText.a /Users/liuwei/Library/Developer/Xcode/DerivedData/Project-ayrfjucpjfizeidykzmsvsmbblkn/Build/Products/Debug-iphonesimulator/libRCTVibration.a /Users/liuwei/Library/Developer/Xcode/DerivedData/Project-ayrfjucpjfizeidykzmsvsmbblkn/Build/Products/Debug-iphonesimulator/libRCTWebSocket.a -Xlinker -dependency_info -Xlinker /Users/liuwei/Library/Developer/Xcode/DerivedData/Project-ayrfjucpjfizeidykzmsvsmbblkn/Build/Intermediates/Project.build/Debug-iphonesimulator/Project.build/Objects-normal/x86_64/Project_dependency_info.dat -o /Users/liuwei/Library/Developer/Xcode/DerivedData/Project-ayrfjucpjfizeidykzmsvsmbblkn/Build/Products/Debug-iphonesimulator/Project.app/Project

Undefined symbols for architecture x86_64:
  "std::__1::__next_prime(unsigned long)", referenced from:
      std::__1::__hash_table<std::__1::__hash_value_type<unsigned long, unsigned long>, std::__1::__unordered_map_hasher<unsigned long, std::__1::__hash_value_type<unsigned long, unsigned long>, std::__1::hash<unsigned long>, true>, std::__1::__unordered_map_equal<unsigned long, std::__1::__hash_value_type<unsigned long, unsigned long>, std::__1::equal_to<unsigned long>, true>, std::__1::allocator<std::__1::__hash_value_type<unsigned long, unsigned long> > >::rehash(unsigned long) in libReact.a(RCTJSCExecutor.o)
  "std::__1::mutex::lock()", referenced from:
      -[RCTModuleData setUpInstanceAndBridge] in libReact.a(RCTModuleData.o)
  "std::__1::mutex::unlock()", referenced from:
      -[RCTModuleData setUpInstanceAndBridge] in libReact.a(RCTModuleData.o)
  "std::__1::mutex::~mutex()", referenced from:
      -[RCTModuleData .cxx_destruct] in libReact.a(RCTModuleData.o)
  "std::terminate()", referenced from:
      ___clang_call_terminate in libReact.a(RCTJSCExecutor.o)
  "operator delete[](void*)", referenced from:
      -[RCTJSCExecutor dealloc] in libReact.a(RCTJSCExecutor.o)
      executeRandomAccessModule(RCTJSCExecutor*, unsigned int, unsigned long, unsigned long) in libReact.a(RCTJSCExecutor.o)
      readRAMBundle(std::__1::unique_ptr<__sFILE, int (*)(__sFILE*)>, RandomAccessBundleData&) in libReact.a(RCTJSCExecutor.o)
      RandomAccessBundleData::~RandomAccessBundleData() in libReact.a(RCTJSCExecutor.o)
  "operator delete(void*)", referenced from:
      std::__1::__hash_table<std::__1::__hash_value_type<unsigned long, unsigned long>, std::__1::__unordered_map_hasher<unsigned long, std::__1::__hash_value_type<unsigned long, unsigned long>, std::__1::hash<unsigned long>, true>, std::__1::__unordered_map_equal<unsigned long, std::__1::__hash_value_type<unsigned long, unsigned long>, std::__1::equal_to<unsigned long>, true>, std::__1::allocator<std::__1::__hash_value_type<unsigned long, unsigned long> > >::~__hash_table() in libReact.a(RCTJSCExecutor.o)
      std::__1::__hash_table<std::__1::__hash_value_type<unsigned long, unsigned long>, std::__1::__unordered_map_hasher<unsigned long, std::__1::__hash_value_type<unsigned long, unsigned long>, std::__1::hash<unsigned long>, true>, std::__1::__unordered_map_equal<unsigned long, std::__1::__hash_value_type<unsigned long, unsigned long>, std::__1::equal_to<unsigned long>, true>, std::__1::allocator<std::__1::__hash_value_type<unsigned long, unsigned long> > >::__deallocate(std::__1::__hash_node<std::__1::__hash_value_type<unsigned long, unsigned long>, void*>*) in libReact.a(RCTJSCExecutor.o)
      std::__1::pair<std::__1::__hash_iterator<std::__1::__hash_node<std::__1::__hash_value_type<unsigned long, unsigned long>, void*>*>, bool> std::__1::__hash_table<std::__1::__hash_value_type<unsigned long, unsigned long>, std::__1::__unordered_map_hasher<unsigned long, std::__1::__hash_value_type<unsigned long, unsigned long>, std::__1::hash<unsigned long>, true>, std::__1::__unordered_map_equal<unsigned long, std::__1::__hash_value_type<unsigned long, unsigned long>, std::__1::equal_to<unsigned long>, true>, std::__1::allocator<std::__1::__hash_value_type<unsigned long, unsigned long> > >::__insert_unique<std::__1::pair<unsigned long const, unsigned long> const&>(std::__1::pair<unsigned long const, unsigned long> const&&&) in libReact.a(RCTJSCExecutor.o)
      std::__1::unique_ptr<std::__1::__hash_node<std::__1::__hash_value_type<unsigned long, unsigned long>, void*>, std::__1::__hash_node_destructor<std::__1::allocator<std::__1::__hash_node<std::__1::__hash_value_type<unsigned long, unsigned long>, void*> > > > std::__1::__hash_table<std::__1::__hash_value_type<unsigned long, unsigned long>, std::__1::__unordered_map_hasher<unsigned long, std::__1::__hash_value_type<unsigned long, unsigned long>, std::__1::hash<unsigned long>, true>, std::__1::__unordered_map_equal<unsigned long, std::__1::__hash_value_type<unsigned long, unsigned long>, std::__1::equal_to<unsigned long>, true>, std::__1::allocator<std::__1::__hash_value_type<unsigned long, unsigned long> > >::__construct_node<std::__1::pair<unsigned long const, unsigned long> const&>(std::__1::pair<unsigned long const, unsigned long> const&&&) in libReact.a(RCTJSCExecutor.o)
      std::__1::__hash_table<std::__1::__hash_value_type<unsigned long, unsigned long>, std::__1::__unordered_map_hasher<unsigned long, std::__1::__hash_value_type<unsigned long, unsigned long>, std::__1::hash<unsigned long>, true>, std::__1::__unordered_map_equal<unsigned long, std::__1::__hash_value_type<unsigned long, unsigned long>, std::__1::equal_to<unsigned long>, true>, std::__1::allocator<std::__1::__hash_value_type<unsigned long, unsigned long> > >::__rehash(unsigned long) in libReact.a(RCTJSCExecutor.o)
      std::__1::__hash_table<std::__1::__hash_value_type<unsigned long, unsigned long>, std::__1::__unordered_map_hasher<unsigned long, std::__1::__hash_value_type<unsigned long, unsigned long>, std::__1::hash<unsigned long>, true>, std::__1::__unordered_map_equal<unsigned long, std::__1::__hash_value_type<unsigned long, unsigned long>, std::__1::equal_to<unsigned long>, true>, std::__1::allocator<std::__1::__hash_value_type<unsigned long, unsigned long> > >::erase(std::__1::__hash_const_iterator<std::__1::__hash_node<std::__1::__hash_value_type<unsigned long, unsigned long>, void*>*>) in libReact.a(RCTJSCExecutor.o)
  "operator new[](unsigned long)", referenced from:
      executeRandomAccessModule(RCTJSCExecutor*, unsigned int, unsigned long, unsigned long) in libReact.a(RCTJSCExecutor.o)
      readRAMBundle(std::__1::unique_ptr<__sFILE, int (*)(__sFILE*)>, RandomAccessBundleData&) in libReact.a(RCTJSCExecutor.o)
  "operator new(unsigned long)", referenced from:
      std::__1::unique_ptr<std::__1::__hash_node<std::__1::__hash_value_type<unsigned long, unsigned long>, void*>, std::__1::__hash_node_destructor<std::__1::allocator<std::__1::__hash_node<std::__1::__hash_value_type<unsigned long, unsigned long>, void*> > > > std::__1::__hash_table<std::__1::__hash_value_type<unsigned long, unsigned long>, std::__1::__unordered_map_hasher<unsigned long, std::__1::__hash_value_type<unsigned long, unsigned long>, std::__1::hash<unsigned long>, true>, std::__1::__unordered_map_equal<unsigned long, std::__1::__hash_value_type<unsigned long, unsigned long>, std::__1::equal_to<unsigned long>, true>, std::__1::allocator<std::__1::__hash_value_type<unsigned long, unsigned long> > >::__construct_node<std::__1::pair<unsigned long const, unsigned long> const&>(std::__1::pair<unsigned long const, unsigned long> const&&&) in libReact.a(RCTJSCExecutor.o)
      std::__1::__hash_table<std::__1::__hash_value_type<unsigned long, unsigned long>, std::__1::__unordered_map_hasher<unsigned long, std::__1::__hash_value_type<unsigned long, unsigned long>, std::__1::hash<unsigned long>, true>, std::__1::__unordered_map_equal<unsigned long, std::__1::__hash_value_type<unsigned long, unsigned long>, std::__1::equal_to<unsigned long>, true>, std::__1::allocator<std::__1::__hash_value_type<unsigned long, unsigned long> > >::__rehash(unsigned long) in libReact.a(RCTJSCExecutor.o)
  "___cxa_begin_catch", referenced from:
      ___clang_call_terminate in libReact.a(RCTJSCExecutor.o)
  "___gxx_personality_v0", referenced from:
      -[RCTJavaScriptContext initWithJSContext:onThread:] in libReact.a(RCTJSCExecutor.o)
      -[RCTJavaScriptContext init] in libReact.a(RCTJSCExecutor.o)
      -[RCTJavaScriptContext invalidate] in libReact.a(RCTJSCExecutor.o)
      +[RCTJSCExecutor runRunLoopThread] in libReact.a(RCTJSCExecutor.o)
      -[RCTJSCExecutor setBridge:] in libReact.a(RCTJSCExecutor.o)
      -[RCTJSCExecutor init] in libReact.a(RCTJSCExecutor.o)
      -[RCTJSCExecutor initWithUseCustomJSCLibrary:] in libReact.a(RCTJSCExecutor.o)
      ...
ld: symbol(s) not found for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)

```
- 2.解决方法如下:


![Snip20161114_3.png](http://upload-images.jianshu.io/upload_images/1494773-0a153b1bc6a18bfa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- 3.运行,大功告成!


![Snip20161114_4.png](http://upload-images.jianshu.io/upload_images/1494773-b996104f1adf8d79.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


