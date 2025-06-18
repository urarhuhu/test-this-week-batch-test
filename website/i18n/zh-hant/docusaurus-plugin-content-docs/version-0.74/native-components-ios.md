---
id: native-components-ios
title: iOS Native UI Components
---

import NativeDeprecated from './the-new-architecture/\_markdown_native_deprecation.mdx'

<NativeDeprecated />

現有大量原生UI元件可直接用於最新應用程式——有些是平台內建元件，其他則來自第三方函式庫，甚至可能是您過往專案中自行開發的元件。React Native已封裝了部分關鍵平台元件（如`ScrollView`和`TextInput`），但並未涵蓋所有元件，特別是您過去為其他應用程式開發的自訂元件。幸運的是，我們可以封裝這些現有元件，使其能與React Native應用無縫整合。

與原生模組指南類似，本篇同屬進階指南，假設您已具備iOS開發基礎知識。本文將透過實作核心React Native函式庫中現有`MapView`元件的部分功能，示範如何建構原生UI元件。

## iOS MapView範例

假設我們需要在應用程式中加入互動式地圖——與其從頭開發，不如直接使用[`MKMapView`](https://developer.apple.com/library/prerelease/mac/documentation/MapKit/Reference/MKMapView_Class/index.html)，我們只需讓它能透過JavaScript操作即可。

原生視圖由`RCTViewManager`的子類別建立與管理。這些子類別功能類似視圖控制器，但本質上是單例模式——橋接器只會為每個子類別建立一個實例。它們將原生視圖暴露給`RCTUIManager`，而`RCTUIManager`會反過來委派它們根據需要設定與更新視圖屬性。`RCTViewManager`通常也作為視圖的委派對象，透過橋接器將事件回傳至JavaScript。

要暴露一個視圖，您可以：

- 繼承`RCTViewManager`來建立元件的管理器
- 添加`RCT_EXPORT_MODULE()`標記宏
- 實作`-(UIView *)view`方法

```objectivec title='RNTMapManager.m'
#import <MapKit/MapKit.h>

#import <React/RCTViewManager.h>

@interface RNTMapManager : RCTViewManager
@end

@implementation RNTMapManager

RCT_EXPORT_MODULE(RNTMap)

- (UIView *)view
{
  return [[MKMapView alloc] init];
}

@end
```

:::note
請勿嘗試在透過`-view`方法暴露的`UIView`實例上設定`frame`或`backgroundColor`屬性。
React Native會覆蓋自訂類別設定的值，以符合JavaScript元件的佈局屬性。
若需此級別的控制，建議將欲設定樣式的`UIView`實例包裹在另一個`UIView`中，並返回包裹後的`UIView`。
詳見[Issue 2948](https://github.com/facebook/react-native/issues/2948)獲取更多背景資訊。
:::

:::info
上述範例中，我們為類別名稱添加了`RNT`前綴。前綴用於避免與其他框架命名衝突。
Apple框架使用雙字母前綴，React Native則使用`RCT`前綴。為避免命名衝突，建議在自訂類別中使用非`RCT`的三字母前綴。
:::

接著您需要少量JavaScript程式碼使其成為可用的React元件：

```tsx title="MapView.tsx"
import {requireNativeComponent} from 'react-native';

// requireNativeComponent automatically resolves 'RNTMap' to 'RNTMapManager'
module.exports = requireNativeComponent('RNTMap');
```

```tsx title="MyApp.tsx"
import MapView from './MapView.tsx';

...

render() {
  return <MapView style={{flex: 1}} />;
}
```

請確保此處使用`RNTMap`。我們需要在此引入管理器，這會將管理器的視圖暴露給JavaScript使用。

:::note
渲染時請記得拉伸視圖，否則您將只會看到空白畫面。
:::

```tsx
  render() {
    return <MapView style={{flex: 1}} />;
  }
```

現在這已是JavaScript中功能完整的原生地圖視圖元件，支援雙指縮放等原生手勢操作。不過，我們還無法真正從JavaScript控制它 :(

## 屬性

要提升元件可用性，首先可橋接部分原生屬性。假設我們想禁用縮放功能並指定可見區域。禁用縮放是布林值，只需添加這行程式碼：

```objectivec title='RNTMapManager.m'
RCT_EXPORT_VIEW_PROPERTY(zoomEnabled, BOOL)
```

請注意我們明確將類型指定為`BOOL`——React Native在橋接通信時會使用`RCTConvert`來轉換各種數據類型，錯誤值會顯示方便的「RedBox」錯誤訊息讓您立即發現問題。當處理如此直觀的類型時，整個轉換過程都會由這個宏自動處理完成。

現在要實際禁用縮放功能，我們在JS中設置屬性：

```tsx title="MyApp.tsx"
<MapView zoomEnabled={false} style={{flex: 1}} />
```

為了記錄MapView組件的屬性（及其接受的數值類型），我們將添加一個包裝組件並使用React的`PropTypes`來定義介面：

```tsx title="MapView.tsx"
import PropTypes from 'prop-types';
import React from 'react';
import {requireNativeComponent} from 'react-native';

class MapView extends React.Component {
  render() {
    return <RNTMap {...this.props} />;
  }
}

MapView.propTypes = {
  /**
   * A Boolean value that determines whether the user may use pinch
   * gestures to zoom in and out of the map.
   */
  zoomEnabled: PropTypes.bool,
};

const RNTMap = requireNativeComponent('RNTMap');

module.exports = MapView;
```

現在我們有了一個文檔完善的包裝組件可供使用。

接著讓我們添加更複雜的`region`屬性。首先添加原生代碼：

```objectivec title='RNTMapManager.m'
RCT_CUSTOM_VIEW_PROPERTY(region, MKCoordinateRegion, MKMapView)
{
  [view setRegion:json ? [RCTConvert MKCoordinateRegion:json] : defaultView.region animated:YES];
}
```

這比之前處理的`BOOL`情況複雜許多。現在我們需要為`MKCoordinateRegion`類型編寫轉換函數，並添加自定義代碼使視圖在從JS設置區域時能產生動畫效果。在我們提供的函數體內，`json`參數代表從JS傳入的原始值。`view`變量讓我們能訪問管理器的視圖實例，而`defaultView`則用於在JS傳入null時將屬性重置為默認值。

您可以為視圖編寫任何需要的轉換函數——這裡是通過`RCTConvert`分類實現的`MKCoordinateRegion`轉換方案，它利用了ReactNative現有的`RCTConvert+CoreLocation`分類：

```objectivec title='RNTMapManager.m'
#import "RCTConvert+Mapkit.h"
```

```objectivec title='RCTConvert+Mapkit.h'
#import <MapKit/MapKit.h>
#import <React/RCTConvert.h>
#import <CoreLocation/CoreLocation.h>
#import <React/RCTConvert+CoreLocation.h>

@interface RCTConvert (Mapkit)

+ (MKCoordinateSpan)MKCoordinateSpan:(id)json;
+ (MKCoordinateRegion)MKCoordinateRegion:(id)json;

@end

@implementation RCTConvert(MapKit)

+ (MKCoordinateSpan)MKCoordinateSpan:(id)json
{
  json = [self NSDictionary:json];
  return (MKCoordinateSpan){
    [self CLLocationDegrees:json[@"latitudeDelta"]],
    [self CLLocationDegrees:json[@"longitudeDelta"]]
  };
}

+ (MKCoordinateRegion)MKCoordinateRegion:(id)json
{
  return (MKCoordinateRegion){
    [self CLLocationCoordinate2D:json],
    [self MKCoordinateSpan:json]
  };
}

@end
```

這些轉換函數被設計為能安全處理JS可能傳入的任何JSON數據，當遇到缺失鍵值或其他開發者錯誤時，會顯示「RedBox」錯誤並返回標準初始化值。

要完成對`region`屬性的支持，我們需要在`propTypes`中進行文檔說明：

```tsx title="MapView.tsx"
MapView.propTypes = {
  /**
   * A Boolean value that determines whether the user may use pinch
   * gestures to zoom in and out of the map.
   */
  zoomEnabled: PropTypes.bool,

  /**
   * The region to be displayed by the map.
   *
   * The region is defined by the center coordinates and the span of
   * coordinates to display.
   */
  region: PropTypes.shape({
    /**
     * Coordinates for the center of the map.
     */
    latitude: PropTypes.number.isRequired,
    longitude: PropTypes.number.isRequired,

    /**
     * Distance between the minimum and the maximum latitude/longitude
     * to be displayed.
     */
    latitudeDelta: PropTypes.number.isRequired,
    longitudeDelta: PropTypes.number.isRequired,
  }),
};
```

```tsx title="MyApp.tsx"
render() {
  const region = {
    latitude: 37.48,
    longitude: -122.16,
    latitudeDelta: 0.1,
    longitudeDelta: 0.1,
  };
  return (
    <MapView
      region={region}
      zoomEnabled={false}
      style={{flex: 1}}
    />
  );
}
```

此處您可以看到區域的數據結構在JS文檔中被明確定義。

## 事件處理

現在我們有了能從JS自由控制的原生地圖組件，但該如何處理來自用戶的事件呢？例如雙指縮放或拖動改變可見區域等操作。

目前我們只是從管理器的`-(UIView *)view`方法返回`MKMapView`實例。由於無法直接為`MKMapView`添加新屬性，我們需要創建繼承自`MKMapView`的子類，並在這個子類上添加`onRegionChange`回調：

```objectivec title='RNTMapView.h'
#import <MapKit/MapKit.h>

#import <React/RCTComponent.h>

@interface RNTMapView: MKMapView

@property (nonatomic, copy) RCTBubblingEventBlock onRegionChange;

@end
```

```objectivec title='RNTMapView.m'
#import "RNTMapView.h"

@implementation RNTMapView

@end
```

注意所有`RCTBubblingEventBlock`都必須以`on`為前綴。接著在`RNTMapManager`上聲明事件處理屬性，使其成為所有暴露視圖的委託，並通過從原生視圖調用事件處理塊來將事件轉發到JS。

```objectivec {9,17,31-48} title='RNTMapManager.m'
#import <MapKit/MapKit.h>
#import <React/RCTViewManager.h>

#import "RNTMapView.h"
#import "RCTConvert+Mapkit.h"

@interface RNTMapManager : RCTViewManager <MKMapViewDelegate>
@end

@implementation RNTMapManager

RCT_EXPORT_MODULE()

RCT_EXPORT_VIEW_PROPERTY(zoomEnabled, BOOL)
RCT_EXPORT_VIEW_PROPERTY(onRegionChange, RCTBubblingEventBlock)

RCT_CUSTOM_VIEW_PROPERTY(region, MKCoordinateRegion, MKMapView)
{
  [view setRegion:json ? [RCTConvert MKCoordinateRegion:json] : defaultView.region animated:YES];
}

- (UIView *)view
{
  RNTMapView *map = [RNTMapView new];
  map.delegate = self;
  return map;
}

#pragma mark MKMapViewDelegate

- (void)mapView:(RNTMapView *)mapView regionDidChangeAnimated:(BOOL)animated
{
  if (!mapView.onRegionChange) {
    return;
  }

  MKCoordinateRegion region = mapView.region;
  mapView.onRegionChange(@{
    @"region": @{
      @"latitude": @(region.center.latitude),
      @"longitude": @(region.center.longitude),
      @"latitudeDelta": @(region.span.latitudeDelta),
      @"longitudeDelta": @(region.span.longitudeDelta),
    }
  });
}
@end
```

在委託方法`-mapView:regionDidChangeAnimated:`中，事件處理塊會在對應視圖上被調用並傳入區域數據。調用`onRegionChange`事件處理塊會觸發JS中的同名回調prop。這個回調會接收原始事件對象，我們通常在包裝組件中進一步處理以簡化API：

```tsx title="MapView.tsx"
class MapView extends React.Component {
  _onRegionChange = event => {
    if (!this.props.onRegionChange) {
      return;
    }

    // process raw event...
    this.props.onRegionChange(event.nativeEvent);
  };
  render() {
    return (
      <RNTMap
        {...this.props}
        onRegionChange={this._onRegionChange}
      />
    );
  }
}
MapView.propTypes = {
  /**
   * Callback that is called continuously when the user is dragging the map.
   */
  onRegionChange: PropTypes.func,
  ...
};
```

```tsx title="MyApp.tsx"
class MyApp extends React.Component {
  onRegionChange(event) {
    // Do stuff with event.region.latitude, etc.
  }

  render() {
    const region = {
      latitude: 37.48,
      longitude: -122.16,
      latitudeDelta: 0.1,
      longitudeDelta: 0.1,
    };
    return (
      <MapView
        region={region}
        zoomEnabled={false}
        onRegionChange={this.onRegionChange}
      />
    );
  }
}
```

## 處理多個原生視圖

React Native視圖在視圖樹中可以擁有多個子視圖，例如：

```tsx
<View>
  <MyNativeView />
  <MyNativeView />
  <Button />
</View>
```

在此範例中，`MyNativeView` 類別是 `NativeComponent` 的封裝器，並公開將在 iOS 平台上呼叫的方法。`MyNativeView` 定義於 `MyNativeView.ios.js` 檔案中，包含 `NativeComponent` 的代理方法。

當使用者與元件互動時（例如點擊按鈕），`MyNativeView` 的 `backgroundColor` 會改變。此時 `UIManager` 無法判斷應處理哪個 `MyNativeView` 或改變哪個元件的 `backgroundColor`。以下提供解決此問題的方法：

```tsx
<View>
  <MyNativeView ref={this.myNativeReference} />
  <MyNativeView ref={this.myNativeReference2} />
  <Button
    onPress={() => {
      this.myNativeReference.callNativeMethod();
    }}
  />
</View>
```

現在上述元件持有特定 `MyNativeView` 的參照，使我們能操作具體的 `MyNativeView` 實例。按鈕可控制哪個 `MyNativeView` 應改變其 `backgroundColor`。此範例假設 `callNativeMethod` 會變更 `backgroundColor`。

```tsx title="MyNativeView.ios.tsx"
class MyNativeView extends React.Component {
  callNativeMethod = () => {
    UIManager.dispatchViewManagerCommand(
      ReactNative.findNodeHandle(this),
      UIManager.getViewManagerConfig('RNCMyNativeView').Commands
        .callNativeMethod,
      [],
    );
  };

  render() {
    return <NativeComponent ref={NATIVE_COMPONENT_REF} />;
  }
}
```

`callNativeMethod` 是我們自訂的 iOS 方法，例如透過 `MyNativeView` 公開改變 `backgroundColor` 的功能。此方法使用需要三個參數的 `UIManager.dispatchViewManagerCommand`：

- `(nonnull NSNumber \*)reactTag`  -  react 視圖的識別碼
- `commandID:(NSInteger)commandID`  -  應呼叫的原生方法 ID
- `commandArgs:(NSArray<id> \*)commandArgs`  -  可從 JS 傳遞至原生的方法參數

```objectivec title='RNCMyNativeViewManager.m'
#import <React/RCTViewManager.h>
#import <React/RCTUIManager.h>
#import <React/RCTLog.h>

RCT_EXPORT_METHOD(callNativeMethod:(nonnull NSNumber*) reactTag) {
    [self.bridge.uiManager addUIBlock:^(RCTUIManager *uiManager, NSDictionary<NSNumber *,UIView *> *viewRegistry) {
        NativeView *view = viewRegistry[reactTag];
        if (!view || ![view isKindOfClass:[NativeView class]]) {
            RCTLogError(@"Cannot find NativeView with tag #%@", reactTag);
            return;
        }
        [view callNativeMethod];
    }];

}
```

此處 `callNativeMethod` 定義於 `RNCMyNativeViewManager.m` 檔案，僅包含 `(nonnull NSNumber*) reactTag` 單一參數。這個匯出的函式會透過包含 `viewRegistry` 參數的 `addUIBlock` 找到特定視圖，並根據 `reactTag` 返回對應元件，從而能在正確的元件上呼叫方法。

## 樣式

由於所有原生 react 視圖都是 `UIView` 的子類別，多數樣式屬性都能直接如預期運作。但某些元件（如固定尺寸的 `UIDatePicker`）需要預設樣式。此預設樣式對佈局演算法至關重要，同時也需能在使用元件時覆寫預設樣式。`DatePickerIOS` 的實作方式是將原生元件包裹於具有彈性樣式的額外視圖中，並對內部原生元件採用固定樣式（該樣式透過從原生端傳入的常數生成）：

```tsx title="DatePickerIOS.ios.tsx"
import {UIManager} from 'react-native';
const RCTDatePickerIOSConsts = UIManager.RCTDatePicker.Constants;
...
  render: function() {
    return (
      <View style={this.props.style}>
        <RCTDatePickerIOS
          ref={DATEPICKER}
          style={styles.rkDatePickerIOS}
          ...
        />
      </View>
    );
  }
});

const styles = StyleSheet.create({
  rkDatePickerIOS: {
    height: RCTDatePickerIOSConsts.ComponentHeight,
    width: RCTDatePickerIOSConsts.ComponentWidth,
  },
});
```

`RCTDatePickerIOSConsts` 常數是從原生端匯出的，透過擷取原生元件的實際框架來實現：

```objectivec title='RCTDatePickerManager.m'
- (NSDictionary *)constantsToExport
{
  UIDatePicker *dp = [[UIDatePicker alloc] init];
  [dp layoutIfNeeded];

  return @{
    @"ComponentHeight": @(CGRectGetHeight(dp.frame)),
    @"ComponentWidth": @(CGRectGetWidth(dp.frame)),
    @"DatePickerModes": @{
      @"time": @(UIDatePickerModeTime),
      @"date": @(UIDatePickerModeDate),
      @"datetime": @(UIDatePickerModeDateAndTime),
    }
  };
}
```

本指南涵蓋了橋接自訂原生元件的多個面向，但仍有更多需要考量的細節，例如用於插入與佈局子視圖的自訂鉤子。若需更深入瞭解，請參考[已實作元件的原始碼](https://github.com/facebook/react-native/tree/main/packages/react-native/React/Views)。