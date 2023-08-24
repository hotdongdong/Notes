## 安装

```tsx
npm install @react-navigation/native @react-navigation/stack
```

## 设置路由配置文件 app.navigator.tsx

```tsx
import React from 'react';
import {NavigationContainer} from '@react-navigation/native';
import {createStackNavigator} from '@react-navigation/stack';

interface IPreviewModeProps {
  itemId:number,
  otherParams:string
}

// NavigatorParamList类型定义了路由参数的类型
// undefined 表示该屏幕不需要参数
// IPreviewModeProps表示该屏幕需要传递一个类型为IPreviewModeProps的参数
export type NavigatorParamList = {
  Start: undefined;
  InfomationConfirmation: undefined;
  PreviewMode: IPreviewModeProps;
};

// 使用 createStackNavigator创建了一个堆栈导航器（Stack Navigator）
// 它接受一个泛型参数，用于指定路由参数的类型。
const Stack = createStackNavigator<NavigatorParamList>();

// 将屏幕模块化，易于开发和维护
const test1Screen = () => {
  <>
    // 每个屏幕都用`Stack.Screen`包装起来，其中指定屏幕的名称和组件
    <Stack.Screen name="Start" component={StartScreen} />
    <Stack.Screen name="InfomationConfirmation" component={InfomationConfirmationScreen} />
    <Stack.Screen name="PreviewMode" component={PreviewModeScreen} />
  </>
}

const test2Screen = () => {...}
const test3Screen = () => {...}
const test4Screen = () => {...}

const AppStack = () => {
  return (
    <Stack.Navigator
      initialRouteName="Start"
      screenOptions={
        {
          headerShown: false,
        }
      }>
      {moduleStoInScreen()}
      {test2Screen()}
      {test3Screen()}
      {test4Screen()}
    </Stack.Navigator>
  );
};

export const AppNavigator = () => {
  return (
    <NavigationContainer>
      <AppStack />
    </NavigationContainer>
  );
};
```

## 引入根组件 App.tsx

```tsx
import React from "react";

import { AppNavigator } from "./navigators/app-navigator";

export const App = () => {
  return <AppNavigator />;
};
```

## 实现路由跳转

```tsx
import { TouchableOpacity, View } from "react-native";
import { useNavigation } from "@react-navigation/native";

const navigation: StackNavigationProp<NavigatorParamList> = useNavigation();

// 携带参数
const click = () => {
  navigation.navigate("PreviewMode", {
    itemId: 123,
    otherParams: "Hello PreviewModeScreen",
  });
};

// 屏幕1
export const TextScreen = () => {
  return <Button title="button" onPress={() => click()} />;
};

// 屏幕2
// 通过route来接收参数
export const PreviewModeScreen = ({ route }) => {
  const { itemId, otherParams } = route?.params;

  return (
    <View>
      <Text>PreviewMode Screen</Text>
      <Text>Item ID: {itemId}</Text>
      <Text>Other Param: {otherParam}</Text>
    </View>
  );
};
```
