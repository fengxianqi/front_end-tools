## antd-table 表格列拖拽

- index.tsx
```js
import React, { MouseEvent, useState, useRef } from 'react';

import './index.scss';

export interface WithDragColumnProps {
  children: JSX.Element;
  onDragEnd: (fromIndex: number, toIndex: number) => void;
  lineClassName?: string;
  targetNode?: string;
}

export const WithDragColumn = (props: WithDragColumnProps): JSX.Element => {
  const { targetNode = 'th' } = props;

  const dragRef = useRef(null);
  let fromIndex = -1;
  let toIndex = -1;

  const [targetNodeHeight, setTargetNodeHeight] = useState(0);

  const [showDragLine, setShowDragLine] = useState(false);
  const [dragLineOffset, setDragLineOffset] = useState<string | number>(0);

  // const refreshDragLine = () => {

  // }

  const onDragEnter = (e: DragEvent) => {
    const target: HTMLElement = (e.target as HTMLElement).closest(targetNode);
    if (target) {
      const { parentNode } = target;
      const index = Array.from(parentNode.children).indexOf(target);
      toIndex = index;

      const { offsetLeft } = target;
      console.log(offsetLeft);
      setDragLineOffset(`${offsetLeft}px`);

      // refreshDragLine();
    }
  };

  const onDragStart = (e: DragEvent) => {
    const target = (e.target as HTMLElement).closest(targetNode);
    const eventData = e;
    if (target) {
      const { parentNode } = target;
      eventData.dataTransfer.setData('Text', '');
      eventData.dataTransfer.effectAllowed = 'move';
      (parentNode as HTMLElement).ondragenter = onDragEnter;
      (parentNode as HTMLElement).ondragover = function (ev) {
        ev.preventDefault();
        return true;
      };
      const index = Array.from(parentNode.children).indexOf(target);
      fromIndex = index;
      setShowDragLine(true);
    }
  };

  const onDragEnd = (e: DragEvent): void => {
    const target: HTMLElement = (e.target as HTMLElement).closest(targetNode);

    if (target) {
      target.removeAttribute('draggable');
      target.ondragstart = null;
      target.ondragend = null;
      const { parentNode } = target;
      (parentNode as HTMLElement).ondragenter = null;
      (parentNode as HTMLElement).ondragover = null;
      if (fromIndex >= 0 && fromIndex !== toIndex) {
        props.onDragEnd(fromIndex, toIndex);
      }
    }
    setShowDragLine(false);
    fromIndex = -1;
    toIndex = -1;
  };

  const onMouseDown = (event: MouseEvent) => {
    const target: HTMLElement = (event.target as HTMLElement).closest(targetNode);

    if (!target) {
      console.log('not found <${targetNode}> element');
      return;
    }

    if (!targetNodeHeight) {
      // 获取拖拽虚线的高度
      const { height } = target.getBoundingClientRect();
      setTargetNodeHeight(height);
    }

    target.setAttribute('draggable', 'true');
    target.ondragstart = onDragStart;
    target.ondragend = onDragEnd;
  };
  return (
    <div role="presentation" className="with-drag-column" onMouseDown={onMouseDown} ref={dragRef}>
      <div
        className="with-drag-column-line"
        style={{ left: dragLineOffset, display: showDragLine ? 'block' : 'none', height: `${targetNodeHeight}px` }}
      ></div>
      {props.children}
    </div>
  );
};

```
- index.scss
```scss
.with-drag-column {
  position: relative;
  &-line {
    position: absolute;
    top: 0;
    left: 0;
    width: 2px;
    z-index: 9999;
    // height: 100%;
    border-right: dashed 2px red;
  }
  table th.ant-table-cell.ant-table-column-has-sorters {
    cursor: move;
  }
}

```

## Usage

```
import { WithDragColumn } from '@/components/with-drag-column';

  const onDragEnd = (fromIndex: number, toIndex: number) => {
    const newCols = [...columns];
    const item = newCols.splice(fromIndex, 1)[0];
    newCols.splice(toIndex, 0, item);
    setColumns(newCols);
  };
  
 <WithDragColumn onDragEnd={onDragEnd}>
        <Table
          rowKey={(_, i) => i}
          scroll={{ y: 800 }}
          sticky={true}
          loading={props.loading}
          columns={columns}
          dataSource={dataSource}
          bordered
          pagination={{ defaultPageSize: 50, hideOnSinglePage: true }}
        />
      </WithDragColumn>
```
