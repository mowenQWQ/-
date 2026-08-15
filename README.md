# 网站测试安全规则（Web Security Test Rules）

一套**通用、负责任、最小影响**的网站 / Web 应用安全测试（授权渗透测试）Skill，适用于 [CodeBuddy](https://codebuddy.ai) 等支持 Skill 的 AI 编码助手。

> 本 Skill 只描述"在**已获授权**前提下如何负责任地做安全测试"，不包含任何针对未授权目标的攻击指引。使用者须自行确保对测试目标拥有合法授权。

> ⚠️ **本 Skill 由 AI（AI 编码助手）自动生成**，内容作为安全规范示例与参考，使用前请人工审阅，并结合目标实际授权情况调整。

> 🌐 **多语言 / Localization**：默认 **中文（`zh-CN`）**（即根目录文件）。英文版本见 `locales/en/`，其他语言按 `locales/README.md` 约定新增。切换：设置 `MOWEN_LANG=<代码>` 或放置 `lang` 文件（内容为语言代码），再读取 `locales/<代码>/` 下对应文件。

---

## 特性

- **授权优先（默认开启门禁）**：主用**非对称签名验证**（`AUTHORIZATION.json` + 受信公钥验签，最强防冒用 / 篡改）；清单白名单、交互确认、`mowenfalse` 等为备选降级路径（详见 SKILL.md 第零节）。
- **最小影响 / 隐蔽测试**：控频率、限外部副作用、不污染生产数据、测完必还原。
- **规范留痕**：强制记录真实当前时间（精确到秒）、结构化漏洞报告、错误记录机制。
- **去重与复查**：7 天内相同问题 / 已用攻击向量不重复；漏洞 7 天 → 1 月 → 半年 → 1 年 闭环复查。
- **扩展规则**：12 条覆盖合规边界、证据上报、技术补充（A/B/C）+ 10 条专项（D）。

---

## 目录结构

```
网站测试安全规则/
├── SKILL.md                                # Skill 主文件（规则与检查清单，权威源）
├── README.md                               # 本文档（项目说明 / 用法 / 授权速查 / 规则索引）
├── LICENSE                                 # MIT 许可证
├── .gitignore                              # 忽略私钥 / 本地授权证据，防误提交
├── references/
│   ├── report_template.md                  # 通用安全测试报告模板
│   ├── auth_notes.md                       # 通用认证 / 凭据管理 / 登录流程说明
│   ├── authorization_template.json         # 授权证据文件模板（含签名字段 + scope）
│   └── authorized_pubkeys/                 # 受信公钥目录（验签用）
│       ├── README.md                       #   公钥管理说明
│       └── example.pem                     #   演示公钥（生产请勿使用）
├── scripts/
│   └── sign_auth.sh                        # 授权声明非对称签名工具（RSA-SHA256）
├── AUTHORIZATION.json                      # （可选）项目级授权证据，门禁默认校验此文件
├── auth.disable                            # （可选）关闭门禁开关，一行一个：mowenfalse / mowenbrokentrue / mowenwaitrue
└── locales/                                # 多语言（默认中文在根目录；英文见 locales/en/）
```

---

## 安装

将本目录复制到项目的 Skill 目录：

## 安装

将本仓库中的 Skill 复制到 CodeBuddy 的 Skill 目录即可使用。下面提供两种安装方式：**安装到当前项目**（仅本项目可用）或 **安装到全局**（所有项目可用），任选其一。

### 方式一：安装到当前项目

#### 1. 先把仓库克隆到临时目录

```bash
git clone https://github.com/mowenQWQ/Web-Security-Test-Rules.git /tmp/web-security-test-rules

```

#### 2. 把 skill 复制到当前项目的 `.codebuddy/skills/` 下

```bash
mkdir -p .codebuddy/skills/
cp -r /tmp/web-security-test-rules/.codebuddy/skills/. .codebuddy/skills/

```

#### 3. 清理临时目录（可选）

```bash
rm -rf /tmp/web-security-test-rules

```

---

### 方式二：安装到全局 Skill 目录

#### 1. 先把仓库克隆到临时目录

```bash
git clone https://github.com/mowenQWQ/Web-Security-Test-Rules.git /tmp/web-security-test-rules

```

#### 2. 把 skill 复制到全局目录 `~/.codebuddy/skills/` 下

```bash
mkdir -p ~/.codebuddy/skills/
cp -r /tmp/web-security-test-rules/.codebuddy/skills/. ~/.codebuddy/skills/

```

#### 3. 清理临时目录（可选）

```bash
rm -rf /tmp/web-security-test-rules

```

> 💡 说明：
> - `cp -r 源/. 目标/` 末尾的 `/.` 表示复制目录的**全部内容**（含隐藏文件），避免多嵌套一层目录。
> - Windows 用户若无 `git`/`cp` 命令，可使用 Git Bash，或手动下载仓库 zip 后解压，将 `.codebuddy/skills/` 下的内容复制到对应目录。

---

## 用法

安装后，在对话中提到 **“安全测试”“渗透测试”“隐蔽测试”** 等关键词，或明确要求对某网站做安全测试时，Skill 会自动激活。

1. **发起测试**：对 AI 说 “帮我测试 `https://example.com` 的安全”，Skill 会先走 **授权门禁**（确认你拥有测试授权 / 为目标所有者），通过后才执行。
2. **遵循最小影响原则**：默认只进行非破坏性、低影响的探测，避免对目标造成干扰。
3. **查看报告**：测试完成后会输出结构化的风险发现与修复建议。

> ⚠️ 本 Skill 仅用于 **已获授权** 的安全测试（自有系统、CTF、Bug Bounty 等）。未经授权对他人系统进行测试属于违法行为，请勿滥用。
## 用法

1. **发起测试**：对 AI 说"帮我测试 https://example.com 的安全"，Skill 先走"授权门禁"。
2. **跟随检查清单**：SKILL.md 内置"测试前 / 中 / 后"清单，确保最小影响与规范留痕。
3. **生成报告**：测试后生成 `YYYY-MM-DD_findings.md`，含漏洞编号 / 等级 / 描述 / PoC / 修复建议 / 后续方向，默认存 `reports/security/`。
4. **复查**：按 7 天 → 1 月 → 半年 → 1 年 周期复查修复情况，结果追加到同一报告。

---

## 授权门禁（默认开启）

为防止误用于未授权目标，本 Skill **默认强制"先验证授权再测试"**。

**主路径（推荐）：签名授权令牌** —— 授权方用私钥对"授权声明"签名，项目放带签名的 `AUTHORIZATION.json`，Skill 用对应公钥（`references/authorized_pubkeys/`）验签，并校验目标范围、有效期、账号等。

**备选降级路径**（细节见 SKILL.md 第零节）：
- 备选 A：仅 `AUTHORIZATION.json` 白名单 + 有效期（无验签，弱一级）
- 备选 B：无清单时一次性交互确认
- 备选 D：签名 + 白名单 + SHA256 + 提交审计的多层组合

### 三级开关（逃生口，写入 `auth.disable` 一行一个，或环境变量 `MOWEN_AUTH_OVERRIDE`）

> 逃生口是"已知风险、主动关闭校验"的开关，**不是授权本身**；关闭不代表可测未授权目标，对未授权目标一律禁止。

| 开关 | 作用 | 注意 |
|------|------|------|
| `mowenfalse` | 跳过**授权验证**（仅非破坏性测试） | 仍须遵守范围约束（规则 23，不跳出授权范围） |
| `mowenbrokentrue` | 跳过破坏性操作的**显式授权**要求 | **备份不可跳过**——无论是否开启，破坏性动作前必须先做可恢复备份（规则 22、A5）；不等于 `mowenfalse`，不关闭授权验证 |
| `mowenwaitrue` | 确认已具备 / 已等待与测试目标相关的**外站授权** | 仅放宽外站关联尝试授权要求，不关门禁、不关破坏性格闸；涉及外站须在报告中列出 |

关闭任一开关后，报告中须标注"授权门禁已关闭（xxx）"，并重申仅用于已授权目标。

### 路径速查

| 场景 | 做法 | 门禁行为 |
|------|------|----------|
| 常规授权（推荐） | 放带签名 `AUTHORIZATION.json` | 公钥验签 + 白名单 + 有效期，通过放行 |
| 不便签名 | 放无签名 `AUTHORIZATION.json` | 仅白名单 + 有效期（备选 A） |
| 临时无文件 | 不放大文件 | 一次性交互确认（备选 B） |
| 已授权、省步骤（非破坏性） | `auth.disable` 写 `mowenfalse` | 跳过验证，报告标注（备选 C） |
| 破坏性想省步骤 | `auth.disable` 写 `mowenbrokentrue` | 跳过破坏性显式授权（备份仍强制），仍受范围约束 |
| 需测目标关联外站 | 取得该外站授权，或写 `mowenwaitrue` | 放行外站关联尝试（仅限本次目标相关，见规则 23） |

---

## 规则清单（索引，详见 SKILL.md）

### 核心规则（23 条）
授权、账号、方法、工具/Skill、还原、无法还原、验证、去重、报告、密钥、基线记录、隐蔽测试、总结报告、方法不重样、前端资源变更跟踪、认证回退检测、用户资源上传先备份、不可逆写入端点控制、漏洞复查周期、操作全程留痕 + 报告位置可自定义、**破坏性须显式授权且先备份（规则 22）**、**禁止越界 / 跳出授权范围（含外站关联须授权，规则 23）**。

### 扩展规则

| 类别 | 规则 |
|------|------|
| A 合规与边界 | A1 禁测项/范围边界 🔒、A2 敏感数据最小化与脱敏、A3 合规与法律边界 🔒、A4 测试时段/变更窗口、A5 破坏性须显式授权并先备份 🔒 |
| B 证据与上报 | B1 证据留存、B2 高危即时上报、B3 停止/熔断、B4 报告分发最小化 |
| C 技术补充 | C1 依赖/供应链标注、C2 非幂等请求谨慎、C3 CSRF/CORS/同源、C4 速率/并发上限 |
| D 专项技术 | D1 令牌/JWT、D2 会话与认证、D3 业务逻辑/并发、D4 越权组合、D5 注入扩展、D6 SSRF、D7 文件上传、D8 信息泄露、D9 配置与传输、D10 供应链/组件 |

> 🔒 = 与授权 / 合规边界直接相关，测试前须向授权方确认范围。

### 推荐工具与方法
仅记名称，详见 `SKILL.md` 第十一节：已安检本地 Skill（`browser-automation` / `playwright-cli` / `github-connector`），推荐开源工具（OWASP ZAP、Nuclei、wapiti、Nikto、testssl.sh、sqlmap、Gitleaks 等）与测试方法（被动收集、Fuzzing、代码审计、威胁建模等）。

---

## 多语言 / Localization

默认 **中文（`zh-CN`）**，即根目录 `SKILL.md` / `README.md` / `references/`。其他语言放 `locales/<代码>/`（示例 `locales/en/`）。

- **切换**：设置 `MOWEN_LANG=<代码>`（如 `en`）或放 `lang` 文件（内容 `en`），再读 `locales/<代码>/SKILL.md`。
- **新增**：复制 `locales/en/` 到 `locales/<新代码>/` 翻译即可（建议 BCP 47，如 `ja`、`zh-TW`）。
- **维护**：根目录中文为权威源（source of truth）；更新规则时同步所有 `locales/*`，保持 `version` 与内容一致。
- 约定详见 [`locales/README.md`](./locales/README.md)。

---

## 开源相关

- **许可证**：[MIT](./LICENSE.txt)。可自由使用、修改、分发，请保留版权与许可声明。
- **贡献**：欢迎提交 Issue / PR 完善规则或补充场景。
- **商标**：本 Skill 与任何具体厂商、站点无关，均为通用规则。

---

## 免责声明

本 Skill 仅用于**已获合法授权的**安全测试与防护加固。使用者须自行承担因 misuse（如对未授权目标测试）产生的一切法律责任。作者与贡献者不对任何滥用行为负责。
