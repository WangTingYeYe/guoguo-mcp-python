# PyPI 发布指南

## 📦 发布到 PyPI

### 1. 前提条件

#### 注册 PyPI 账户
1. 访问 [PyPI](https://pypi.org/) 并注册账户
2. 访问 [TestPyPI](https://test.pypi.org/) 注册测试账户（推荐先在测试环境发布）

#### 生成 API Token
1. 登录 PyPI 账户
2. 进入 Account settings → API tokens
3. 创建新的 API token（选择"Entire account"权限）
4. 保存生成的 token（格式：`pypi-...`）

### 2. 配置凭据

#### 方法1: 使用 keyring（推荐）
```bash
# 安装 keyring
pip install keyring

# 设置 PyPI 凭据
keyring set https://upload.pypi.org/legacy/ __token__
# 输入你的 API token（包含 pypi- 前缀）

# 设置 TestPyPI 凭据
keyring set https://test.pypi.org/legacy/ __token__
# 输入你的 TestPyPI API token
```

#### 方法2: 创建 .pypirc 文件
在用户主目录创建 `~/.pypirc` 文件：
```ini
[distutils]
index-servers =
    pypi
    testpypi

[pypi]
username = __token__
password = pypi-your-api-token-here

[testpypi]
repository = https://test.pypi.org/legacy/
username = __token__
password = pypi-your-testpypi-token-here
```

### 3. 发布流程

#### 第一步：发布到 TestPyPI（测试）
```bash
# 上传到测试环境
twine upload --repository testpypi dist/*

# 测试安装
pip install --index-url https://test.pypi.org/simple/ guoguo-mcp-server
```

#### 第二步：发布到正式 PyPI
```bash
# 上传到正式环境
twine upload dist/*
```

### 4. 验证发布

#### 安装测试
```bash
# 从 PyPI 安装
pip install guoguo-mcp-server

# 验证安装
guoguo-mcp-server --help
```

#### 查看包信息
```bash
pip show guoguo-mcp-server
```

### 5. 更新版本

#### 修改版本号
编辑 `pyproject.toml` 中的版本号：
```toml
[project]
version = "1.0.1"  # 增加版本号
```

#### 重新构建和发布
```bash
# 清理旧的构建文件
rm -rf dist/ build/ *.egg-info/

# 重新构建
python -m build

# 检查包
twine check dist/*

# 发布新版本
twine upload dist/*
```

### 6. 自动化发布（可选）

#### 使用 GitHub Actions
创建 `.github/workflows/publish.yml`：
```yaml
name: Publish to PyPI

on:
  release:
    types: [published]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.x'
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install build twine
    - name: Build package
      run: python -m build
    - name: Publish to PyPI
      uses: pypa/gh-action-pypi-publish@release/v1
      with:
        password: ${{ secrets.PYPI_API_TOKEN }}
```

### 7. 常见问题

#### 包名冲突
如果包名已存在，修改 `pyproject.toml` 中的 `name` 字段：
```toml
name = "guoguo-mcp-server-your-suffix"
```

#### 权限错误
确保 API token 有正确的权限，或者使用 `--verbose` 查看详细错误信息：
```bash
twine upload --verbose dist/*
```

#### 网络问题
使用代理或镜像：
```bash
twine upload --repository-url https://upload.pypi.org/legacy/ dist/*
```

### 8. 发布后管理

#### 检查下载统计
- 访问 PyPI 项目页面查看下载统计
- 使用 [pypistats](https://pypistats.org/) 查看详细统计

#### 维护项目
- 定期更新依赖版本
- 回应用户 issues 和 pull requests
- 发布安全更新和 bug 修复

### 9. 最佳实践

1. **语义化版本控制**：使用 [SemVer](https://semver.org/) 标准
2. **详细的 README**：包含安装、使用说明和示例
3. **变更日志**：维护 CHANGELOG.md 记录版本变更
4. **测试覆盖**：确保代码质量
5. **安全考虑**：定期更新依赖，扫描漏洞

---

## 🚀 快速发布命令

```bash
# 1. 构建包
python -m build

# 2. 检查包
twine check dist/*

# 3. 先发布到测试环境
twine upload --repository testpypi dist/*

# 4. 测试安装
pip install --index-url https://test.pypi.org/simple/ guoguo-mcp-server

# 5. 发布到正式环境
twine upload dist/*
``` 