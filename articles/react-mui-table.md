---
title: "React+Material UIのテーブルプラグイン"
emoji: "📊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["react", "materialui", "javascript"]
published: true
---

## 経緯

- React+Material UI をベースに進んでいるプロジェクトで、下記のようなテーブルを作るということになり、プラグインを探してみました。
  - 行ごとにチェックボックスがついていて、選択できる
  - またヘッダーをチェックするとテーブル全体の選択状態がトグルされる
  - 列の幅が可変
  - 列を fixed できる
  - ページングできる
  - セルの中身をカスタムできる(アイコンを入れたりできる)

## 候補になったもの

https://devexpress.github.io/devextreme-reactive/react/grid/docs/guides/getting-started/
https://react-table.tanstack.com/
https://mui.com/components/data-grid/

## どうだったか

### React Grid

- 採用しました。要件を比較的簡単に実装することができました
- ただ、スタイルを整えるのはなかなか大変でした
- 2022/2/10追記 コメント頂いた通り。商用利用は有料とのこと。

### React Table

- 1 番スター数が多かったですが、下記がネックでした
  - チェックボックスの実装は自力でやる必要があるっぽい
  - fixed にしようとすると、また[別のプラグイン](https://github.com/GuillaumeJasmin/react-table-hoc-fixed-columns)を使う必要がある
  - React Grid と比較すると行数がとっても多くなってしまう
  - [この例](https://codesandbox.io/s/github/tannerlinsley/react-table/tree/master/examples/column-resizing)の通り、resize をしようとすると、table タグを使うことができない

### DataGrid

- 課金しないと要件が実現できなそうだったので採用せず

## 実装内容

ざっくり再現するとこんな感じです
https://codesandbox.io/s/distracted-mirzakhani-krrxc?file=/src/App.js

下記のように、プラグインのタグをどんどん置いていけばそれっぽいテーブルができてしまいました

```jsx
<Grid rows={rows} columns={gridColumns}>
  <PagingState defaultCurrentPage={0} defaultPageSize={10} />
  <DataTypeProvider
    key="label"
    for={["label"]}
    formatterComponent={({ value }) => (
      <Box textAlign="center" sx={{ backgroundColor: "lightBlue" }}>
        {value}
      </Box>
    )}
  />

  <SelectionState selection={selection} onSelectionChange={setSelection} />
  <IntegratedPaging />
  <IntegratedSelection />
  <Table />
  <TableColumnResizing
    columnWidths={columnWidths}
    onColumnWidthsChange={setColumnWidths}
  />
  <TableHeaderRow />
  <TableSelection showSelectAll />
  <TableFixedColumns leftColumns={[TableSelection.COLUMN_TYPE, "name"]} />
  <PagingPanel />
</Grid>
```
