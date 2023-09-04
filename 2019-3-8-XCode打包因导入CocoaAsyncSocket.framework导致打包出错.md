---
layout:     post
title:      打包因导入CocoaAsyncSocket.framework导致打包出错
subtitle:   因AppStore不允许framework内包含i386或者其他平台的内容
date:       2019-3-8
author:     sinfml
catalog: true
tags:
    - iOS
    - XCode
---

### 问题

出现:
ERROR ITMS-90087: "Unsupported Architectures. The executable for LiveStorage.app/Frameworks/SpeechSDK.framework contains unsupported architectures '[x86_64, i386]'."

 

 ERROR ITMS-90209: "Invalid Segment Alignment. The app binary at 'LiveStorage.app/Frameworks/SpeechSDK.framework/SpeechSDK' does not have proper segment alignment. Try rebuilding the app with the latest Xcode version."

 

ERROR ITMS-90125: "The binary is invalid. The encryption info in the LC_ENCRYPTION_INFO load command is either missing or invalid, or the binary is already encrypted. This binary does not seem to have been built with Apple's linker."

### fix

在TARGETS->Build Phases->点击加号选择New Run Script Phase->然后复制粘贴下面代码

```shell

APP_PATH="${TARGET_BUILD_DIR}/${WRAPPER_NAME}"

# This script loops through the frameworks embedded in the application and
# removes unused architectures.
find "$APP_PATH" -name '*.framework' -type d | while read -r FRAMEWORK
do
FRAMEWORK_EXECUTABLE_NAME=$(defaults read "$FRAMEWORK/Info.plist" CFBundleExecutable)
FRAMEWORK_EXECUTABLE_PATH="$FRAMEWORK/$FRAMEWORK_EXECUTABLE_NAME"
echo "Executable is $FRAMEWORK_EXECUTABLE_PATH"

EXTRACTED_ARCHS=()

for ARCH in $ARCHS
do
echo "Extracting $ARCH from $FRAMEWORK_EXECUTABLE_NAME"
lipo -extract "$ARCH" "$FRAMEWORK_EXECUTABLE_PATH" -o "$FRAMEWORK_EXECUTABLE_PATH-$ARCH"
EXTRACTED_ARCHS+=("$FRAMEWORK_EXECUTABLE_PATH-$ARCH")
done

echo "Merging extracted architectures: ${ARCHS}"
lipo -o "$FRAMEWORK_EXECUTABLE_PATH-merged" -create "${EXTRACTED_ARCHS[@]}"
rm "${EXTRACTED_ARCHS[@]}"

echo "Replacing original executable with thinned version"
rm "$FRAMEWORK_EXECUTABLE_PATH"
mv "$FRAMEWORK_EXECUTABLE_PATH-merged" "$FRAMEWORK_EXECUTABLE_PATH"

done

```