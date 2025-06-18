---
title: Better List Views in React Native
author: Spencer Ahrens
authorTitle: Software Engineer at Facebook
authorURL: 'https://github.com/sahrens'
authorImageURL: 'https://avatars1.githubusercontent.com/u/1509831'
authorTwitter: sahrens2012
tags: [engineering]
---

Many of you have started playing with some of our new List components already after our [teaser announcement in the community group](https://www.facebook.com/groups/react.native.community/permalink/921378591331053), but we are officially announcing them today! No more `ListView`s or `DataSource`s, stale rows, ignored bugs, or excessive memory consumption - with the latest React Native March 2017 release candidate (`0.43-rc.1`) you can pick from the new suite of components what best fits your use-case, with great perf and feature sets out of the box:

### [`<FlatList>`](/docs/flatlist)

This is the workhorse component for simple, performant lists. Provide an array of data and a `renderItem` function and you're good to go:

```
<FlatList
  data={[{title: 'Title Text', key: 'item1'}, ...]}
  renderItem={({item}) => <ListItem title={item.title} />}
/>
```

### [`<SectionList>`](/docs/sectionlist)

If you want to render a set of data broken into logical sections, maybe with section headers (e.g. in an alphabetical address book), and potentially with heterogeneous data and rendering (such as a profile view with some buttons followed by a composer, then a photo grid, then a friend grid, and finally a list of stories), this is the way to go.

```
<SectionList
  renderItem={({item}) => <ListItem title={item.title} />}
  renderSectionHeader={({section}) => <H1 title={section.key} />}
  sections={[ // homogeneous rendering between sections
    {data: [...], key: ...},
    {data: [...], key: ...},
    {data: [...], key: ...},
  ]}
/>

<SectionList
  sections={[ // heterogeneous rendering between sections
    {data: [...], key: ..., renderItem: ...},
    {data: [...], key: ..., renderItem: ...},
    {data: [...], key: ..., renderItem: ...},
  ]}
/>
```

### [`<VirtualizedList>`](/docs/virtualizedlist)

The implementation behind the scenes with a more flexible API. Especially handy if your data is not in a plain array (e.g. an immutable list).

## Features

Lists are used in many contexts, so we packed the new components full of features to handle the majority of use cases out of the box:

- Scroll loading (`onEndReached`).
- Pull to refresh (`onRefresh` / `refreshing`).
- [Configurable](https://github.com/facebook/react-native/blob/master/Libraries/CustomComponents/Lists/ViewabilityHelper.js) viewability (VPV) callbacks (`onViewableItemsChanged` / `viewabilityConfig`).
- Horizontal mode (`horizontal`).
- Intelligent item and section separators.
- Multi-column support (`numColumns`)
- `scrollToEnd`, `scrollToIndex`, and `scrollToItem`
- Better Flow typing.

### Some Caveats

- The internal state of item subtrees is not preserved when content scrolls out of the render window. Make sure all your data is captured in the item data or external stores like Flux, Redux, or Relay.

- These components are based on `PureComponent` which means that they will not re-render if `props` remains shallow-equal. Make sure that everything your `renderItem` function depends on directly is passed as a prop that is not `===` after updates, otherwise your UI may not update on changes. This includes the `data` prop and parent component state. For example:

  ```jsx
  <FlatList
    data={this.state.data}
    renderItem={({item}) => (
      <MyItem
        item={item}
        onPress={() =>
          this.setState(oldState => ({
            selected: {
              // New instance breaks `===`
              ...oldState.selected, // copy old data
              [item.key]: !oldState.selected[item.key], // toggle
            },
          }))
        }
        selected={
          !!this.state.selected[item.key] // renderItem depends on state
        }
      />
    )}
    selected={
      // Can be any prop that doesn't collide with existing props
      this.state.selected // A change to selected should re-render FlatList
    }
  />
  ```

- In order to constrain memory and enable smooth scrolling, content is rendered asynchronously offscreen. This means it's possible to scroll faster than the fill rate and momentarily see blank content. This is a tradeoff that can be adjusted to suit the needs of each application, and we are working on improving it behind the scenes.

- By default, these new lists look for a `key` prop on each item and use that for the React key. Alternatively, you can provide a custom `keyExtractor` prop.

## Performance

除了簡化 API 之外，新的列表元件還具有顯著的效能提升，主要特點是無論有多少行，記憶體使用量幾乎保持恆定。這是通過「虛擬化」渲染視窗外的元素來實現的，即完全從元件層次結構中卸載它們，並回收 React 元件佔用的 JS 記憶體，以及陰影樹和 UI 視圖佔用的原生記憶體。這有一個注意點，即內部元件狀態將不會被保留，因此**請確保在元件外部（例如在 Relay、Redux 或 Flux 儲存中）追蹤任何重要的狀態。**

限制渲染視窗也減少了 React 和原生平台需要完成的工作量，例如視圖遍歷。即使你正在渲染一百萬個元素中的最後一個，使用這些新列表也無需遍歷所有元素即可渲染。你甚至可以使用 `scrollToIndex` 直接跳轉到中間，而無需過度渲染。

我們還對排程進行了一些改進，這應該有助於提升應用程式的響應速度。渲染視窗邊緣的項目會以較低優先級且不頻繁地渲染，確保在手勢、動畫或其他互動完成後才進行。

## 進階用法

與 `ListView` 不同，渲染視窗中的所有項目會在 props 變更時重新渲染。通常這沒問題，因為視窗化將項目數量限制為常數，但如果你的項目較複雜，應確保遵循 React 的效能最佳實踐，並在元件中適當使用 `React.PureComponent` 和/或 `shouldComponentUpdate` 來限制遞迴子樹的重新渲染。

如果無需渲染即可計算行高，可以通過提供 `getItemLayout` prop 來改善使用者體驗。這使得使用 `scrollToIndex` 等滾動到特定項目的操作更加流暢，並會提升滾動指示器 UI 的表現，因為內容高度無需渲染即可確定。

如果你有替代資料類型（如不可變列表），`<VirtualizedList>` 是理想的選擇。它接受 `getItem` prop，允許你返回任意索引的項目資料，並且具有更寬鬆的 Flow 類型檢查。

還有許多參數可供調整以應對特殊用例。例如，可以使用 `windowSize` 來權衡記憶體使用與使用者體驗，`maxToRenderPerBatch` 來調整填充率與響應速度，`onEndReachedThreshold` 來控制滾動載入的觸發時機等。

## 未來工作

- 遷移現有介面（最終棄用 `ListView`）。
- 根據需求新增更多功能（歡迎反饋！）。
- 支援黏性區段標頭。
- 更多效能優化。
- 支援帶狀態的功能性項目元件。