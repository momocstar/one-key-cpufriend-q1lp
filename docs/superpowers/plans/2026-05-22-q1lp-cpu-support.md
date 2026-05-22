# Q1LP CPU 支持实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 为 one-key-cpufriend 项目添加 q1lp 魔改 CPU (i9-13950HX 马甲) 支持

**Architecture:** 新增独立的 Q1LP_MODELS 数组和 handleQ1LP 函数，通过 support=5 标识符实现高内聚低耦合的代码结构

**Tech Stack:** Bash shell script, sed, plist 处理

---

## 文件结构

| 文件 | 职责 |
|------|------|
| `one-key-cpufriend_cn.sh` | 中文版主脚本 - 添加 q1lp 数据、检测、处理逻辑 |
| `one-key-cpufriend.sh` | 英文版主脚本 - 同步修改 |

---

## Task 1: 修改中文版脚本 (one-key-cpufriend_cn.sh)

**Files:**
- Modify: `one-key-cpufriend_cn.sh`

- [ ] **Step 1: 添加 Q1LP_MODELS 数组**

在 `LFM_800_MODELS` 数组定义后（约第68行），新增 Q1LP_MODELS 数组：

```bash
LFM_800_MODELS=(
  'Mac-0CFF9C7C2B63DF8D' # MacBookAir9,1
  'Mac-5F9802EFE386AA28' # MacBookPro16,2
)

# q1lp 魔改CPU专用 (微码189, i9-13950HX马甲)
Q1LP_MODELS=(
  'Mac-27AD2F918AE68F61'
)
```

- [ ] **Step 2: 修改 checkBoardID 函数**

找到 `checkBoardID` 函数（约第91-104行），在 `else` 分支前添加 q1lp 检测：

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

- [ ] **Step 3: 添加 handleQ1LP 函数**

在 `customizeLFM` 函数后（约第278行），新增 handleQ1LP 函数：

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

- [ ] **Step 4: 修改 main 函数**

找到 `main` 函数（约第440-461行），添加 support=5 分支：

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
  echo -e "[ ${GREEN}OK${OFF} ]脚本运行结束, 请把桌面上的 CPUFriend 和 CPUFriendDataProvider"
  echo "Clover: 放入 /CLOVER/kexts/Other/ 或者 L/E/ 路径下"
  echo "OC: 放入 /OC/Kexts/ 并添加 README_CN 中的补丁到 config.plist - Kernel - Add"
  exit 0
}

main
```

---

## Task 2: 修改英文版脚本 (one-key-cpufriend.sh)

**Files:**
- Modify: `one-key-cpufriend.sh`

- [ ] **Step 1: 添加 Q1LP_MODELS 数组**

在 `LFM_800_MODELS` 数组定义后，新增 Q1LP_MODELS 数组：

```bash
LFM_800_MODELS=(
  'Mac-0CFF9C7C2B63DF8D' # MacBookAir9,1
  'Mac-5F9802EFE386AA28' # MacBookPro16,2
)

# q1lp modified CPU support (microcode 189, i9-13950HX rebranded)
Q1LP_MODELS=(
  'Mac-27AD2F918AE68F61'
)
```

- [ ] **Step 2: 修改 checkBoardID 函数**

找到 `checkBoardID` 函数，在 `else` 分支前添加 q1lp 检测：

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
    echo -e "[ ${RED}ERROR${OFF} ]: Sorry, your board-id is not supported yet!"
    exit 1
  fi
}
```

- [ ] **Step 3: 添加 handleQ1LP 函数**

在 `customizeLFM` 函数后，新增 handleQ1LP 函数（英文版）：

```bash
# Handle q1lp modified CPU (fixed LFM 800MHz, EPP balanced performance mode)
function handleQ1LP() {
  echo
  echo "------------------------------"
  echo "|****** q1lp CPU Tune ******|"
  echo "------------------------------"
  echo "LFM: 800MHz"
  echo "EPP: Balanced Performance"
  echo

  # LFM: Set to 800MHz
  # 020000000d000000 -> 0200000008000000
  /usr/bin/sed -i "" "s:AgAAAA0AAAA:AgAAAAgAAAA:g" "$BOARD_ID.plist"
  /usr/bin/sed -i "" "s:AgAAAAwAAAA:AgAAAAgAAAA:g" "$BOARD_ID.plist"

  # EPP: Set to balanced performance mode (0x40)
  # 0x80 -> 0x40
  /usr/bin/sed -i "" "s:CAAAAAAAAAAAAAAAAAAAAAc:BAAAAAAAAAAAAAAAAAAAAAc:g" "$BOARD_ID.plist"
  /usr/bin/sed -i "" "s:CAAAAAAAAAAAAAAAAAAAAAd:BAAAAAAAAAAAAAAAAAAAAAd:g" "$BOARD_ID.plist"
  /usr/bin/sed -i "" "s:CSAAAAAAAAAAAAAAAAAAAAc:BAAAAAAAAAAAAAAAAAAAAAc:g" "$BOARD_ID.plist"
  /usr/bin/sed -i "" "s:CQAAAAAAAAAAAAAAAAAAAAc:BAAAAAAAAAAAAAAAAAAAAAc:g" "$BOARD_ID.plist"
  /usr/bin/sed -i "" "s:ZXBwAAAAAAAAAAAAAAAAAAAAAACS:ZXBwAAAAAAAAAAAAAAAAAAAAAABA:g" "$BOARD_ID.plist"
  /usr/bin/sed -i "" "s:ZXBwAAAAAAAAAAAAAAAAAAAAAACA:ZXBwAAAAAAAAAAAAAAAAAAAAAABA:g" "$BOARD_ID.plist"
  /usr/bin/sed -i "" "s:ZXBwAAAAAAAAAAAAAAAAAAAAAACQ:ZXBwAAAAAAAAAAAAAAAAAAAAAABA:g" "$BOARD_ID.plist"

  echo -e "[ ${GREEN}OK${OFF} ] q1lp CPU configuration done"
}
```

- [ ] **Step 4: 修改 main 函数**

找到 `main` 函数，添加 support=5 分支：

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
  echo -e "[ ${GREEN}OK${OFF} ] Script completed. Please copy CPUFriend and CPUFriendDataProvider from Desktop"
  echo "Clover: to /CLOVER/kexts/Other/ or L/E/"
  echo "OC: to /OC/Kexts/ and add patches from README to config.plist - Kernel - Add"
  exit 0
}

main
```

---

## Task 3: 提交代码

**Files:**
- Modify: `one-key-cpufriend_cn.sh`, `one-key-cpufriend.sh`

- [ ] **Step 1: 验证修改**

```bash
bash -n one-key-cpufriend_cn.sh && echo "中文版语法正确"
bash -n one-key-cpufriend.sh && echo "英文版语法正确"
```

Expected: 两个脚本都输出 "语法正确"

- [ ] **Step 2: 提交更改**

```bash
git add one-key-cpufriend_cn.sh one-key-cpufriend.sh
git commit -m "$(cat <<'EOF'
Feat: Add q1lp modified CPU support (i9-13950HX rebranded)

- Add Q1LP_MODELS array for board-id detection
- Add handleQ1LP function with fixed LFM 800MHz and EPP balanced performance
- Update checkBoardID to support=5 for q1lp detection
- Update main function with q1lp branch

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
EOF
)"
```

Expected: 提交成功

---

## 验证清单

- [ ] 在 q1lp 机器上运行脚本，确认自动应用 LFM 800MHz 和 EPP 平衡性能模式
- [ ] 验证生成的 CPUFriendDataProvider.kext 中配置正确
- [ ] 确认其他机型不受影响（现有 support=1-4 逻辑保持不变）
