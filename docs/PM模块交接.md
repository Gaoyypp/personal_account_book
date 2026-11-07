# 6 号：类别与校验模块交接

## 已实现

- `backend/app/categories.py`：唯一类别来源，返回独立列表。
- `backend/app/validators.py`：录入参数校验、金额精确转分、日期和类别校验、RecordFilter、筛选解析、分页解析。
- `backend/app/routes/categories.py`：GET /api/categories，返回 README 约定的 data 对象。
- `backend/app/errors.py`：最小共享 ValidationError 契约，供 1 号接入统一错误处理。
- `tests/backend/test_validators.py`：业务校验及类别 HTTP 接口验证。

代码按用户授权通过 Codex 辅助实现；Git 提交使用本机已有身份，不代其他成员提交。

## 其他成员如何接入

1 号在 create_app 中注册 `categories_bp`，此 Blueprint 已含完整 `/api/categories` 路径，注册时不要再次添加 `/api` 前缀。

```python
from .routes.categories import categories_bp
from .errors import ValidationError

app.register_blueprint(categories_bp)

@app.errorhandler(ValidationError)
def handle_validation(error):
    return {"error": {"code": error.code, "message": error.message,
                      "fields": error.fields}}, error.status_code
```

3 号调用 `validate_record_payload(payload)`，取得 date/type/amount_cents/category/note，再生成 UUID 和 created_at、保存文件。输入 amount 为元字符串，输出 amount_cents 为整数分。

4 号调用 `parse_filters(request.args)` 和 `parse_pagination(request.args)`；后者返回 `(limit, offset)`。5 号只调用 parse_filters；金额筛选允许零。RecordFilter 日期为 Python date，比较时将记录 date 字符串转为 date，或使用过滤日期的 isoformat()。

路由按自己的白名单拒绝未知查询参数：记录接口允许 FILTER_FIELDS 加 limit/offset；统计接口仅允许 FILTER_FIELDS。parse_filters 有意不拒绝分页参数，以便查询接口共用。未知录入字段由 validate_record_payload 直接拒绝。

7/8 号从类别接口加载合法类别；校验失败的字段信息位于 error.fields。

## 本地验证

已在 Python 3.11.9、Flask 3.1.2、pytest 9.1.0 环境验证。

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements-dev.txt
.\.venv\Scripts\python.exe -m pytest -q
```

当前为目录骨架和 PM 模块，应用工厂、其他业务路由、前端和发布脚本仍待各负责人实现。类别接口已通过独立 Flask 测试应用验证；完整应用联调待 1 号装配后执行。

## 目录约定

根目录即项目根目录，不再套 co-lab2。目录 README 标注负责人，避免空目录无法被 Git 跟踪；不创建假业务实现。data/、web/ 为本地运行/构建目录，按规范忽略 Git，其他机器运行或构建时创建。
