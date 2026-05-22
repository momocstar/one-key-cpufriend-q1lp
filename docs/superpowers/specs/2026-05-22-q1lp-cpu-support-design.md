# Q1LP 魔改 CPU 支持设计文档

## 背景

用户需要在 one-key-cpufriend 项目中添加对 q1lp 魔改 CPU（i9-13950HX 马甲，微码 189）的支持。

## 需求

- **board-id**: `Mac-27AD2F918AE68F61`
- **CPU**: q1lp（魔改 i9-13950HX），微码 189
- **LFM**: 固定 800 MHz
- **EPP**: 固定平衡性能模式 (0x40)

## 设计原则

- **高内聚**：q1lp 相关逻辑集中管理
- **低耦合**：q1lp 处理不影响现有 CPU 支持

## 架构设计

### 1. 数据层

在现有支持数组后新增独立数组：

```bash
# q1lp 魔改CPU专用 (微码189, i9-13950HX马甲)
Q1LP_MODELS=(
  'Mac-27AD2F918AE68F61'
)
```

### 2. 检测层

扩展 `checkBoardID` 函数，新增 support=5 标识：

```bash
function checkBoardID() {
  if echo "${EPP_SUPPORTED_MODELS[@]}" | grep -w "${BOARD_ID}" &> /dev/null; then
    support=2
  elif echo "${EPP_SUPPORTED_MODELS_SPECIAL[@]}" | grep -w "${BOARD_ID}" &> /dev/null; then
    support=3
  elif echo "${LFM_800_MODELS[@]}" | grep -w "${BOARD_ID}" &> /dev/null; then
    support=4
  elif echo "${LFM_SUPPORTED_MODELS[@]}" | grep -w "${BOARD_ID}" &> /dev/null; then
    support=1
  elif echo "${Q1LP_MODELS[@]}" | grep -w "${BOARD_ID}" &> /dev/null; then
    support=5
  else
    echo -e "[ ${RED}ERROR${OFF} ]: 抱歉，你的board-id暂不被支持!"
    exit 1
  fi
}
```

### 3. 处理层

新增独立函数 `handleQ1LP`，处理 LFM 和 EPP 设置：

```bash
# 处理q1lp魔改CPU (固定LFM 800MHz, EPP平衡性能模式)
function handleQ1LP() {
  echo
  echo "------------------------------"
  echo "|****** q1lp CPU 优化 ******|"
  echo "------------------------------"
  echo "LFM: 800MHz"
  echo "EPP: 平衡性能模式"
  echo

  # LFM: 修改为800MHz
  # 020000000d000000 -> 0200000008000000
  /usr/bin/sed -i "" "s:AgAAAA0AAAA:AgAAAAgAAAA:g" "$BOARD_ID.plist"
  /usr/bin/sed -i "" "s:AgAAAAwAAAA:AgAAAAgAAAA:g" "$BOARD_ID.plist"

  # EPP: 修改为平衡性能模式 (0x40)
  # 0x80 -> 0x40
  /usr/bin/sed -i "" "s:CAAAAAAAAAAAAAAAAAAAAAc:BAAAAAAAAAAAAAAAAAAAAAc:g" "$BOARD_ID.plist"
  /usr/bin/sed -i "" "s:CAAAAAAAAAAAAAAAAAAAAAd:BAAAAAAAAAAAAAAAAAAAAAd:g" "$BOARD_ID.plist"
  /usr/bin/sed -i "" "s:CSAAAAAAAAAAAAAAAAAAAAc:BAAAAAAAAAAAAAAAAAAAAAc:g" "$BOARD_ID.plist"
  /usr/bin/sed -i "" "s:CQAAAAAAAAAAAAAAAAAAAAc:BAAAAAAAAAAAAAAAAAAAAAc:g" "$BOARD_ID.plist"
  /usr/bin/sed -i "" "s:ZXBwAAAAAAAAAAAAAAAAAAAAAACS:ZXBwAAAAAAAAAAAAAAAAAAAAAABA:g" "$BOARD_ID.plist"
  /usr/bin/sed -i "" "s:ZXBwAAAAAAAAAAAAAAAAAAAAAACA:ZXBwAAAAAAAAAAAAAAAAAAAAAABA:g" "$BOARD_ID.plist"
  /usr/bin/sed -i "" "s:ZXBwAAAAAAAAAAAAAAAAAAAAAACQ:ZXBwAAAAAAAAAAAAAAAAAAAAAABA:g" "$BOARD_ID.plist"

  echo -e "[ ${GREEN}OK${OFF} ] q1lp CPU 配置完成"
}
```

### 4. 主流程层

修改 `main` 函数，添加 q1lp 分支：

```bash
function main(){
  init
  printHeader
  checkBoardID
  downloadKext
  if [ "${support}" == 1 ]; then
    copyPlist
    changeLFM
  elif [ "${support}" == 2 ] || [ "${support}" == 3 ] || [ "${support}" == 4 ]; then
    copyPlist
    changeLFM
    changeEPP
  elif [ "${support}" == 5 ]; then
    copyPlist
    handleQ1LP
  fi
  generateKext
  clean
  # ... 输出信息 ...
  exit 0
}
```

## 流程图

```
main()
  │
  ├─ init()
  ├─ printHeader()
  ├─ checkBoardID()
  │     └─ support=5 (q1lp detected)
  ├─ downloadKext()
  │
  ├─ support=5?
  │     ├─ YES → copyPlist() → handleQ1LP()
  │     └─ NO  → 现有逻辑
  │
  ├─ generateKext()
  ├─ clean()
  └─ exit 0
```

## 文件修改清单

| 文件 | 修改内容 |
|------|----------|
| `one-key-cpufriend_cn.sh` | 新增 Q1LP_MODELS、handleQ1LP 函数、修改 checkBoardID 和 main |
| `one-key-cpufriend.sh` | 同步修改英文版本 |

## 测试验证

1. 在 q1lp 机器上运行脚本
2. 验证生成的 CPUFriendDataProvider.kext 中 LFM 为 800 MHz
3. 验证 EPP 为平衡性能模式 (0x40)
4. 验证其他机型不受影响
