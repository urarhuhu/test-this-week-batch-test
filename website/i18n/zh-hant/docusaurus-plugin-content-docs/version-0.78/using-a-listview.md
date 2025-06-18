---
id: using-a-listview
title: Using List Views
---

React Native 提供了一系列用於展示數據列表的組件。通常情況下，你會想要使用 [FlatList](flatlist.md) 或 [SectionList](sectionlist.md)。

`FlatList` 組件用於顯示一個可滾動的、結構相似但內容變化的數據列表。`FlatList` 非常適合用於長列表數據，其中項目的數量可能會隨時間變化。與更通用的 [`ScrollView`](using-a-scrollview.md) 不同，`FlatList` 只會渲染當前顯示在屏幕上的元素，而不是一次性渲染所有元素。

`FlatList` 組件需要兩個屬性：`data` 和 `renderItem`。`data` 是列表的數據來源。`renderItem` 從數據源中取出一項並返回一個格式化後的組件進行渲染。

這個例子創建了一個基於硬編碼數據的基本 `FlatList`。`data` 屬性中的每個項目都被渲染為一個 `Text` 組件。`FlatListBasics` 組件隨後渲染 `FlatList` 和所有的 `Text` 組件。

```SnackPlayer name=FlatList%20Basics
import React from 'react';
import {FlatList, StyleSheet, Text, View} from 'react-native';

const styles = StyleSheet.create({
  container: {
    flex: 1,
    paddingTop: 22,
  },
  item: {
    padding: 10,
    fontSize: 18,
    height: 44,
  },
});

const FlatListBasics = () => {
  return (
    <View style={styles.container}>
      <FlatList
        data={[
          {key: 'Devin'},
          {key: 'Dan'},
          {key: 'Dominic'},
          {key: 'Jackson'},
          {key: 'James'},
          {key: 'Joel'},
          {key: 'John'},
          {key: 'Jillian'},
          {key: 'Jimmy'},
          {key: 'Julie'},
        ]}
        renderItem={({item}) => <Text style={styles.item}>{item.key}</Text>}
      />
    </View>
  );
};

export default FlatListBasics;
```

如果你想渲染一組按邏輯分段的數據，可能還需要分段標題，類似於 iOS 上的 `UITableView`，那麼 [SectionList](sectionlist.md) 就是你的選擇。

```SnackPlayer name=SectionList%20Basics
import React from 'react';
import {SectionList, StyleSheet, Text, View} from 'react-native';

const styles = StyleSheet.create({
  container: {
    flex: 1,
    paddingTop: 22,
  },
  sectionHeader: {
    paddingTop: 2,
    paddingLeft: 10,
    paddingRight: 10,
    paddingBottom: 2,
    fontSize: 14,
    fontWeight: 'bold',
    backgroundColor: 'rgba(247,247,247,1.0)',
  },
  item: {
    padding: 10,
    fontSize: 18,
    height: 44,
  },
});

const SectionListBasics = () => {
  return (
    <View style={styles.container}>
      <SectionList
        sections={[
          {title: 'D', data: ['Devin', 'Dan', 'Dominic']},
          {
            title: 'J',
            data: [
              'Jackson',
              'James',
              'Jillian',
              'Jimmy',
              'Joel',
              'John',
              'Julie',
            ],
          },
        ]}
        renderItem={({item}) => <Text style={styles.item}>{item}</Text>}
        renderSectionHeader={({section}) => (
          <Text style={styles.sectionHeader}>{section.title}</Text>
        )}
        keyExtractor={item => `basicListEntry-${item}`}
      />
    </View>
  );
};

export default SectionListBasics;
```

列表視圖最常見的用途之一是顯示從服務器獲取的數據。要做到這一點，你需要 [學習 React Native 中的網絡請求](network.md)。