---
title: "react-beautiful-dndでカンバンボードをつくる"
emoji: "🪧"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["react", "javascript"]
published: true
---

## サンプル

https://codesandbox.io/s/sleepy-cherry-8hyt6?file=/src/App.tsx

こんな感じです。下記仕様で作ってます

- notSelected と selected の 2 つのカラムを作って、アイテムを移動できる様にする
- 各アイテムはクリックするとクリックイベントが発火する

## 登場人物

- DragDropContext
  - ドラッグ＆ドロップする全体のエリア
  - onDragEnd でドロップした後の処理をセット
- Droppable
  - ドロップする先。サンプルの場合 notSelected とか selected のカラムを指します
- Draggable
  - 1 つ 1 つのアイテムを指します。ドラッグして運べるもの

## 動くものを作る

- 必要最低限、スタイルも何もなしにしようとするとこんな感じでしょうか。
- 上記の登場人物に、ref やら各種 props やらを渡すといい感じのアニメーション付きでサクッと実装できます。いいね。

```jsx
<DragDropContext onDragEnd={handleDragEnd}>
  <Droppable key="selected" droppableId="selected">
    {(provided) => (
      <div ref={provided.innerRef} {...provided.droppableProps}>
        {selected.map((item, i) => (
          <Draggable draggableId={item.id} key={item.id} index={i}>
            {(provided) => (
              <div
                ref={provided.innerRef}
                {...provided.draggableProps}
                {...provided.dragHandleProps}
              >
                {item.name}
              </div>
            )}
          </Draggable>
        ))}
        {provided.placeholder}
      </div>
    )}
  </Droppable>
</DragDropContext>
```

## その他のポイント

- onDragEnd に渡している関数の中身でコアの部分は下記のコードです。
- 抜粋のコードだけでは完全には読み取れないかもしれませんが、要するに、カラムごとに配列を用意して、splice で UI 上の順番の入れ替えを配列に反映しています。
- 条件分岐は、同一カラム内での移動かカラムをまたぐ移動かで分岐しています。

```js
// reorder
if (source.droppableId === destination.droppableId) {
  const _items =
    source.droppableId === "notSelected" ? _notSelected : _Selected;
  const setData =
    source.droppableId === "notSelected" ? setNotSelected : setSelected;
  const [removed] = _items.splice(source.index, 1);
  _items.splice(destination.index, 0, removed);
  setData(_items);
}
// move to other column
else {
  const sourceList =
    source.droppableId === "notSelected" ? _notSelected : _selected;
  const destinationList =
    source.droppableId === "notSelected" ? _selected : _notSelected;
  const [removed] = sourceList.splice(source.index, 1);
  destinationList.splice(destination.index, 0, removed);
  setNotSelected(_notSelected);
  setSelected(_selected);
}
```

- Draggable 同士の間隔の調整は、直下の div でやる必要があります。
- 元々この子要素でスタイリングしていたのですが、それをやると、ドロップした後にガタつきました。

```jsx
<div
  ref={provided.innerRef}
  {...provided.draggableProps}
  {...provided.dragHandleProps}
  style={{
    marginBottom: "8px",
    ...provided.draggableProps.style,
  }}
></div>
```

- ドラッグ&ドロップとクリックイベントを両立するためには、下記の様な条件分岐を使うとできます。
- サンプルでもクリックするとイベントが発火するようになってます。

```js
const handleClick = useCallback((e) => {
  if (e.defaultPrevented) {
    return;
  }
  alert("clicked!!");
}, []);
```
