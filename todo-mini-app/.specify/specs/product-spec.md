# SimpleTodo MiniProgram 产品规格说明书

**版本**: 1.0.0  
**创建日期**: 2026-03-11  
**状态**: 已批准  
**关联宪法**: `.specify/memory/constitution.md`

---

## 📋 目录

1. [全局要求](#全局要求)
2. [页面规格](#页面规格)
3. [数据模型](#数据模型)
4. [交互流程](#交互流程)

---

## 全局要求

### 导航结构

- 使用 tabBar 底部导航
- 两个页面：
  - `/pages/reminders/index` → "提醒"
  - `/pages/mine/index` → "我的"

### 数据持久化

- 使用 `wx.setStorageSync` / `wx.getStorageSync`
- 存储键名：`'reminders'`
- 数据格式：JSON 数组
- 存储限制：10MB

---

## 页面规格

### 页面 1：提醒（Reminders）

**路径**: `/pages/reminders/index`

#### UI 元素

| 元素 | 类型 | 说明 |
|------|------|------|
| 导航栏 | NavBar | 标题："提醒" |
| 浮动按钮 | FAB | 右上角"+"按钮，固定定位 |
| 列表区域 | ScrollView | 垂直滚动，展示所有提醒项 |
| 空状态 | Empty | 无数据时显示"暂无提醒" |

#### 列表项结构

```
┌─────────────────────────────────────┐
│ 提醒内容（左对齐）                    │
│                          提醒时间    │
└─────────────────────────────────────┘
```

- 提醒内容：文本，左对齐
- 提醒时间：格式 `YYYY-MM-DD HH:mm`，右对齐

#### 交互逻辑

| 操作 | 响应 |
|------|------|
| 点击"+"按钮 | 跳转至 `/pages/reminders/create` |
| 点击列表项 | （预留：切换完成状态） |
| 长按列表项 | （预留：删除功能） |

---

### 页面 2：创建提醒（Create Reminder）

**路径**: `/pages/reminders/create`

#### UI 元素

| 元素 | 类型 | 说明 |
|------|------|------|
| 输入框 | Textarea | placeholder="请输入提醒事项"，最大 200 字 |
| 日期选择器 | Picker (date) | 选择提醒日期 |
| 时间选择器 | Picker (time) | 选择提醒时间 |
| 保存按钮 | Button | 底部固定，校验通过后可用 |

#### 校验规则

| 字段 | 规则 | 错误提示 |
|------|------|----------|
| 内容 | 非空，≤200 字 | "请输入提醒事项" |
| 日期 | 已选择 | "请选择日期" |
| 时间 | 已选择 | "请选择时间" |

#### 保存逻辑

```
1. 校验所有字段
   ↓
2. 生成 Reminder 对象：
   {
     id: Date.now().toString(),
     content: 输入内容，
     reminderTime: "YYYY-MM-DD HH:mm",
     createdAt: new Date().toLocaleString(),
     completed: false
   }
   ↓
3. 读取现有列表 → 添加新项 → 排序 → 存入 storage
   ↓
4. wx.navigateBack() 返回上一页
   ↓
5. 列表页 onShow() 刷新数据
```

---

### 页面 3：我的（Mine）

**路径**: `/pages/mine/index`

#### UI 元素

| 元素 | 类型 | 说明 |
|------|------|------|
| 头像 | Image | 圆形，居中，80x80rpx |
| 昵称 | Text | 头像下方，居中 |
| 分隔线 | Divider | 灰色，1rpx |
| 使用说明 | Collapse | 默认展开 |
| 常见问题 | Collapse | 默认收起 |

#### 用户信息获取

```javascript
// 使用 open-type="getUserInfo" 按钮
<button open-type="getUserInfo" bindgetuserinfo="getUserProfile">
  获取用户信息
</button>

// 授权成功
setData({ userInfo: res.userInfo })

// 未授权
setData({ userInfo: null }) // 显示默认头像和"未获取用户信息"
```

#### 使用说明内容

```
1. 点击"提醒"页右上角 + 号创建新事项
2. 所有数据仅保存在本机，更换设备将丢失
3. 目前不支持修改或删除提醒
```

#### 常见问题内容

| 问题 | 答案 |
|------|------|
| 为什么没有提醒通知？ | 当前版本仅记录事项，不支持系统级通知。 |
| 数据会同步到云端吗？ | 不会，所有数据仅存储在当前设备。 |

---

## 数据模型

### Reminder 接口

```typescript
interface Reminder {
  id: string;              // 时间戳生成，唯一标识
  content: string;         // 提醒内容，≤200 字
  reminderTime: string;    // 格式：YYYY-MM-DD HH:mm
  createdAt: string;       // 创建时间，LocaleString
  completed: boolean;      // 完成状态，默认 false
}
```

### 存储结构

```typescript
// wx.getStorageSync('reminders') 返回：
Reminder[] = [
  {
    id: "1710144000000",
    content: "明天上午 10 点开会",
    reminderTime: "2026-03-12 10:00",
    createdAt: "2026-03-11 14:30:00",
    completed: false
  },
  // ...
]
```

### 排序规则

- 按 `reminderTime` 升序排序
- 未到期的提醒在前
- 已到期的提醒在后

---

## 交互流程

### 创建提醒流程

```
[提醒列表页]
    │
    ├─→ 点击"+"按钮
    │
    ↓
[创建提醒页]
    │
    ├─→ 输入内容
    ├─→ 选择日期
    ├─→ 选择时间
    │
    ↓
    ├─→ 点击"保存"
    │
    ↓
[校验]
    │
    ├─ 失败 → 显示错误提示
    │
    └─ 成功 → 存入 storage
              │
              ↓
          返回列表页
              │
              ↓
          刷新列表
```

### 获取用户信息流程

```
[我的页面]
    │
    ├─→ onLoad() 读取存储的用户信息
    │
    ↓
[已有信息] ──→ 显示头像和昵称
    │
    └─ [无信息] ──→ 显示默认头像和"未获取用户信息"
                      │
                      ↓
                  显示"获取用户信息"按钮
                      │
                      ↓
                  用户点击授权
                      │
                      ├─ 成功 → 存储并显示
                      │
                      └─ 失败 → 保持默认状态
```

---

## 验收标准

### 功能验收

- [ ] tabBar 正常显示两个页签
- [ ] 提醒列表可正常显示
- [ ] 创建提醒可成功保存
- [ ] 数据持久化正常（关闭重开数据不丢失）
- [ ] 我的页面可获取用户信息（或显示默认）

### UI 验收

- [ ] 导航栏标题正确
- [ ] 浮动按钮位置正确
- [ ] 列表项布局正确
- [ ] 空状态显示正常
- [ ] 头像圆形显示

### 兼容性验收

- [ ] iOS 微信测试通过
- [ ] Android 微信测试通过
- [ ] 开发者工具测试通过

---

**本规格说明书与宪法具有同等约束力，开发工作必须遵循本文档的规定。**
