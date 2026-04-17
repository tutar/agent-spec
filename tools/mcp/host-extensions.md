# MCP Host Extensions

## 职责

本文件定义宿主可以建立在 MCP 之上的私有扩展。

这些扩展可以很实用，但不属于 MCP `2025-11-25` core conformance。

## Host Extension 范围

典型扩展包括：

- 基于 resource URI scheme 或 metadata 发现 `mcp skill`
- MCP bundle / package host，例如 `.mcpb`
- 本地 UI install / trust / verify affordance
- channel permission relay
- host-managed naming / namespacing 约定

## `mcp skill`


- `fetchMcpSkillsForClient()` 从 `skill://` resources 发现 skills

这是有价值的 host convention，但不应上升为 MCP core requirement。

推荐接口：

```text
McpSkillAdapter
  - discover_skills_from_resources(server_id, resources)
  - adapt_mcp_skill(server_id, remote_skill)
```

约束：

- `mcp skill` 的发现规则必须与普通 MCP prompt 保持语义分离
- 不应因为某个 host 支持 `mcp skill`，就要求所有 MCP client 都必须支持

## Bundle Host

桌面或 managed host 可以补：

```text
McpBundleHost
  - install_bundle(bundle_file)
  - verify_bundle(bundle_file)
  - load_bundle(bundle_id)
  - update_bundle(bundle_id)
  - uninstall_bundle(bundle_id)
```

这属于 host packaging / distribution capability，而不是 MCP protocol 本体。

## Trust Boundary

host extension 必须显式说明：

- 哪些能力属于安装/分发层
- 哪些能力属于 runtime 连接层
- 哪些能力会额外引入 trust / permission surface

否则实现者很容易把 host convenience feature 误写成 protocol mandate。
