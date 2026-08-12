# sing-box-rules geoip/geosite ruleset 自动导出

该项目通过 GitHub Actions 监听 `lyc8503/sing-box-rules` 的 Release 更新，并自动生成 `sing-box` 的 `geoip/geosite` 分组规则文件，整理后发布到 `traition/sing-box-ruleset-json` 的新 Release。

## 如何使用

在sing-box的配置文件中添加你需要的rule-set

```JSON
{
  "route": {
    "rule_set": [
      {
        "type": "local",
        "tag": "ai-services",
        "format": "source",
        "path": "./rules/ai-services.json"
      }
    ]
  }
}
```

## 工作流做了什么

当 `github.com/lyc8503/sing-box-rules` 发布新 Release 时，工作流执行以下逻辑：

1. 安装 `sing-box`
2. 下载并覆盖到 `sing-box` 配置目录 `/etc/sing-box`：
   - `geoip.db`
   - `geosite.db`
3. 在 `/etc/sing-box` 中执行：
   - `sing-box geoip list`
   - 输出每一行一个“组名”（每个组名对应一份组规则文件）
4. 遍历每个组名，执行：
   - `sing-box geoip export 组名称`
5. 执行：
   - `sing-box geosite list`
   - 每行包含“组名 + 额外信息”，仅取每行的**第一段组名**
6. 遍历每个组名，执行：
   - `sing-box geosite 组名称`
7. 在 `/etc/sing-box` 下整理导出结果：
   - 新建目录：`/etc/sing-box/geoip`、`/etc/sing-box/geosite`
   - 将 `geoip-*` 前缀的文件移动到 `geoip/`
   - 将 `geosite-*` 前缀的文件移动到 `geosite/`
8. 去掉导出文件名的前缀：
   - 将 `geoip-xxx` 重命名为 `xxx`
   - 将 `geosite-xxx` 重命名为 `xxx`
9. 删除 `/etc/sing-box/config.json`
10. 将 `/etc/sing-box/geoip` 与 `/etc/sing-box/geosite` 下的所有文件打包，并发布到`github.com/traition/sing-box-ruleset-json`
