## antd-table 虚拟列表
- index.tsx
```js
/**
 * 该文件来源于antd table 虚拟列表例子：
 * https://ant.design/components/table-cn/#components-table-demo-virtual-list
 */
import React, { useState, useRef, useEffect } from 'react';
import { VariableSizeGrid as Grid } from 'react-window';
import ResizeObserver from 'rc-resize-observer';
import classNames from 'classnames';
import { Table, TableColumnType } from 'antd';
import './index.scss';

const ROW_HEIGHT = 54;

interface TableDataSource {
  [p: string]: string | number | unknown;
}

interface CustomTableColumnsType<T> extends TableColumnType<T> {
  dataIndex: string;
}

export const VirtualTable = (props: Parameters<typeof Table>[0]): JSX.Element => {
  const { columns, scroll } = props;
  const [tableWidth, setTableWidth] = useState(0);
  const widthColumnCount = columns.filter(({ width }) => !width).length;

  // TODO: Number(cur.width)限定了传进来的width必须是数字类型，不能写`百分比`或`带px的字符串`，待优化
  const usedWidth = columns.reduce((prev, cur) => prev + Number(cur.width) || 0, 0);

  const mergedColumns = columns.map((column) => {
    // 如果所有列加起来的宽度仍然小于表格总宽度，则按比例分配宽度，用以模拟实现colgroup
    if (usedWidth < tableWidth) {
      return {
        ...column,
        width: tableWidth * (Number(column.width) / usedWidth),
      };
    }

    if (column.width) {
      return column;
    }

    // 未填写width属性，则按列数平均分配宽度
    return {
      ...column,
      width: Math.floor(tableWidth / widthColumnCount),
    };
  });

  const gridRef = useRef(null);

  const resetVirtualGrid = () => {
    gridRef.current.resetAfterIndices({
      columnIndex: 0,
      shouldForceUpdate: true,
    });
  };

  useEffect(() => resetVirtualGrid, [tableWidth, columns]);

  const renderVirtualList = (
    rawData: TableDataSource[],
    {
      scrollbarSize,
      onScroll,
    }: {
      scrollbarSize: number;
      // ref: React.MutableRefObject<{ scrollLeft: number }>;
      onScroll: (info: { currentTarget?: HTMLElement; scrollLeft?: number }) => void;
    },
  ) => {
    const totalHeight = rawData.length * ROW_HEIGHT;

    const renderCell = (
      value: unknown,
      record: TableDataSource,
      index: number,
      curColumn: TableColumnType<TableDataSource>,
    ) => {
      // 若存在render函数，则使用render函数进行渲染
      if (curColumn.render) {
        return curColumn.render(value, record, index);
      }
      return value;
    };

    return (
      <Grid
        ref={gridRef}
        className="virtual-grid"
        columnCount={mergedColumns.length}
        columnWidth={(index: number) => {
          const { width } = mergedColumns[index];
          return totalHeight > scroll.y && index === mergedColumns.length - 1
            ? (width as number) - scrollbarSize - 1
            : (width as number);
        }}
        height={scroll.y as number}
        rowCount={rawData.length}
        rowHeight={() => ROW_HEIGHT}
        width={tableWidth}
        onScroll={({ scrollLeft }: { scrollLeft: number }) => {
          onScroll({ scrollLeft });
        }}
      >
        {({ columnIndex, rowIndex, style }: { columnIndex: number; rowIndex: number; style: React.CSSProperties }) => (
          <div
            className={classNames('virtual-table-cell', {
              'virtual-table-cell-last': columnIndex === mergedColumns.length - 1,
            })}
            style={style}
          >
            {renderCell(
              // 当前单元格的值
              (rawData[rowIndex] as TableDataSource)[
                (mergedColumns as CustomTableColumnsType<TableDataSource>[])[columnIndex].dataIndex
              ],
              rawData[rowIndex] as TableDataSource,
              rowIndex,
              // 当前列的column配置
              (mergedColumns as CustomTableColumnsType<TableDataSource>[])[columnIndex],
            )}
          </div>
        )}
      </Grid>
    );
  };

  return (
    <ResizeObserver
      onResize={({ width }) => {
        setTableWidth(width);
      }}
    >
      <Table
        {...props}
        className="virtual-table"
        columns={mergedColumns}
        pagination={false}
        components={{
          body: renderVirtualList,
        }}
      />
    </ResizeObserver>
  );
};

```

- index.scss
```scss
.virtual-table {
  .ant-table-container:before,
  .ant-table-container:after {
    display: none;
  }

  .virtual-table-cell {
    box-sizing: border-box;
    padding: 16px;
    border-bottom: 1px solid #e8e8e8;
    background: #fff;
    text-overflow: ellipsis;
    white-space: nowrap;
    overflow: hidden;
    border-right: 1px solid #f0f0f0;
    &-last {
      border-right: none;
    }
  }
  .virtual-grid {
    border-right: 1px solid #f0f0f0;
    border-bottom: 1px solid #f0f0f0;
  }
}

```


### Usage
```js
import {VirtalTable} from './virtual-table'

export default (props):JSX.Element => {
  const columns = [];
  const dataSource = [];
  return <VirtualTable
            scroll={{ y: 600, x: '100vw' }}
            loading={props.loading}
            columns={columns}
            dataSource={dataSource}
            bordered
          />
}
```
