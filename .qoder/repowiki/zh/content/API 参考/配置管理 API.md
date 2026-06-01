# 配置管理 API

<cite>
**本文引用的文件**
- [chromium_options.py](file://DrissionPage/_configs/chromium_options.py)
- [chromium_options.pyi](file://DrissionPage/_configs/chromium_options.pyi)
- [session_options.py](file://DrissionPage/_configs/session_options.py)
- [session_options.pyi](file://DrissionPage/_configs/session_options.pyi)
- [options_manage.py](file://DrissionPage/_configs/options_manage.py)
- [options_manage.pyi](file://DrissionPage/_configs/options_manage.pyi)
- [configs.ini](file://DrissionPage/_configs/configs.ini)
- [__init__.py](file://DrissionPage/__init__.py)
</cite>

## 目录
1. [简介](#简介)
2. [项目结构](#项目结构)
3. [核心组件](#核心组件)
4. [架构总览](#架构总览)
5. [详细组件分析](#详细组件分析)
6. [依赖关系分析](#依赖关系分析)
7. [性能考量](#性能考量)
8. [故障排查指南](#故障排查指南)
9. [结论](#结论)
10. [附录](#附录)

## 简介
本文件为 DrissionPage 配置管理模块的完整 API 参考，覆盖 ChromiumOptions、SessionOptions 与 OptionsManager 三大类。内容包括：
- 所有公共方法与属性的签名、用途与参数说明
- 配置项设置、默认值获取、配置验证与持久化流程
- 配置文件读写、运行时修改、配置合并与优先级
- 作用域管理与配置迁移的指导原则
- 使用示例与最佳实践

## 项目结构
配置管理模块位于 DrissionPage/_configs 目录，包含以下关键文件：
- chromium_options.py / .pyi：Chromium 浏览器配置类
- session_options.py / .pyi：HTTP 会话配置类
- options_manage.py / .pyi：INI 配置文件读写与管理
- configs.ini：默认配置模板
- __init__.py：对外导出 ChromiumOptions 与 SessionOptions

```mermaid
graph TB
subgraph "配置管理模块"
CO["ChromiumOptions<br/>浏览器配置"]
SO["SessionOptions<br/>会话配置"]
OM["OptionsManager<br/>INI文件管理"]
CFG["configs.ini<br/>默认配置模板"]
end
CO --> OM
SO --> OM
OM --> CFG
```

图表来源
- [chromium_options.py:16-417](file://DrissionPage/_configs/chromium_options.py#L16-L417)
- [session_options.py:20-372](file://DrissionPage/_configs/session_options.py#L20-L372)
- [options_manage.py:15-143](file://DrissionPage/_configs/options_manage.py#L15-L143)
- [configs.ini:1-34](file://DrissionPage/_configs/configs.ini#L1-L34)

章节来源
- [chromium_options.py:16-417](file://DrissionPage/_configs/chromium_options.py#L16-L417)
- [session_options.py:20-372](file://DrissionPage/_configs/session_options.py#L20-L372)
- [options_manage.py:15-143](file://DrissionPage/_configs/options_manage.py#L15-L143)
- [configs.ini:1-34](file://DrissionPage/_configs/configs.ini#L1-L34)
- [__init__.py:22-28](file://DrissionPage/__init__.py#L22-L28)

## 核心组件
- ChromiumOptions：管理 Chromium 浏览器启动参数、用户数据、偏好、标志位、代理、超时、加载模式等。
- SessionOptions：管理 requests.Session 的 headers、cookies、auth、proxies、params、verify、cert、adapters、stream、trust_env、max_redirects、timeout、下载路径等。
- OptionsManager：统一读取/写入 INI 配置文件，提供节与键值的增删改查与持久化能力，并维护默认配置模板。

章节来源
- [chromium_options.py:16-417](file://DrissionPage/_configs/chromium_options.py#L16-L417)
- [session_options.py:20-372](file://DrissionPage/_configs/session_options.py#L20-L372)
- [options_manage.py:15-143](file://DrissionPage/_configs/options_manage.py#L15-L143)

## 架构总览
ChromiumOptions 与 SessionOptions 在构造时通过 OptionsManager 读取默认或指定的 INI 文件；二者均提供 save/save_to_default 方法将当前内存配置写回 INI。OptionsManager 统一负责 INI 文件的读取、解析、写入与默认模板初始化。

```mermaid
classDiagram
class OptionsManager {
+ini_path : Path?
+file_exists : bool
+get_value(section, item) Any
+get_option(section) dict
+set_item(section, item, value) OptionsManager
+remove_item(section, item) OptionsManager
+save(path) str
+save_to_default() str
+show() void
}
class ChromiumOptions {
+ini_path : str?
+download_path : str
+browser_path : str
+user_data_path : str?
+tmp_path : str?
+user : str
+load_mode : str
+timeouts : dict
+proxy : str?
+address : str
+arguments : list
+extensions : list
+preferences : dict
+flags : dict
+retry_times : int
+retry_interval : float
+is_headless : bool
+set_retry(times, interval) ChromiumOptions
+set_argument(arg, value) ChromiumOptions
+remove_argument(value) ChromiumOptions
+add_extension(path) ChromiumOptions
+remove_extensions() ChromiumOptions
+set_pref(arg, value) ChromiumOptions
+remove_pref(arg) ChromiumOptions
+remove_pref_from_file(arg) ChromiumOptions
+set_flag(flag, value) ChromiumOptions
+clear_flags_in_file() ChromiumOptions
+clear_flags() ChromiumOptions
+clear_arguments() ChromiumOptions
+clear_prefs() ChromiumOptions
+set_timeouts(base, page_load, script) ChromiumOptions
+set_user(user) ChromiumOptions
+headless(on_off) ChromiumOptions
+no_imgs(on_off) ChromiumOptions
+no_js(on_off) ChromiumOptions
+mute(on_off) ChromiumOptions
+incognito(on_off) ChromiumOptions
+new_env(on_off) ChromiumOptions
+ignore_certificate_errors(on_off) ChromiumOptions
+set_user_agent(user_agent) ChromiumOptions
+set_proxy(proxy) ChromiumOptions
+set_load_mode(value) ChromiumOptions
+set_local_port(port) ChromiumOptions
+set_address(address) ChromiumOptions
+set_browser_path(path, edge) ChromiumOptions
+set_download_path(path) ChromiumOptions
+set_tmp_path(path) ChromiumOptions
+set_user_data_path(path) ChromiumOptions
+set_cache_path(path) ChromiumOptions
+use_system_user_path(on_off) ChromiumOptions
+auto_port(on_off, scope) ChromiumOptions
+existing_only(on_off) ChromiumOptions
+save(path) str
+save_to_default() str
+remove_test_type() ChromiumOptions
}
class SessionOptions {
+ini_path : str?
+download_path : str
+timeout : float
+proxies : dict
+retry_times : int
+retry_interval : float
+headers : dict?
+cookies : list?
+auth : Any?
+hooks : dict?
+params : dict?
+verify : bool?
+cert : str|tuple?
+adapters : list?
+stream : bool?
+trust_env : bool?
+max_redirects : int?
+set_download_path(path) SessionOptions
+set_timeout(second) SessionOptions
+set_proxies(http, https) SessionOptions
+set_retry(times, interval) SessionOptions
+set_headers(headers) SessionOptions
+set_a_header(name, value) SessionOptions
+remove_a_header(name) SessionOptions
+clear_headers() SessionOptions
+set_cookies(cookies) SessionOptions
+set_auth(auth) SessionOptions
+set_hooks(hooks) SessionOptions
+set_params(params) SessionOptions
+set_verify(on_off) SessionOptions
+set_cert(cert) SessionOptions
+add_adapter(url, adapter) SessionOptions
+set_stream(on_off) SessionOptions
+set_trust_env(on_off) SessionOptions
+set_max_redirects(times) SessionOptions
+save(path) str
+save_to_default() str
+as_dict() dict
+make_session() (Session, CaseInsensitiveDict?)
+from_session(session, headers) SessionOptions
}
ChromiumOptions --> OptionsManager : "读取/写入"
SessionOptions --> OptionsManager : "读取/写入"
```

图表来源
- [chromium_options.py:16-417](file://DrissionPage/_configs/chromium_options.py#L16-L417)
- [session_options.py:20-372](file://DrissionPage/_configs/session_options.py#L20-L372)
- [options_manage.py:15-143](file://DrissionPage/_configs/options_manage.py#L15-L143)

## 详细组件分析

### ChromiumOptions API 参考
- 初始化
  - 构造函数：ChromiumOptions(read_file=True, ini_path=None)
  - 功能：从 INI 文件读取默认配置，或禁用文件读取直接使用内存默认值
  - 关键点：若 ini_path 指定且不存在，抛出异常；内部解析 arguments 中的 --headless、--user-data-dir、--profile-directory 等并设置相应属性

- 属性（只读）
  - download_path：默认下载路径
  - browser_path：浏览器可执行文件路径
  - user_data_path：用户数据文件夹路径
  - tmp_path：临时文件夹路径
  - user：用户配置文件夹名称
  - load_mode：页面加载策略，'normal' | 'eager' | 'none'
  - timeouts：超时字典 {base, page_load, script}
  - proxy：代理字符串
  - address：浏览器地址（ip:port）
  - arguments：命令行参数列表
  - extensions：扩展路径列表
  - preferences：用户首选项字典
  - flags：实验项字典
  - system_user_path：是否使用系统用户路径
  - is_existing_only：是否只接管现有浏览器
  - is_auto_port：是否启用自动端口（可为范围）
  - retry_times/retry_interval：重试次数与间隔
  - is_headless：是否无头模式

- 配置设置与修改
  - set_retry(times=None, interval=None)：设置重试次数与间隔
  - set_argument(arg, value=None)：添加或移除参数；特殊处理 --headless
  - remove_argument(value)：移除指定参数
  - add_extension(path) / remove_extensions()：扩展管理
  - set_pref(arg, value) / remove_pref(arg) / remove_pref_from_file(arg)：首选项管理
  - set_flag(flag, value=None) / clear_flags_in_file() / clear_flags()：实验项管理
  - clear_arguments() / clear_prefs()：清空已设置项
  - set_timeouts(base=None, page_load=None, script=None)：设置超时
  - set_user(user='Default')：设置用户配置文件夹
  - headless(on_off=True)：无头模式开关
  - no_imgs/on_js/mute/incognito/new_env(ignore_certificate_errors)：常用开关
  - set_user_agent(user_agent)：设置 UA
  - set_proxy(proxy)：设置代理并解析 URL/用户名/密码
  - set_load_mode(value)：设置加载模式
  - set_local_port(port)：设置本地端口（与 auto_port/set_address 互斥）
  - set_address(address)：设置浏览器地址（支持 ws/wss）
  - set_browser_path(path=None, edge=False)：设置浏览器路径
  - set_download_path(path) / set_tmp_path(path)：设置路径
  - set_user_data_path(path) / set_cache_path(path)：设置用户数据与缓存路径
  - use_system_user_path(on_off=True)：使用系统用户路径
  - auto_port(on_off=True, scope=None)：自动端口分配（含范围）
  - existing_only(on_off=True)：接管现有浏览器
  - remove_test_type()：移除测试相关参数

- 保存与默认配置
  - save(path=None)：保存到指定路径或当前读取的 INI；支持传入 'default'
  - save_to_default()：保存到默认 INI
  - 返回值：保存文件的绝对路径

- 使用示例（路径）
  - 基础使用与参数设置：[chromium_options.py:16-417](file://DrissionPage/_configs/chromium_options.py#L16-L417)
  - 保存配置到 INI：[chromium_options.py:364-417](file://DrissionPage/_configs/chromium_options.py#L364-L417)

章节来源
- [chromium_options.py:16-417](file://DrissionPage/_configs/chromium_options.py#L16-L417)
- [chromium_options.pyi:12-408](file://DrissionPage/_configs/chromium_options.pyi#L12-L408)

### SessionOptions API 参考
- 初始化
  - 构造函数：SessionOptions(read_file=True, ini_path=None)
  - 功能：从 INI 文件读取默认配置，或禁用文件读取
  - 关键点：headers、cookies、auth、params、verify、cert、stream、trust_env、max_redirects 等按需设置；proxies 从 INI 读取；timeout、download_path 同步

- 属性（只读）
  - download_path：默认下载路径
  - timeout：超时秒数
  - proxies：代理字典 {http, https}
  - retry_times/retry_interval：重试次数与间隔
  - headers：请求头字典（大小写不敏感）
  - cookies：cookies 列表
  - auth：认证信息
  - hooks、params、verify、cert、adapters、stream、trust_env、max_redirects：其他会话参数

- 配置设置与修改
  - set_download_path(path) / set_timeout(second)：基础设置
  - set_proxies(http=None, https=None)：设置代理
  - set_retry(times=None, interval=None)：重试设置
  - set_headers(headers)：设置请求头；传入 None 标记从 INI 删除
  - set_a_header(name, value) / remove_a_header(name) / clear_headers()：单个头管理
  - set_cookies(cookies)：设置 cookies（支持多种输入类型）
  - set_auth(auth) / set_hooks(hooks) / set_params(params)：其他参数设置
  - set_verify/on/off_cert/add_adapter/set_stream/set_trust_env/set_max_redirects：SSL、适配器、流式响应等设置
  - _sets(arg, val)：内部通用设置与删除标记

- 会话构建与导出
  - as_dict()：以字典形式返回当前配置
  - make_session()：创建 requests.Session 并分离 headers
  - from_session(session, headers=None)：从现有 Session 读取配置

- 保存与默认配置
  - save(path=None)：保存到指定路径或当前读取的 INI；支持传入 'default'
  - save_to_default()：保存到默认 INI
  - 返回值：保存文件的绝对路径

- 使用示例（路径）
  - 基础使用与参数设置：[session_options.py:20-372](file://DrissionPage/_configs/session_options.py#L20-L372)
  - 保存配置到 INI：[session_options.py:268-317](file://DrissionPage/_configs/session_options.py#L268-L317)

章节来源
- [session_options.py:20-372](file://DrissionPage/_configs/session_options.py#L20-L372)
- [session_options.pyi:19-299](file://DrissionPage/_configs/session_options.pyi#L19-L299)

### OptionsManager API 参考
- 初始化
  - 构造函数：OptionsManager(path=None)
  - 功能：确定 INI 文件路径；若不存在则创建默认模板（包含 paths、chromium_options、session_options、timeouts、proxies、others）
  - 默认模板字段：见 [configs.ini:1-34](file://DrissionPage/_configs/configs.ini#L1-L34)

- 读取与写入
  - get_value(section, item)：获取某项值（尝试 eval 解析）
  - get_option(section)：获取整节内容为字典
  - set_item(section, item, value)：设置某项值（字符串化）
  - remove_item(section, item)：删除某项
  - save(path=None)：保存 INI 文件；支持 'default'
  - save_to_default()：保存到默认 INI
  - show()：打印所有节与内容

- 使用示例（路径）
  - INI 读取与默认模板初始化：[options_manage.py:15-143](file://DrissionPage/_configs/options_manage.py#L15-L143)
  - 默认配置模板：[configs.ini:1-34](file://DrissionPage/_configs/configs.ini#L1-L34)

章节来源
- [options_manage.py:15-143](file://DrissionPage/_configs/options_manage.py#L15-L143)
- [options_manage.pyi:13-78](file://DrissionPage/_configs/options_manage.pyi#L13-L78)
- [configs.ini:1-34](file://DrissionPage/_configs/configs.ini#L1-L34)

## 依赖关系分析
- ChromiumOptions 与 SessionOptions 依赖 OptionsManager 进行 INI 文件读写
- OptionsManager 依赖 configparser.RawConfigParser 与 pathlib.Path
- 默认配置模板由 OptionsManager 在首次创建时写入
- 外部导出：__init__.py 导出 ChromiumOptions 与 SessionOptions

```mermaid
graph LR
CO["ChromiumOptions"] --> OM["OptionsManager"]
SO["SessionOptions"] --> OM
OM --> CFG["configs.ini"]
INIT["__init__.py 导出"] --> CO
INIT --> SO
```

图表来源
- [chromium_options.py:11-41](file://DrissionPage/_configs/chromium_options.py#L11-L41)
- [session_options.py:14-38](file://DrissionPage/_configs/session_options.py#L14-L38)
- [options_manage.py:16-78](file://DrissionPage/_configs/options_manage.py#L16-L78)
- [__init__.py:22-28](file://DrissionPage/__init__.py#L22-L28)

章节来源
- [chromium_options.py:11-41](file://DrissionPage/_configs/chromium_options.py#L11-L41)
- [session_options.py:14-38](file://DrissionPage/_configs/session_options.py#L14-L38)
- [options_manage.py:16-78](file://DrissionPage/_configs/options_manage.py#L16-L78)
- [__init__.py:22-28](file://DrissionPage/__init__.py#L22-L28)

## 性能考量
- 配置读取：OptionsManager 在初始化时一次性读取 INI 文件，后续通过内存字典访问，避免重复 IO
- 配置写入：save/save_to_default 将内存配置批量写入，减少磁盘写入次数
- 字符串化与 eval：get_value/get_option 对字符串进行 eval 解析，注意配置项应为安全的 Python 字面量
- 代理与 UA：set_proxy 会解析代理 URL、用户名与密码，建议在运行时统一设置以减少重复解析

## 故障排查指南
- INI 文件不存在
  - 现象：构造 ChromiumOptions/SessionOptions 时抛出“未找到 INI”异常
  - 处理：确认 ini_path 存在，或使用默认配置模板
  - 参考：[chromium_options.py:34-36](file://DrissionPage/_configs/chromium_options.py#L34-L36)、[session_options.py:33-35](file://DrissionPage/_configs/session_options.py#L33-L35)

- 无效的 load_mode 值
  - 现象：set_load_mode 抛出值错误异常
  - 处理：仅接受 'normal' | 'eager' | 'none'
  - 参考：[chromium_options.py:299-303](file://DrissionPage/_configs/chromium_options.py#L299-L303)

- 保存 INI 未设置路径
  - 现象：OptionsManager.save 抛出“未设置 INI”异常
  - 处理：确保 ini_path 已正确设置或传入 'default'
  - 参考：[options_manage.py:115-118](file://DrissionPage/_configs/options_manage.py#L115-L118)

- 代理格式不正确
  - 现象：set_proxy 无法解析代理
  - 处理：确保传入 URL 或 scheme://username:password@host:port 格式
  - 参考：[chromium_options.py:293-297](file://DrissionPage/_configs/chromium_options.py#L293-L297)

章节来源
- [chromium_options.py:34-36](file://DrissionPage/_configs/chromium_options.py#L34-L36)
- [session_options.py:33-35](file://DrissionPage/_configs/session_options.py#L33-L35)
- [options_manage.py:115-118](file://DrissionPage/_configs/options_manage.py#L115-L118)
- [chromium_options.py:299-303](file://DrissionPage/_configs/chromium_options.py#L299-L303)
- [chromium_options.py:293-297](file://DrissionPage/_configs/chromium_options.py#L293-L297)

## 结论
配置管理模块通过 OptionsManager 统一管理 INI 文件，ChromiumOptions 与 SessionOptions 分别面向浏览器与 HTTP 会话场景，提供完善的配置读取、运行时修改、持久化与默认模板能力。遵循本文档的 API 规范与最佳实践，可实现稳定、可维护的配置管理。

## 附录

### 配置优先级与作用域管理
- 优先级顺序（从高到低）
  1) 运行时显式设置（如 set_* 方法）
  2) INI 文件中对应节与键值
  3) 默认模板（首次创建时写入）
- 作用域
  - ChromiumOptions：影响浏览器启动与运行期行为
  - SessionOptions：影响 HTTP 请求行为
  - OptionsManager：全局 INI 文件作用域
- 配置合并
  - 保存时：ChromiumOptions/SessionOptions 将当前内存配置写入 INI 对应节
  - 删除标记：SessionOptions 支持对某些项在 INI 中标记删除（如 headers=None）

章节来源
- [options_manage.py:16-78](file://DrissionPage/_configs/options_manage.py#L16-L78)
- [chromium_options.py:364-417](file://DrissionPage/_configs/chromium_options.py#L364-L417)
- [session_options.py:268-317](file://DrissionPage/_configs/session_options.py#L268-L317)

### 配置迁移指导
- 新增字段：在默认模板中添加新字段后，旧 INI 文件不会自动包含新字段，需手动迁移或重新生成默认模板
- 字段变更：若字段类型或含义变更，建议在 OptionsManager 中增加兼容解析逻辑
- 路径迁移：download_path、tmp_path 等路径字段建议使用绝对路径，避免相对路径在不同工作目录下行为不一致

章节来源
- [configs.ini:1-34](file://DrissionPage/_configs/configs.ini#L1-L34)
- [options_manage.py:46-78](file://DrissionPage/_configs/options_manage.py#L46-L78)