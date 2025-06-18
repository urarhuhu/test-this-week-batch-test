---
id: components-and-apis
title: Core Components and APIs
---

React Native 提供了許多內建的[核心元件](intro-react-native-components)，可直接用於您的應用程式。您可以在左側邊欄（或窄螢幕上方的選單）中找到所有元件。如果不確定從何開始，請參考以下分類：

- [基礎元件](components-and-apis#basic-components)
- [使用者介面](components-and-apis#user-interface)
- [列表視圖](components-and-apis#list-views)
- [Android 專屬](components-and-apis#android-components-and-apis)
- [iOS 專屬](components-and-apis#ios-components-and-apis)
- [其他](components-and-apis#others)

您不僅限於使用 React Native 內建的元件與 API。React Native 擁有數千名開發者組成的社群。若需尋找特定功能的函式庫，請參閱[尋找函式庫指南](libraries#finding-libraries)。

## 基礎元件

大多數應用程式最終都會使用這些基礎元件。

<div className="component-grid component-grid-border">
  <div className="component">
    <a href="./view">
      <h3>View</h3>
      <p>The most fundamental component for building a UI.</p>
    </a>
  </div>
  <div className="component">
    <a href="./text">
      <h3>Text</h3>
      <p>A component for displaying text.</p>
    </a>
  </div>
  <div className="component">
    <a href="./image">
      <h3>Image</h3>
      <p>A component for displaying images.</p>
    </a>
  </div>
  <div className="component">
    <a href="./textinput">
      <h3>TextInput</h3>
      <p>A component for inputting text into the app via a keyboard.</p>
    </a>
  </div>
  <div className="component">
    <a href="./scrollview">
      <h3>ScrollView</h3>
      <p>Provides a scrolling container that can host multiple components and views.</p>
    </a>
  </div>
  <div className="component">
    <a href="./stylesheet">
      <h3>StyleSheet</h3>
      <p>Provides an abstraction layer similar to CSS stylesheets.</p>
    </a>
  </div>
</div>

## 使用者介面

這些通用使用者介面控制項可在任何平台上渲染。

<div className="component-grid component-grid-border">
  <div className="component">
    <a href="./button">
      <h3>Button</h3>
      <p>A basic button component for handling touches that should render nicely on any platform.</p>
    </a>
  </div>
  <div className="component">
    <a href="./switch">
      <h3>Switch</h3>
      <p>Renders a boolean input.</p>
    </a>
  </div>
</div>

## 列表視圖

與通用型 [`ScrollView`](./scrollview) 不同，以下列表視圖元件僅渲染當前顯示在畫面上的元素。這使其成為顯示長列表資料的高效能選擇。

<div className="component-grid component-grid-border">
  <div className="component">
    <a href="./flatlist">
      <h3>FlatList</h3>
      <p>A component for rendering performant scrollable lists.</p>
    </a>
  </div>
  <div className="component">
    <a href="./sectionlist">
      <h3>SectionList</h3>
      <p>Like <code>FlatList</code>, but for sectioned lists.</p>
    </a>
  </div>
</div>

## Android 元件與 API

以下許多元件封裝了常用的 Android 類別。

<div className="component-grid component-grid-border">
  <div className="component">
    <a href="./backhandler">
      <h3>BackHandler</h3>
      <p>Detect hardware button presses for back navigation.</p>
    </a>
  </div>
  <div className="component">
    <a href="./drawerlayoutandroid">
      <h3>DrawerLayoutAndroid</h3>
      <p>Renders a <code>DrawerLayout</code> on Android.</p>
    </a>
  </div>
  <div className="component">
    <a href="./permissionsandroid">
      <h3>PermissionsAndroid</h3>
      <p>Provides access to the permissions model introduced in Android M.</p>
    </a>
  </div>
  <div className="component">
    <a href="./toastandroid">
      <h3>ToastAndroid</h3>
      <p>Create an Android Toast alert.</p>
    </a>
  </div>
</div>

## iOS 元件與 API

以下許多元件封裝了常用的 UIKit 類別。

<div className="component-grid component-grid-border">
  <div className="component">
    <a href="./actionsheetios">
      <h3>ActionSheetIOS</h3>
      <p>API to display an iOS action sheet or share sheet.</p>
    </a>
  </div>
</div>

## 其他

這些元件可能對特定應用有所幫助。如需完整的元件與 API 清單，請查看左側邊欄（或窄螢幕上方的選單）。

<div className="component-grid">
  <div className="component">
    <a href="./activityindicator">
      <h3>ActivityIndicator</h3>
      <p>Displays a circular loading indicator.</p>
    </a>
  </div>
  <div className="component">
    <a href="./alert">
      <h3>Alert</h3>
      <p>Launches an alert dialog with the specified title and message.</p>
    </a>
  </div>
  <div className="component">
    <a href="./animated">
      <h3>Animated</h3>
      <p>A library for creating fluid, powerful animations that are easy to build and maintain.</p>
    </a>
  </div>
  <div className="component">
    <a href="./dimensions">
      <h3>Dimensions</h3>
      <p>Provides an interface for getting device dimensions.</p>
    </a>
  </div>
  <div className="component">
    <a href="./keyboardavoidingview">
      <h3>KeyboardAvoidingView</h3>
      <p>Provides a view that moves out of the way of the virtual keyboard automatically.</p>
    </a>
  </div>
  <div className="component">
    <a href="./linking">
      <h3>Linking</h3>
      <p>Provides a general interface to interact with both incoming and outgoing app links.</p>
    </a>
  </div>
  <div className="component">
    <a href="./modal">
      <h3>Modal</h3>
      <p>Provides a simple way to present content above an enclosing view.</p>
    </a>
  </div>
  <div className="component">
    <a href="./pixelratio">
      <h3>PixelRatio</h3>
      <p>Provides access to the device pixel density.</p>
    </a>
  </div>
  <div className="component">
    <a href="./refreshcontrol">
      <h3>RefreshControl</h3>
      <p>This component is used inside a <code>ScrollView</code> to add pull to refresh functionality.</p>
    </a>
  </div>
  <div className="component">
    <a href="./statusbar">
      <h3>StatusBar</h3>
      <p>Component to control the app status bar.</p>
    </a>
  </div>
</div>