---
id: native-components-ios
title: iOS Native UI Components
---

import NativeDeprecated from './the-new-architecture/\_markdown_native_deprecation.mdx'

<NativeDeprecated />

現有大量原生UI元件可供最新應用程式使用——有些是平台內建元件，有些來自第三方函式庫，還有一些可能是您過去開發的專屬元件。React Native已封裝了部分核心平台元件如`ScrollView`和`TextInput`，但並未涵蓋所有元件，特別是您先前為其他應用程式開發的自訂元件。幸運的是，我們可以將這些現有元件封裝起來，使其能無縫整合到React Native應用中。

與原生模組指南類似，本進階指南假設您具備iOS開發基礎知識。我們將透過實作React Native核心庫中現有`MapView`元件的部分功能，示範如何建構原生UI元件。

## iOS MapView範例

假設我們需要在應用中加入互動式地圖——與其從頭開發，不如直接使用[`MKMapView`](https://developer.apple.com/library/prerelease/mac/documentation/MapKit/Reference/MKMapView_Class/index.html)，只需讓它能透過JavaScript呼叫即可。

原生視圖由`RCTViewManager`的子類別創建和管理。這些子類別功能類似視圖控制器，但實際上是單例模式——橋接器只會為每個管理器創建一個實例。它們將原生視圖暴露給`RCTUIManager`，由後者委派設置和更新視圖屬性。`RCTViewManager`通常也作為視圖的代理，透過橋接器將事件傳回JavaScript。

要暴露視圖您需要：

- 繼承`RCTViewManager`創建元件管理器
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
請勿嘗試在透過`-view`方法暴露的`UIView`實例上設置`frame`或`backgroundColor`屬性。
React Native會覆寫自訂類別設置的值以匹配JavaScript元件的佈局屬性。
若需此級別的控制，建議將目標`UIView`實例包裹在另一個`UIView`中並返回包裹後的實例。
詳見[Issue 2948](https://github.com/facebook/react-native/issues/2948)。
:::

:::info
上例中我們使用`RNT`作為類別前綴。前綴用於避免與其他框架命名衝突。
Apple框架使用雙字母前綴，React Native使用`RCT`前綴。建議您使用非`RCT`的三字母前綴命名自訂類別。
:::

接著需要少量JavaScript程式碼使其成為可用的React元件：

```tsx {3} title="MapView.tsx"
import {requireNativeComponent} from 'react-native';

export default requireNativeComponent('RNTMap');
```

`requireNativeComponent`函數會自動將`RNTMap`解析為`RNTMapManager`，並導出原生視圖供JavaScript使用。

```tsx title="MyApp.tsx"
import MapView from './MapView.tsx';

export default function MyApp() {
  return <MapView style={{flex: 1}} />;
}
```

:::note
渲染時請記得拉伸視圖，否則您將只會看到空白畫面。
:::

現在這已是功能完整的原生地圖視圖元件，支援雙指縮放等原生手勢操作。但目前還無法透過JavaScript控制它。

## 屬性

首先可透過橋接原生屬性增強元件功能。假設我們要禁用縮放並指定可見區域。禁用縮放是布林值屬性，只需添加這行程式碼：

```objectivec title='RNTMapManager.m'
RCT_EXPORT_VIEW_PROPERTY(zoomEnabled, BOOL)
```

請注意我們明確將類型指定為`BOOL`——React Native在橋接通信時底層使用`RCTConvert`來轉換各種數據類型，錯誤值會顯示方便的「RedBox」錯誤提示，讓您立即發現問題。當處理如此直觀的情況時，整個實現過程都會由這個宏自動處理完成。

現在要實際禁用縮放功能，我們在JavaScript中設置該屬性：

```tsx {4} title="MyApp.tsx"
import MapView from './MapView.tsx';

export default function MyApp() {
  return <MapView zoomEnabled={false} style={{flex: 1}} />;
}
```

為了記錄MapView元件的屬性（及其接受的數值類型），我們將添加一個包裝元件並使用TypeScript定義介面：

```tsx {6-9} title="MapView.tsx"
import {requireNativeComponent} from 'react-native';

const RNTMap = requireNativeComponent('RNTMap');

export default function MapView(props: {
  /**
   * Whether the user may use pinch gestures to zoom in and out.
   */
  zoomEnabled?: boolean;
}) {
  return <RNTMap {...props} />;
}
```

現在我們有了一個文件完善的包裝元件可供使用。

接著，我們來添加更複雜的`region`屬性。首先添加原生代碼：

```objectivec title='RNTMapManager.m'
RCT_CUSTOM_VIEW_PROPERTY(region, MKCoordinateRegion, MKMapView)
{
  [view setRegion:json ? [RCTConvert MKCoordinateRegion:json] : defaultView.region animated:YES];
}
```

這比之前處理的`BOOL`情況複雜許多。現在需要處理`MKCoordinateRegion`類型的轉換函數，並添加自定義代碼使視圖能在JS設置區域時產生動畫效果。在我們提供的函數體內，`json`參數代表從JS傳入的原始值。`view`變量讓我們能訪問管理器視圖實例，而`defaultView`則用於在JS傳入null哨兵值時將屬性重置為默認值。

您可以為視圖編寫任何轉換函數——這裡是通過`RCTConvert`的擴展類別實現`MKCoordinateRegion`轉換。它利用了ReactNative現有的`RCTConvert+CoreLocation`類別：

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

這些轉換函數設計用於安全處理JS可能傳入的任何JSON數據，當遇到缺失鍵值或其他開發錯誤時，會顯示「RedBox」錯誤並返回標準初始化值。

要完成對`region`屬性的支持，我們可以用TypeScript進行文檔說明：

```tsx {6-25} title="MapView.tsx"
import {requireNativeComponent} from 'react-native';

const RNTMap = requireNativeComponent('RNTMap');

export default function MapView(props: {
  /**
   * The region to be displayed by the map.
   *
   * The region is defined by the center coordinates and the span of
   * coordinates to display.
   */
  region?: {
    /**
     * Coordinates for the center of the map.
     */
    latitude: number;
    longitude: number;

    /**
     * Distance between the minimum and the maximum latitude/longitude
     * to be displayed.
     */
    latitudeDelta: number;
    longitudeDelta: number;
  };
  /**
   * Whether the user may use pinch gestures to zoom in and out.
   */
  zoomEnabled?: boolean;
}) {
  return <RNTMap {...props} />;
}
```

現在我們可以為`MapView`提供`region`屬性：

```tsx {4-9,12} title="MyApp.tsx"
import MapView from './MapView.tsx';

export default function MyApp() {
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

## 事件處理

現在我們有了能從JS自由控制的原生地圖元件，但該如何處理來自用戶的事件，例如縮放或平移改變可見區域？

目前我們只是從管理器的`-(UIView *)view`方法返回`MKMapView`實例。由於無法直接為`MKMapView`添加新屬性，我們需要創建繼承自`MKMapView`的子類作為視圖使用。然後可以在這個子類上添加`onRegionChange`回調：

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

注意所有`RCTBubblingEventBlock`必須以`on`為前綴。接著在`RNTMapManager`上聲明事件處理屬性，使其成為所有暴露視圖的代理，並通過從原生視圖調用事件處理區塊來將事件轉發到JS。

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

在代理方法`-mapView:regionDidChangeAnimated:`中，事件處理區塊會使用區域數據在對應視圖上被調用。調用`onRegionChange`事件處理區塊會觸發JS中相同的回調屬性。這個回調會攜帶原始事件被調用，我們通常在包裝元件中處理這些事件以簡化API：

```tsx {3-10,14-17,19} title="MapView.tsx"
// ...

type RegionChangeEvent = {
  nativeEvent: {
    latitude: number;
    longitude: number;
    latitudeDelta: number;
    longitudeDelta: number;
  };
};

export default function MapView(props: {
  // ...
  /**
   * Callback that is called continuously when the user is dragging the map.
   */
  onRegionChange: (event: RegionChangeEvent) => unknown;
}) {
  return <RNTMap {...props} onRegionChange={onRegionChange} />;
}
```

```tsx {6-9,14} title="MyApp.tsx"
import MapView from './MapView.tsx';

export default function MyApp() {
  // ...

  const onRegionChange = useCallback(event => {
    const {region} = event.nativeEvent;
    // Do something with `region.latitude`, etc.
  });

  return (
    <MapView
      // ...
      onRegionChange={onRegionChange}
    />
  );
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

在此範例中，`MyNativeView`類是`NativeComponent`的包裝器，並暴露將在iOS平台調用的方法。`MyNativeView`定義於`MyNativeView.ios.js`文件中，包含`NativeComponent`的代理方法。

當使用者與元件互動時，例如點擊按鈕，`MyNativeView` 的 `backgroundColor` 會改變。在這種情況下，`UIManager` 無法判斷應該處理哪一個 `MyNativeView` 以及哪一個應該改變 `backgroundColor`。以下將提供解決此問題的方法：

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

現在上述元件已參照特定的 `MyNativeView`，這讓我們能夠使用 `MyNativeView` 的特定實例。現在按鈕可以控制哪一個 `MyNativeView` 應該改變其 `backgroundColor`。在此範例中，我們假設 `callNativeMethod` 會改變 `backgroundColor`。

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

`callNativeMethod` 是我們自訂的 iOS 方法，例如它會透過 `MyNativeView` 公開的介面來改變 `backgroundColor`。此方法使用 `UIManager.dispatchViewManagerCommand`，它需要三個參數：

- `(nonnull NSNumber \*)reactTag`  -  react 視圖的 id。
- `commandID:(NSInteger)commandID`  -  應該被呼叫的原生方法 id。
- `commandArgs:(NSArray<id> \*)commandArgs`  -  我們可以從 JS 傳遞到原生的方法參數。

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

這裡的 `callNativeMethod` 定義在 `RNCMyNativeViewManager.m` 檔案中，並且只包含一個參數 `(nonnull NSNumber*) reactTag`。這個匯出的函數會使用 `addUIBlock` 來找到特定的視圖，`addUIBlock` 包含 `viewRegistry` 參數，並根據 `reactTag` 回傳元件，從而允許它在正確的元件上呼叫方法。

## 樣式

由於我們所有的原生 react 視圖都是 `UIView` 的子類別，大多數樣式屬性都會如預期般直接生效。然而，某些元件會需要預設樣式，例如固定大小的 `UIDatePicker`。這個預設樣式對於佈局演算法正常運作非常重要，但我們也希望在使用元件時能夠覆寫預設樣式。`DatePickerIOS` 透過將原生元件包裝在一個具有彈性樣式的額外視圖中來實現這一點，並在內部原生元件上使用固定樣式（該樣式是透過從原生傳遞的常數生成的）：

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

`RCTDatePickerIOSConsts` 常數是從原生端匯出的，它透過抓取原生元件的實際框架來實現，如下所示：

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

本指南涵蓋了橋接自訂原生元件的許多面向，但還有更多可能需要考慮的內容，例如用於插入和佈局子視圖的自訂鉤子。如果你想更深入瞭解，可以查看一些已實作元件的[原始碼](https://github.com/facebook/react-native/tree/main/packages/react-native/React/Views)。