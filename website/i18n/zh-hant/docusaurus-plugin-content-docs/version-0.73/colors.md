---
id: colors
title: Color Reference
---

React Native 中的元件[使用 JavaScript 進行樣式設定](style)。顏色屬性通常與[網頁上的 CSS 運作方式](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value)相符。各平台顏色使用的一般指南如下：

- [Android](https://material.io/design/color/color-usage.html)
- [iOS](https://developer.apple.com/design/human-interface-guidelines/ios/visual-design/color/)

## 顏色 API

React Native 提供了多種顏色 API，旨在讓您充分利用平台的設計和使用者偏好。

- [PlatformColor](platformcolor) 讓您可以參考平台的顏色系統。
- [DynamicColorIOS](dynamiccolorios) 是 iOS 專用的，允許您指定在淺色或深色模式下應使用的顏色。

## 顏色表示法

### 紅綠藍 (RGB)

React Native 支援十六進制和函數表示法的 `rgb()` 和 `rgba()`：

- `'#f0f'` (#rgb)
- `'#ff00ff'` (#rrggbb)
- `'#f0ff'` (#rgba)
- `'#ff00ff00'` (#rrggbbaa)
- `'rgb(255, 0, 255)'`
- `'rgb(255 0 255)'`
- `'rgba(255, 0, 255, 1.0)'`
- `'rgba(255 0 255 / 1.0)'`

### 色相飽和度亮度 (HSL)

React Native 支援函數表示法的 `hsl()` 和 `hsla()`：

- `'hsl(360, 100%, 100%)'`
- `'hsl(360 100% 100%)'`
- `'hsla(360, 100%, 100%, 1.0)'`
- `'hsla(360 100% 100% / 1.0)'`

### 色相白度黑度 (HWB)

React Native 支援函數表示法的 `hwb()`：

- `'hwb(0, 0%, 100%)'`
- `'hwb(360, 100%, 100%)'`
- `'hwb(0 0% 0%)'`
- `'hwb(70 50% 0%)'`

### 顏色整數

React Native 也支援以 `int` 值表示的顏色（RGB 色彩模式）：

- `0xff00ff00` (0xrrggbbaa)

:::caution
這可能看起來類似於 Android 的 [Color](https://developer.android.com/reference/android/graphics/Color) 整數表示法，但在 Android 上，值是以 SRGB 色彩模式儲存的 (0xaarrggbb)。
:::

### 命名顏色

在 React Native 中，您也可以使用顏色名稱字串作為值。

:::info
React Native 僅支援小寫的顏色名稱。大寫的顏色名稱不受支援。
:::

#### `transparent`

這是 `rgba(0,0,0,0)` 的快捷方式，與 [CSS3](https://www.w3.org/TR/css-color-3/#transparent) 中的相同。

#### 顏色關鍵字

命名顏色的實現遵循 [CSS3/SVG 規範](https://www.w3.org/TR/css-color-3/#svg-color)：

<!-- alex ignore black white -->

- <ins style={{background: '#f0f8ff'}} className="color-box" /> 愛麗絲藍 (<code>#f0f8ff</code>)
- <ins style={{background: '#faebd7'}} className="color-box" /> 古董白 (<code>#faebd7</code>)
- <ins style={{background: '#00ffff'}} className="color-box" /> 水色 (<code>#00ffff</code>)
- <ins style={{background: '#7fffd4'}} className="color-box" /> 碧綠色 (<code>#7fffd4</code>)
- <ins style={{background: '#f0ffff'}} className="color-box" /> 天藍 (<code>#f0ffff</code>)
- <ins style={{background: '#f5f5dc'}} className="color-box" /> 米色 (<code>#f5f5dc</code>)
- <ins style={{background: '#ffe4c4'}} className="color-box" /> 陶坯黃 (<code>#ffe4c4</code>)
- <ins style={{background: '#000000'}} className="color-box" /> 黑色 (<code>#000000</code>)
- <ins style={{background: '#ffebcd'}} className="color-box" /> 杏仁白 (<code>#ffebcd</code>)
- <ins style={{background: '#0000ff'}} className="color-box" /> 藍色 (<code>#0000ff</code>)
- <ins style={{background: '#8a2be2'}} className="color-box" /> 藍紫 (<code>#8a2be2</code>)
- <ins style={{background: '#a52a2a'}} className="color-box" /> 棕色 (<code>#a52a2a</code>)
- <ins style={{background: '#deb887'}} className="color-box" /> 原木色 (<code>#deb887</code>)
- <ins style={{background: '#5f9ea0'}} className="color-box" /> 軍服藍 (<code>#5f9ea0</code>)
- <ins style={{background: '#7fff00'}} className="color-box" /> 查特酒綠 (<code>#7fff00</code>)
- <ins style={{background: '#d2691e'}} className="color-box" /> 巧克力色 (<code>#d2691e</code>)
- <ins style={{background: '#ff7f50'}} className="color-box" /> 珊瑚色 (<code>#ff7f50</code>)
- <ins style={{background: '#6495ed'}} className="color-box" /> 矢車菊藍 (<code>#6495ed</code>)
- <ins style={{background: '#fff8dc'}} className="color-box" /> 玉米絲色 (<code>#fff8dc</code>)
- <ins style={{background: '#dc143c'}} className="color-box" /> 緋紅 (<code>#dc143c</code>)
- <ins style={{background: '#00ffff'}} className="color-box" /> 青色 (<code>#00ffff</code>)
- <ins style={{background: '#00008b'}} className="color-box" /> 深藍 (<code>#00008b</code>)
- <ins style={{background: '#008b8b'}} className="color-box" /> 深青 (<code>#008b8b</code>)
- <ins style={{background: '#b8860b'}} className="color-box" /> 暗金 (<code>#b8860b</code>)
- <ins style={{background: '#a9a9a9'}} className="color-box" /> 深灰 (<code>#a9a9a9</code>)
- <ins style={{background: '#006400'}} className="color-box" /> 深綠 (<code>#006400</code>)
- <ins style={{background: '#a9a9a9'}} className="color-box" /> 暗灰 (<code>#a9a9a9</code>)
- <ins style={{background: '#bdb76b'}} className="color-box" /> 暗卡其 (<code>#bdb76b</code>)
- <ins style={{background: '#8b008b'}} className="color-box" /> 深洋紅 (<code>#8b008b</code>)
- <ins style={{background: '#556b2f'}} className="color-box" /> 暗橄欖綠 (<code>#556b2f</code>)
- <ins style={{background: '#ff8c00'}} className="color-box" /> 深橙 (<code>#ff8c00</code>)
- <ins style={{background: '#9932cc'}} className="color-box" /> 暗蘭 (<code>#9932cc</code>)
- <ins style={{background: '#8b0000'}} className="color-box" /> 深紅 (<code>#8b0000</code>)
- <ins style={{background: '#e9967a'}} className="color-box" /> 暗鮭紅 (<code>#e9967a</code>)
- <ins style={{background: '#8fbc8f'}} className="color-box" /> 暗海綠 (<code>#8fbc8f</code>)
- <ins style={{background: '#483d8b'}} className="color-box" /> 暗岩藍 (<code>#483d8b</code>)
- <ins style={{background: '#2f4f4f'}} className="color-box" /> 暗岩灰 (<code>#2f4f4f</code>)
- <ins style={{background: '#00ced1'}} className="color-box" /> 暗土耳其玉 (<code>#00ced1</code>)
- <ins style={{background: '#9400d3'}} className="color-box" /> 暗紫 (<code>#9400d3</code>)
- <ins style={{background: '#ff1493'}} className="color-box" /> 深粉紅 (<code>#ff1493</code>)
- <ins style={{background: '#00bfff'}} className="color-box" /> 深天藍 (<code>#00bfff</code>)
- <ins style={{background: '#696969'}} className="color-box" /> 昏灰 (<code>#696969</code>)
- <ins style={{background: '#696969'}} className="color-box" /> 暗灰 (<code>#696969</code>)
- <ins style={{background: '#1e90ff'}} className="color-box" /> 道奇藍 (<code>#1e90ff</code>)
- <ins style={{background: '#b22222'}} className="color-box" /> 耐火磚紅 (<code>#b22222</code>)
- <ins style={{background: '#fffaf0'}} className="color-box" /> 花白 (<code>#fffaf0</code>)
- <ins style={{background: '#228b22'}} className="color-box" /> 森林綠 (<code>#228b22</code>)
- <ins style={{background: '#ff00ff'}} className="color-box" /> 紫紅 (<code>#ff00ff</code>)
- <ins style={{background: '#dcdcdc'}} className="color-box" /> 庚斯博羅灰 (<code>#dcdcdc</code>)
- <ins style={{background: '#f8f8ff'}} className="color-box" /> 幽靈白 (<code>#f8f8ff</code>)
- <ins style={{background: '#ffd700'}} className="color-box" /> 金色 (<code>#ffd700</code>)
- <ins style={{background: '#daa520'}} className="color-box" /> 金菊 (<code>#daa520</code>)
- <ins style={{background: '#808080'}} className="color-box" /> 灰色 (<code>#808080</code>)
- <ins style={{background: '#008000'}} className="color-box" /> 綠色 (<code>#008000</code>)
- <ins style={{background: '#adff2f'}} className="color-box" /> 綠黃 (<code>#adff2f</code>)
- <ins style={{background: '#808080'}} className="color-box" /> 灰 (<code>#808080</code>)
- <ins style={{background: '#f0fff0'}} className="color-box" /> 蜜瓜綠 (<code>#f0fff0</code>)
- <ins style={{background: '#ff69b4'}} className="color-box" /> 亮粉紅 (<code>#ff69b4</code>)
- <ins style={{background: '#cd5c5c'}} className="color-box" /> 印度紅 (<code>#cd5c5c</code>)
- <ins style={{background: '#4b0082'}} className="color-box" /> 靛青 (<code>#4b0082</code>)
- <ins style={{background: '#fffff0'}} className="color-box" /> 象牙白 (<code>#fffff0</code>)
- <ins style={{background: '#f0e68c'}} className="color-box" /> 卡其色 (<code>#f0e68c</code>)
- <ins style={{background: '#e6e6fa'}} className="color-box" /> 薰衣草紫 (<code>#e6e6fa</code>)
- <ins style={{background: '#fff0f5'}} className="color-box" /> 薰衣草腮紅 (<code>#fff0f5</code>)
- <ins style={{background: '#7cfc00'}} className="color-box" /> 草坪綠 (<code>#7cfc00</code>)
- <ins style={{background: '#fffacd'}} className="color-box" /> 檸檬綢 (<code>#fffacd</code>)
- <ins style={{background: '#add8e6'}} className="color-box" /> 淺藍 (<code>#add8e6</code>)
- <ins style={{background: '#f08080'}} className="color-box" /> 亮珊瑚 (<code>#f08080</code>)
- <ins style={{background: '#e0ffff'}} className="color-box" /> 淺青 (<code>#e0ffff</code>)
- <ins style={{background: '#fafad2'}} className="color-box" /> 亮金菊黃 (<code>#fafad2</code>)
- <ins style={{background: '#d3d3d3'}} className="color-box" /> 亮灰 (<code>#d3d3d3</code>)
- <ins style={{background: '#90ee90'}} className="color-box" /> 亮綠 (<code>#90ee90</code>)
- <ins style={{background: '#d3d3d3'}} className="color-box" /> 淺灰 (<code>#d3d3d3</code>)
- <ins style={{background: '#ffb6c1'}} className="color-box" /> 亮粉 (<code>#ffb6c1</code>)
- <ins style={{background: '#ffa07a'}} className="color-box" /> 亮鮭紅 (<code>#ffa07a</code>)
- <ins style={{background: '#20b2aa'}} className="color-box" /> 亮海綠 (<code>#20b2aa</code>)
- <ins style={{background: '#87cefa'}} className="color-box" /> 亮天藍 (<code>#87cefa</code>)
- <ins style={{background: '#778899'}} className="color-box" /> 亮岩灰 (<code>#778899</code>)
- <ins style={{background: '#b0c4de'}} className="color-box" /> 亮鋼藍 (<code>#b0c4de</code>)
- <ins style={{background: '#ffffe0'}} className="color-box" /> 亮黃 (<code>#ffffe0</code>)
- <ins style={{background: '#00ff00'}} className="color-box" /> 酸橙綠 (<code>#00ff00</code>)
- <ins style={{background: '#32cd32'}} className="color-box" /> 酸橙綠 (<code>#32cd32</code>)
- <ins style={{background: '#faf0e6'}} className="color-box" /> 亞麻色 (<code>#faf0e6</code>)
- <ins style={{background: '#ff00ff'}} className="color-box" /> 品紅 (<code>#ff00ff</code>)
- <ins style={{background: '#800000'}} className="color-box" /> 栗色 (<code>#800000</code>)
- <ins style={{background: '#66cdaa'}} className="color-box" /> 中碧綠 (<code>#66cdaa</code>)
- <ins style={{background: '#0000cd'}} className="color-box" /> 中藍 (<code>#0000cd</code>)
- <ins style={{background: '#ba55d3'}} className="color-box" /> 中蘭 (<code>#ba55d3</code>)
- <ins style={{background: '#9370db'}} className="color-box" /> 中紫 (<code>#9370db</code>)
- <ins style={{background: '#3cb371'}} className="color-box" /> 中海綠 (<code>#3cb371</code>)
- <ins style={{background: '#7b68ee'}} className="color-box" /> 中岩藍 (<code>#7b68ee</code>)
- <ins style={{background: '#00fa9a'}} className="color-box" /> 中春綠 (<code>#00fa9a</code>)
- <ins style={{background: '#48d1cc'}} className="color-box" /> 中土耳其玉 (<code>#48d1cc</code>)
- <ins style={{background: '#c71585'}} className="color-box" /> 中紫紅 (<code>#c71585</code>)
- <ins style={{background: '#191970'}} className="color-box" /> 午夜藍 (<code>#191970</code>)
- <ins style={{background: '#f5fffa'}} className="color-box" /> 薄荷乳白 (<code>#f5fffa</code>)
- <ins style={{background: '#ffe4e1'}} className="color-box" /> 霧玫瑰 (<code>#ffe4e1</code>)
- <ins style={{background: '#ffe4b5'}} className="color-box" /> 鹿皮色 (<code>#ffe4b5</code>)
- <ins style={{background: '#ffdead'}} className="color-box" /> 納瓦白 (<code>#ffdead</code>)
- <ins style={{background: '#000080'}} className="color-box" /> 海軍藍 (<code>#000080</code>)
- <ins style={{background: '#fdf5e6'}} className="color-box" /> 舊蕾絲 (<code>#fdf5e6</code>)
- <ins style={{background: '#808000'}} className="color-box" /> 橄欖色 (<code>#808000</code>)
- <ins style={{background: '#6b8e23'}} className="color-box" /> 橄欖褐 (<code>#6b8e23</code>)
- <ins style={{background: '#ffa500'}} className="color-box" /> 橙色 (<code>#ffa500</code>)
- <ins style={{background: '#ff4500'}} className="color-box" /> 橙紅 (<code>#ff4500</code>)
- <ins style={{background: '#da70d6'}} className="color-box" /> 蘭花色 (<code>#da70d6</code>)
- <ins style={{background: '#eee8aa'}} className="color-box" /> 蒼金菊 (<code>#eee8aa</code>)
- <ins style={{background: '#98fb98'}} className="color-box" /> 蒼綠 (<code>#98fb98</code>)
- <ins style={{background: '#afeeee'}} className="color-box" /> 蒼土耳其玉 (<code>#afeeee</code>)
- <ins style={{background: '#db7093'}} className="color-box" /> 蒼紫紅 (<code>#db7093</code>)
- <ins style={{background: '#ffefd5'}} className="color-box" /> 木瓜橙 (<code>#ffefd5</code>)
- <ins style={{background: '#ffdab9'}} className="color-box" /> 桃色 (<code>#ffdab9</code>)
- <ins style={{background: '#cd853f'}} className="color-box" /> 秘魯色 (<code>#cd853f</code>)
- <ins style={{background: '#ffc0cb'}} className="color-box" /> 粉紅色 (<code>#ffc0cb</code>)
- <ins style={{background: '#dda0dd'}} className="color-box" /> 梅紅色 (<code>#dda0dd</code>)
- <ins style={{background: '#b0e0e6'}} className="color-box" /> 粉藍 (<code>#b0e0e6</code>)
- <ins style={{background: '#800080'}} className="color-box" /> 紫色 (<code>#800080</code>)
- <ins style={{background: '#663399'}} className="color-box" /> 麗貝