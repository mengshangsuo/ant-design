---
order: 26
title:
  zh-CN: 可配置表格
  en-US: Configurable table
---

## zh-CN

对表格进行动态配置。

## en-US

A configurable table.

````jsx
import {
  Table,
  Dropdown,
  Button,
  Icon,
  Menu,
  Popover,
  Divider,
  Checkbox,
  InputNumber,
  Tooltip,
} from 'antd';
import { DragDropContext, Droppable, Draggable } from 'react-beautiful-dnd';
import classnames from 'classnames';

const ButtonGroup = Button.Group;

const data = [];
for (let i = 0; i < 100; i++) {
  data.push({
    key: i,
    name: `Edrward ${i}`,
    age: 32,
    address: `London Park no. ${i}`,
  });
}

const menu = (
  <Menu>
    <Menu.Item key="1">Duplicate</Menu.Item>
    <Menu.Item key="3">Delete</Menu.Item>
  </Menu>
);

// a little function to help us with reordering the result
const reorder = (list, startIndex, endIndex) => {
  const result = Array.from(list);
  const [removed] = result.splice(startIndex, 1);
  result.splice(endIndex, 0, removed);

  return result;
};

class ColumnList extends React.Component {
  handleDragEnd = (result) => {
    const { columns, onChange } = this.props;
    // dropped outside the list
    if (!result.destination) {
      return;
    }
    const nextColumns = reorder(
      columns,
      result.source.index,
      result.destination.index
    );
    onChange({ columns: nextColumns });
  }

  handleSelect = key => (e) => {
    let showColumnKeys = this.props.showColumnKeys.slice();
    if (e.target.checked) {
      showColumnKeys.push(key);
    } else {
      showColumnKeys = showColumnKeys.filter(k => k !== key);
    }
    this.props.onChange({ showColumnKeys });
  }

  handleFix = direction => () => {
    let { leftFixed, rightFixed } = this.props;
    if (direction === 'left') {
      leftFixed = !leftFixed;
    } else {
      rightFixed = !rightFixed;
    }
    this.props.onChange({ leftFixed, rightFixed });
  }

  renderItem = (col, index) => {
    const { columns, showColumnKeys, leftFixed, rightFixed } = this.props;
    const canLeftFixed = index === 0;
    const canRightFixed = index === columns.length - 1;
    const fixed = (canLeftFixed && leftFixed) || (canRightFixed && rightFixed);
    return (
      <Draggable key={col.key} draggableId={col.key} index={index}>
        {(provided, snapshot) => (
          <li
            className={classnames({
              dragging: snapshot.isDragging,
            })}
            ref={provided.innerRef}
            {...provided.dragHandleProps}
            {...provided.draggableProps}
            style={provided.draggableProps.style}
          >
            <Checkbox
              checked={showColumnKeys.indexOf(col.key) > -1}
              onChange={this.handleSelect(col.key)}
            />
            <span>{col.title}</span>
            <span className="setting-panel-column-actions">
              {(canLeftFixed || canRightFixed) && (
                <Icon
                  className={classnames('fix-btn', { fixed })}
                  onClick={this.handleFix(canLeftFixed ? 'left' : 'right')}
                  type={fixed ? 'pushpin' : 'pushpin-o'}
                />
              )}
            </span>
          </li>
        )}
      </Draggable>
    );
  }

  render() {
    const { columns } = this.props;
    return (
      <DragDropContext onDragEnd={this.handleDragEnd}>
        <Droppable droppableId="droppable">
          {provided => (
            <ul ref={provided.innerRef}>
              {columns.map(this.renderItem)}
              {provided.placeholder}
            </ul>
          )}
        </Droppable>
      </DragDropContext>
    );
  }
}

class SettingPanel extends React.Component {
  state = {
    columns: this.props.columns,
    leftFixed: this.props.leftFixed,
    rightFixed: this.props.rightFixed,
    pageSize: this.props.pageSize,
    showColumnKeys: this.props.showColumnKeys,
  }

  handleColumnsChange = (state) => {
    this.setState(state);
  }

  handlePageSizeChange = (value) => {
    this.setState({ pageSize: value });
  }

  handleSubmit = () => {
    this.props.onChange(this.state);
  }

  render() {
    const { columns, leftFixed, rightFixed, showColumnKeys, pageSize } = this.state;
    return (
      <div>
        <div className="setting-panel-column">
          <h4>列</h4>
          <ColumnList
            columns={columns}
            leftFixed={leftFixed}
            rightFixed={rightFixed}
            showColumnKeys={showColumnKeys}
            onChange={this.handleColumnsChange}
          />
        </div>
        <Divider />
        <div className="setting-panel-row">
          <h4>行</h4>
          <div>
            每页展示
            <InputNumber
              className="setting-panel-row-count"
              value={pageSize}
              onChange={this.handlePageSizeChange}
            />
            行
          </div>
        </div>
        <Divider />
        <div className="setting-panel-actions">
          <Button type="primary" size="small" onClick={this.handleSubmit}>确定</Button>
          <Button size="small" onClick={this.props.onCancel}>取消</Button>
        </div>
      </div>
    );
  }
}

class Demo extends React.Component {
  columns = [
    { title: 'Full Name', width: 100, dataIndex: 'name', key: 'name' },
    { title: 'Age', width: 100, dataIndex: 'age', key: 'age' },
    { title: 'Column 1', dataIndex: 'address', key: '1', width: 150 },
    { title: 'Column 2', dataIndex: 'address', key: '2', width: 150 },
    { title: 'Column 3', dataIndex: 'address', key: '3', width: 150 },
    { title: 'Column 4', dataIndex: 'address', key: '4', width: 150 },
    { title: 'Column 5', dataIndex: 'address', key: '5', width: 150 },
    { title: 'Column 6', dataIndex: 'address', key: '6', width: 150 },
    { title: 'Column 7', dataIndex: 'address', key: '7', width: 150 },
    { title: 'Column 8', dataIndex: 'address', key: '8' },
    {
      title: 'Action',
      key: 'operation',
      width: 100,
      render: () => <a href="javascript:;">action</a>,
    },
  ];

  state = {
    columns: this.columns,
    leftFixed: true,
    rightFixed: true,
    showColumnKeys: this.columns.map(c => c.key),
    pageSize: 20,
    panelVisible: false,
    fullScreen: false,
  }

  handleSettingChange = (state) => {
    this.setState(state);
    this.handlePanelVisibleChange(false);
  }

  handlePanelVisibleChange = (visible) => {
    this.setState({ panelVisible: visible });
  }

  handleFullScreen = () => {
    this.setState({ fullScreen: !this.state.fullScreen });
  }

  getColumns() {
    const { columns, showColumnKeys, leftFixed, rightFixed } = this.state;
    const finnalColumns = columns.slice();
    if (leftFixed) {
      finnalColumns[0] = {
        ...finnalColumns[0],
        fixed: 'left',
      };
    }
    if (rightFixed) {
      const lastIndex = columns.length - 1;
      finnalColumns[lastIndex] = {
        ...finnalColumns[lastIndex],
        fixed: 'right',
      };
    }
    return finnalColumns.filter(c => showColumnKeys.indexOf(c.key) > -1);
  }

  render() {
    const {
      columns,
      showColumnKeys,
      leftFixed,
      rightFixed,
      pageSize,
      panelVisible,
      fullScreen,
    } = this.state;

    const height = fullScreen ? window.innerHeight - 170 : 300;

    const settingPanel = (
      <SettingPanel
        columns={columns}
        showColumnKeys={showColumnKeys}
        leftFixed={leftFixed}
        rightFixed={rightFixed}
        pageSize={pageSize}
        onChange={this.handleSettingChange}
        onCancel={() => this.handlePanelVisibleChange(false)}
      />
    );

    return (
      <div className={classnames('table-container', { 'full-screen': fullScreen })}>
        <div className="toolbar clearfix">
          <div className="batch-actions">
            <Dropdown overlay={menu}>
              <Button type="primary">
                Actions <Icon type="down" />
              </Button>
            </Dropdown>
          </div>
          {fullScreen ? (
            <div className="actions">
              <button type="button" className="shrink-btn" onClick={this.handleFullScreen}>
                <Icon type="shrink" />
              </button>
            </div>
          ) : (
            <ButtonGroup className="actions">
              <Tooltip title="下载全部结果">
                <Button><Icon type="download" /></Button>
              </Tooltip>
              <Popover
                destroyTooltipOnHide
                visible={panelVisible}
                overlayClassName="setting-panel"
                placement="bottomLeft"
                content={settingPanel}
                onVisibleChange={this.handlePanelVisibleChange}
                trigger="click"
              >
                <Tooltip title="表格设置">
                  <Button><Icon type="setting" /></Button>
                </Tooltip>
              </Popover>
              <Tooltip title="全屏展示">
                <Button onClick={this.handleFullScreen}>
                  <Icon type="arrows-alt" />
                </Button>
              </Tooltip>
            </ButtonGroup>
          )}
        </div>
        <Table
          bordered
          columns={this.getColumns()}
          rowSelection={{}}
          dataSource={data}
          pagination={{ pageSize }}
          scroll={{ x: 1600, y: height }}
        />
      </div>
    );
  }
}

ReactDOM.render(<Demo />, mountNode);
````

````css
#components-table-demo-configurable-table .toolbar {
  margin-bottom: 16px;
}

#components-table-demo-configurable-table .actions {
  float: right;
}

#components-table-demo-configurable-table .batch-actions {
  float: left;
}

.setting-panel .ant-popover-inner-content {
  padding: 12px 0;
}

.setting-panel-column h4,
.setting-panel-row,
.setting-panel-actions {
  padding: 0 12px;
}

.setting-panel-column ul {
  overflow-x: auto;
}

.setting-panel-column ul li {
  padding: 5px 12px;
}

.setting-panel-column ul li:hover,
.setting-panel-column ul li.selected,
.setting-panel-column ul li.dragging {
  background-color: #e6f7ff;
}

.setting-panel-column-actions {
  float: right;
}

.setting-panel-column-actions .fix-btn {
  cursor: pointer;
}

.setting-panel-column-actions .fix-btn:hover,
.setting-panel-column-actions .fix-btn.fixed {
  color: #1890ff;
}

.setting-panel-row-count {
  width: 60px;
  margin-left: 8px;
  margin-right: 8px;
}

.setting-panel-actions button {
  margin-right: 8px;
}

.table-container.full-screen {
  position: fixed;
  z-index: 1000;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  background: #fff;
  padding-bottom: 8px;
}

#components-table-demo-configurable-table .table-container.full-screen .toolbar {
  background-color: #fafafa;
  margin-bottom: 0;
}

#components-table-demo-configurable-table .table-container.full-screen .batch-actions {
  padding: 16px;
}

#components-table-demo-configurable-table .table-container.full-screen .shrink-btn {
  cursor: pointer;
  height: 64px;
  width: 64px;
  border: none;
  background: #fafafa;
  border-left: 1px solid #e8e8e8;
}

````
