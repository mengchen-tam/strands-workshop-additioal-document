# AWS Workshop 指南

本指南针对 [AWS Workshop](https://catalog.us-east-1.prod.workshops.aws/workshops/d674f40f-d636-4654-9322-04dafc7cc63e/zh-CN/30-lab-3/lab-3-1/3-1-2-cdk) 提供详细的配置和故障排除说明。

## 第一部分：使用 VSCode 连接 EC2

### 1. 生成 EC2 Key Pair

首先在 AWS 控制台中创建一个新的 Key Pair，下载 `.pem` 文件到本地。

### 2. 在 EC2 实例中添加 SSH 公钥

在 EC2 Connect 或已连接的终端中执行以下脚本：

```bash
cat > add_ssh_key.sh << 'EOF'
#!/bin/bash

if [ $# -ne 1 ]; then
    echo "用法: $0 <key-name>"
    exit 1
fi

KEY_NAME="$1"
REGION=$(curl -s http://169.254.169.254/latest/meta-data/placement/region)

echo "添加SSH Key: $KEY_NAME (区域: $REGION)"

PUBLIC_KEY=$(aws ec2 describe-key-pairs --key-names "$KEY_NAME" --region "$REGION" --include-public-key --query 'KeyPairs[0].PublicKey' --output text 2>/dev/null)

if [ $? -ne 0 ] || [ -z "$PUBLIC_KEY" ] || [ "$PUBLIC_KEY" = "None" ]; then
    echo "错误: 无法获取key '$KEY_NAME' 的公钥"
    exit 1
fi

mkdir -p ~/.ssh && chmod 700 ~/.ssh
[ -f ~/.ssh/authorized_keys ] && cp ~/.ssh/authorized_keys ~/.ssh/authorized_keys.backup.$(date +%s)
echo "$PUBLIC_KEY" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

echo "✓ SSH Key '$KEY_NAME' 已成功添加"
EOF

chmod +x add_ssh_key.sh

# 使用脚本添加key（替换为你的key名称）
./add_ssh_key.sh west-2-workshop
```

### 3. 配置 VSCode Remote SSH

安装 VSCode 的 Remote-SSH 插件，然后配置 SSH 连接。

在本地创建或编辑 `.ssh/config` 文件：

```
Host workshop
    IdentityFile "C:\Users\0749\Downloads\west-2-workshop.pem"
    HostName 35.95.37.253
    User ubuntu
```

**注意：** 
- 将 `IdentityFile` 路径替换为你的 `.pem` 文件实际路径
- 将 `HostName` 替换为你的 EC2 实例公网 IP
- 确保 `.pem` 文件权限正确（Windows 上右键属性 → 安全 → 高级，移除除当前用户外的所有权限）

## 第二部分：依赖安装故障排除

如果在安装 AWS CDK 依赖时遇到版本冲突或其他错误，使用以下命令安装特定版本：

```bash
npm install aws-cdk-lib@2.178.2 --save
npm audit fix --force
```

这将安装兼容的 CDK 库版本，避免版本不匹配问题。

## 第三部分：Exa MCP 配置

如果需要配置 Exa MCP 服务，在 `.kiro/settings/mcp.json` 文件中添加以下配置：

```json
{
  "mcpServers": {
    "exa": {
      "command": "npx",
      "args": ["-y", "exa-mcp-server"],
      "env": {
        "EXA_API_KEY": "<your-key>"
      }
    }
  }
}
```

**注意：** 请将 `EXA_API_KEY` 替换为你的实际 API 密钥。

## 常见问题

### SSH 连接问题
- 确保 EC2 安全组允许 SSH (端口 22) 访问
- 检查 `.pem` 文件权限
- 验证 EC2 实例状态为 "running"

### CDK 部署问题
- 确保 AWS 凭证配置正确
- 检查 IAM 权限是否足够
- 使用 `cdk bootstrap` 初始化环境

### MCP 配置问题
- 确保 Node.js 和 npm 已正确安装
- 检查网络连接和 API 密钥有效性

## 支持

如遇到其他问题，请参考 [AWS Workshop 文档](https://catalog.us-east-1.prod.workshops.aws/workshops/d674f40f-d636-4654-9322-04dafc7cc63e/zh-CN/30-lab-3/lab-3-1/3-1-2-cdk) 或联系技术支持。
