---
title: Obsidian + Claude Code + BDD 实践探索
author: 小二的学习基地
date: 
url: https://mp.weixin.qq.com/s?__biz=MzIyMjExNTUwNg==&mid=2247483657&idx=1&sn=e4de415bb9b0343c5655a279e504ba85&chksm=e93c86e1e9eee3d08c4564aa06c3631986c212b039d3d563a5a7ec03bf6325db33b3f8e06edc&mpshare=1&scene=24&srcid=1118bITvXWfpBn92BmPpSlh7&sharer_shareinfo=c281ca49871462b74e73722fbcd0e891&sharer_shareinfo_first=c281ca49871462b74e73722fbcd0e891#rd
---

**适用对象**: 产品经理、设计师、开发者  
**核心理念**: 用行为驱动开发思维，让需求文档变成可执行的代码

***1、核心观点***

## 

***1.1***

### 

****🎯 为什么需要 Obsidian + Claude Code + BDD？****

**传统痛点**

> * 需求文档与代码脱节
> * 沟通成本高，理解偏差大
> * 测试用例难以追溯需求
> * 文档更新不及时

**解决方案**

> * **Obsidian**: 作为知识管理和需求文档平台
> * **BDD**: 提供结构化的需求描述语言
> * **Claude Code**: 将 BDD 场景直接转换为可执行代码

***1.2***

### 

****💡 核心价值****

> * **需求即测试**: BDD 场景既是需求文档，也是测试用例
> * **降低沟通成本**: Given-When-Then 语法让所有人都能读懂
> * **提高开发效率**: Claude Code 可以直接根据 BDD 场景生成代码
> * **文档永不过时**: 代码和文档同源，自动保持同步

***2、**BDD的Given-When-Then结构*****

## 

***2.1***

### 

****📐 结构说明****

BDD (Behavior-Driven Development，行为驱动开发) 使用自然语言描述系统行为，核心是 **Given-When-Then** 三段式结构：

```
Feature: 功能名称  功能的业务描述Scenario: 场景名称    Given [前置条件] - 描述初始状态    When [触发动作] - 描述用户操作    Then [预期结果] - 描述期望的结果    And [额外条件] - 补充说
```

***2.2***

### 

****🔍 详细解释****

|  |  |  |  |
| --- | --- | --- | --- |
| 关键词 | 含义 | 作用 | 示例 |
| Given | 前置条件 | 设定场景的初始状态 | Given 用户已登录系统 |
| When | 触发动作 | 描述用户的具体操作 | When 用户点击"添加到购物车"按钮 |
| Then | 预期结果 | 描述期望的系统响应 | Then 购物车商品数量应该增加 1 |
| And | 补充条件 | 补充前面的 Given/When/Then | And 显示成功提示消息 |
| But | 相反条件 | 描述相反的情况 | But 不应该扣除库存 |

***2.3***

### 

****🌟 编写原则****

> * **使用业务语言**: 避免技术术语，让 PM 和设计师也能理解
> * **保持简洁**: 每个场景专注于一个核心行为
> * **可验证性**: 每个 Then 语句应该可以被测试验证
> * **独立性**: 每个场景应该能独立运行

***3、实践案例***

## 

***3.1***

### 

****案例 1：电商购物车功能****

****📝 需求文档（Obsidian 中编写）****

```
Feature: 购物车管理  作为一个电商用户  我希望能够管理我的购物车  以便我可以选择要购买的商品  
Scenario: 添加商品到购物车    Given 用户已登录    And 商品"iPhone 15 Pro"有库存    And 购物车中有 0 件商品    When 用户访问商品详情页    And 选择数量为 2    And 点击"加入购物车"按钮    Then 购物车中应该有 2 件商品    And 显示提示消息"已成功添加到购物车"    And 购物车图标显示数字"2"    Scenario: 移除购物车中的商品    Given 用户已登录    And 购物车中有 3 件"iPhone 15 Pro"    When 用户在购物车页面点击商品的删除按钮    And 确认删除操作    Then 该商品应该从购物车中移除    And 购物车商品数量显示为"0"    And 显示"购物车为空"的提示    Scenario: 修改购物车商品数量    Given 用户已登录    And 购物车中有 1 件"iPhone 15 Pro"    And 该商品库存充足    When 用户将数量修改为 5    And 点击更新按钮    Then 购物车中该商品数量应该为 5    And 总价应该更新为对应的金额    And 显示"已更新"提示    Scenario: 库存不足时添加商品    Given 用户已登录    And 商品"限量版手办"仅剩 2 件库存    And 购物车中有 0 件商品    When 用户尝试添加 5 件到购物车    Then 应该显示错误消息"库存不足，仅剩 2 件"    And 购物车数量保持不变    But 用户仍可以选择添加 2 件或更少
```

****💻 Claude Code 生成的代码示例****

```
// javascript// cart.service.jsclass CartService {  constructor() {    this.cart = [];  }  /**   * Scenario: 添加商品到购物车   * Given: 用户已登录, 商品有库存, 购物车为空   * When: 用户添加商品   * Then: 购物车更新，显示提示   */  addToCart(product, quantity, userLoggedIn, stockAvailable) {    // Given: 验证前置条件    if (!userLoggedIn) {      throw new Error('用户未登录');    }  
    if (stockAvailable < quantity) {      return {        success: false,        message: `库存不足，仅剩 ${stockAvailable} 件`      };    }    // When: 执行添加操作    const existingItem = this.cart.find(item => item.id === product.id);  
    if (existingItem) {      existingItem.quantity += quantity;    } else {      this.cart.push({        id: product.id,        name: product.name,        price: product.price,        quantity: quantity      });    }    // Then: 返回预期结果    return {      success: true,      message: '已成功添加到购物车',      cartCount: this.getCartCount(),      items: this.cart    };  }  /**   * Scenario: 移除购物车中的商品   */  removeFromCart(productId) {    const index = this.cart.findIndex(item => item.id === productId);  
    if (index !== -1) {      this.cart.splice(index, 1);      return {        success: true,        message: this.cart.length === 0 ? '购物车为空' : '商品已移除',        cartCount: this.getCartCount()      };    }  
    return {      success: false,      message: '商品不存在'    };  }  /**   * Scenario: 修改购物车商品数量   */  updateQuantity(productId, newQuantity, stockAvailable) {    const item = this.cart.find(item => item.id === productId);  
    if (!item) {      return { success: false, message: '商品不存在' };    }  
    if (newQuantity > stockAvailable) {      return { success: false, message: '库存不足' };    }  
    item.quantity = newQuantity;  
    return {      success: true,      message: '已更新',      totalPrice: this.getTotalPrice(),      items: this.cart    };  }  getCartCount() {    return this.cart.reduce((sum, item) => sum + item.quantity, 0);  }  getTotalPrice() {    return this.cart.reduce((sum, item) => sum + (item.price * item.quantity), 0);  }}module.exports = CartService;
```

****🧪 测试代码****

```
// javascript// cart.service.test.jsconst CartService = require('./cart.service');describe('购物车管理功能', () => {  let cartService;  
  beforeEach(() => {    cartService = new CartService();  });  describe('Scenario: 添加商品到购物车', () => {    test('Given 用户已登录, 商品有库存 When 添加商品 Then 购物车更新', () => {      // Given      const product = { id: 1, name: 'iPhone 15 Pro', price: 7999 };      const userLoggedIn = true;      const stockAvailable = 10;  
      // When      const result = cartService.addToCart(product, 2, userLoggedIn, stockAvailable);  
      // Then      expect(result.success).toBe(true);      expect(result.message).toBe('已成功添加到购物车');      expect(result.cartCount).toBe(2);      expect(cartService.cart).toHaveLength(1);    });    test('Given 库存不足 When 添加超过库存的数量 Then 显示错误', () => {      // Given      const product = { id: 2, name: '限量版手办', price: 299 };      const userLoggedIn = true;      const stockAvailable = 2;  
      // When      const result = cartService.addToCart(product, 5, userLoggedIn, stockAvailable);  
      // Then      expect(result.success).toBe(false);      expect(result.message).toBe('库存不足，仅剩 2 件');      expect(cartService.getCartCount()).toBe(0);    });  });  describe('Scenario: 移除购物车中的商品', () => {    test('Given 购物车有商品 When 删除商品 Then 商品被移除', () => {      // Given      const product = { id: 1, name: 'iPhone 15 Pro', price: 7999 };      cartService.addToCart(product, 3, true, 10);  
      // When      const result = cartService.removeFromCart(1);  
      // Then      expect(result.success).toBe(true);      expect(result.message).toBe('购物车为空');      expect(result.cartCount).toBe(0);    });  });  describe('Scenario: 修改购物车商品数量', () => {    test('Given 购物车有商品 When 修改数量 Then 数量和总价更新', () => {      // Given      const product = { id: 1, name: 'iPhone 15 Pro', price: 7999 };      cartService.addToCart(product, 1, true, 10);  
      // When      const result = cartService.updateQuantity(1, 5, 10);  
      // Then      expect(result.success).toBe(true);      expect(result.message).toBe('已更新');      expect(result.totalPrice).toBe(39995);      expect(cartService.cart[0].quantity).toBe(5);    });  });});
```

***3.2***

### 

****案例 2：用户权限管理****

****📝 需求文档****

```
Feature: 用户权限管理  作为系统管理员  我希望能够控制用户的访问权限  以便保护敏感数据和功能  
Scenario: 普通用户访问公开页面    Given 用户"张三"已登录    And 用户角色为"普通用户"    When 用户访问"首页"    Then 页面应该正常显示    And 显示用户导航菜单  
Scenario: 普通用户访问管理后台    Given 用户"张三"已登录    And 用户角色为"普通用户"    When 用户尝试访问"管理后台"    Then 应该显示"403 禁止访问"错误    And 跳转到错误页面    And 记录未授权访问日志    Scenario: 管理员访问管理后台    Given 用户"李四"已登录    And 用户角色为"管理员"    When 用户访问"管理后台"    Then 页面应该正常显示    And 显示完整的管理功能菜单    And 可以查看所有用户数据    Scenario: 分配权限给用户    Given 管理员已登录    And 用户"王五"存在于系统中    And 用户"王五"当前角色为"普通用户"    When 管理员选择用户"王五"    And 将角色修改为"编辑者"    And 点击保存按钮    Then 用户"王五"的角色应该更新为"编辑者"    And 显示"权限更新成功"提示    And 发送邮件通知用户"王五"    Scenario: 权限过期后自动降级    Given 用户"赵六"的"VIP 权限"将在今天过期    And 当前时间为过期时间后 1 小时    When 系统执行定时任务    Then 用户"赵六"的角色应该降级为"普通用户"    And 发送邮件通知"您的 VIP 权限已过期"    And 在用户下次登录时显示续费提示
```

****💻 Claude Code 生成的代码示例****

```
# python# auth_service.pyfrom datetime import datetimefrom enum import Enumfrom typing import Optional, Dict, Listclass Role(Enum):    """用户角色枚举"""    USER = "普通用户"    EDITOR = "编辑者"    ADMIN = "管理员"    VIP = "VIP用户"class Permission(Enum):    """权限枚举"""    VIEW_PUBLIC = "查看公开内容"    VIEW_ADMIN = "查看管理后台"    EDIT_CONTENT = "编辑内容"    MANAGE_USERS = "管理用户"  
# 角色权限映射ROLE_PERMISSIONS = {    Role.USER: [Permission.VIEW_PUBLIC],    Role.EDITOR: [Permission.VIEW_PUBLIC, Permission.EDIT_CONTENT],    Role.ADMIN: [Permission.VIEW_PUBLIC, Permission.EDIT_CONTENT,                  Permission.VIEW_ADMIN, Permission.MANAGE_USERS],    Role.VIP: [Permission.VIEW_PUBLIC, Permission.EDIT_CONTENT]}  
class User:    """用户模型"""    def __init__(self, name: str, role: Role, permission_expiry: Optional[datetime] = None):        self.name = name        self.role = role        self.permission_expiry = permission_expiry        self.is_logged_in = False        class AuthService:    """    权限管理服务    基于 BDD 场景实现    """  
    def __init__(self):        self.users: Dict[str, User] = {}        self.access_logs: List[Dict] = []  
    def check_access(self, user: User, page: str) -> Dict:        """        Scenario: 检查用户访问权限        Given: 用户已登录并具有特定角色        When: 用户访问页面        Then: 返回访问结果        """        # Given: 验证前置条件        if not user.is_logged_in:            return {                "allowed": False,                "error": "用户未登录",                "redirect": "/login"            }  
        # 检查权限是否过期        if user.permission_expiry and datetime.now() > user.permission_expiry:            self._downgrade_user(user)  
        # When: 判断页面访问权限        required_permission = self._get_required_permission(page)        user_permissions = ROLE_PERMISSIONS.get(user.role, [])  
        # Then: 返回访问结果        if required_permission in user_permissions:            return {                "allowed": True,                "page": page,                "menu": self._get_user_menu(user.role)            }        else:            # 记录未授权访问            self._log_unauthorized_access(user, page)            return {                "allowed": False,                "error": "403 禁止访问",                "redirect": "/error/403"            }  
    def update_user_role(self, admin: User, target_user_name: str,                         new_role: Role) -> Dict:        """        Scenario: 管理员分配权限        Given: 管理员已登录，目标用户存在        When: 管理员修改用户角色        Then: 角色更新并发送通知        """        # Given: 验证管理员权限        if admin.role != Role.ADMIN:            return {                "success": False,                "message": "无权限执行此操作"            }  
        target_user = self.users.get(target_user_name)        if not target_user:            return {                "success": False,                "message": "用户不存在"            }  
        # When: 更新角色        old_role = target_user.role        target_user.role = new_role  
        # Then: 返回结果并发送通知        self._send_notification(            target_user_name,            f"您的角色已从 {old_role.value} 更新为 {new_role.value}"        )  
        return {            "success": True,            "message": "权限更新成功",            "user": target_user_name,            "old_role": old_role.value,            "new_role": new_role.value        }  
    def check_and_downgrade_expired_permissions(self) -> List[Dict]:        """        Scenario: 权限过期自动降级        Given: 用户权限有过期时间        When: 系统执行定时任务        Then: 过期用户降级并通知        """        downgraded_users = []        current_time = datetime.now()  
        for user_name, user in self.users.items():            # Given: 检查是否有过期时间设置            if user.permission_expiry and current_time > user.permission_expiry:                # When: 权限已过期                old_role = user.role  
                # Then: 降级为普通用户                user.role = Role.USER                user.permission_expiry = None  
                # 发送通知                self._send_notification(                    user_name,                    f"您的 {old_role.value} 权限已过期，已降级为普通用户"                )  
                downgraded_users.append({                    "user": user_name,                    "old_role": old_role.value,                    "new_role": Role.USER.value,                    "message": "权限已过期并降级"                })  
        return downgraded_users  
    def _get_required_permission(self, page: str) -> Permission:        """根据页面获取所需权限"""        page_permissions = {            "首页": Permission.VIEW_PUBLIC,            "管理后台": Permission.VIEW_ADMIN,            "内容编辑": Permission.EDIT_CONTENT,            "用户管理": Permission.MANAGE_USERS        }        return page_permissions.get(page, Permission.VIEW_PUBLIC)  
    def _get_user_menu(self, role: Role) -> List[str]:        """根据角色获取菜单"""        menus = {            Role.USER: ["首页", "个人中心"],            Role.EDITOR: ["首页", "个人中心", "内容编辑"],            Role.ADMIN: ["首页", "个人中心", "内容编辑", "管理后台", "用户管理"],            Role.VIP: ["首页", "个人中心", "VIP 专区"]        }        return menus.get(role, [])  
    def _log_unauthorized_access(self, user: User, page: str):        """记录未授权访问日志"""        self.access_logs.append({            "user": user.name,            "role": user.role.value,            "page": page,            "timestamp": datetime.now(),            "status": "未授权访问"        })  
    def _send_notification(self, user_name: str, message: str):        """发送通知（模拟）"""        print(f"📧 发送邮件给 {user_name}: {message}")  
    def _downgrade_user(self, user: User):        """降级用户权限"""        user.role = Role.USER        user.permission_expiry = None        
```

****🧪 测试代码****

```
# python# test_auth_service.pyimport pytestfrom datetime import datetime, timedeltafrom auth_service import AuthService, User, Roleclass TestAuthService:    """基于 BDD 场景的测试"""  
    @pytest.fixture    def auth_service(self):        service = AuthService()        # 准备测试数据        service.users = {            "张三": User("张三", Role.USER),            "李四": User("李四", Role.ADMIN),            "王五": User("王五", Role.USER),            "赵六": User("赵六", Role.VIP,                        permission_expiry=datetime.now() - timedelta(hours=1))        }        # 模拟已登录        for user in service.users.values():            user.is_logged_in = True        return service  
    def test_普通用户访问公开页面(self, auth_service):        """        Scenario: 普通用户访问公开页面        Given: 用户"张三"已登录，角色为"普通用户"        When: 用户访问"首页"        Then: 页面应该正常显示        """        # Given        user = auth_service.users["张三"]        assert user.role == Role.USER        assert user.is_logged_in == True  
        # When        result = auth_service.check_access(user, "首页")  
        # Then        assert result["allowed"] == True        assert result["page"] == "首页"        assert "首页" in result["menu"]  
    def test_普通用户访问管理后台(self, auth_service):        """        Scenario: 普通用户访问管理后台        Given: 用户"张三"已登录，角色为"普通用户"        When: 用户尝试访问"管理后台"        Then: 应该显示"403 禁止访问"错误        """        # Given        user = auth_service.users["张三"]  
        # When        result = auth_service.check_access(user, "管理后台")  
        # Then        assert result["allowed"] == False        assert result["error"] == "403 禁止访问"        assert result["redirect"] == "/error/403"        assert len(auth_service.access_logs) > 0  
    def test_管理员访问管理后台(self, auth_service):        """        Scenario: 管理员访问管理后台        Given: 用户"李四"已登录，角色为"管理员"        When: 用户访问"管理后台"        Then: 页面应该正常显示，显示完整管理菜单        """        # Given        user = auth_service.users["李四"]        assert user.role == Role.ADMIN  
        # When        result = auth_service.check_access(user, "管理后台")  
        # Then        assert result["allowed"] == True        assert "管理后台" in result["menu"]        assert "用户管理" in result["menu"]  
    def test_分配权限给用户(self, auth_service):        """        Scenario: 分配权限给用户        Given: 管理员已登录，用户"王五"当前角色为"普通用户"        When: 管理员将角色修改为"编辑者"        Then: 用户角色更新，显示成功提示        """        # Given        admin = auth_service.users["李四"]        target_user = auth_service.users["王五"]        assert target_user.role == Role.USER  
        # When        result = auth_service.update_user_role(admin, "王五", Role.EDITOR)  
        # Then        assert result["success"] == True        assert result["message"] == "权限更新成功"        assert target_user.role == Role.EDITOR        assert result["new_role"] == "编辑者"  
    def test_权限过期后自动降级(self, auth_service):        """        Scenario: 权限过期后自动降级        Given: 用户"赵六"的 VIP 权限已过期        When: 系统执行定时任务        Then: 用户降级为普通用户并发送通知        """        # Given        user = auth_service.users["赵六"]        assert user.role == Role.VIP        assert user.permission_expiry < datetime.now()  
        # When        downgraded = auth_service.check_and_downgrade_expired_permissions()  
        # Then        assert len(downgraded) == 1        assert downgraded[0]["user"] == "赵六"        assert downgraded[0]["new_role"] == "普通用户"        assert user.role == Role.USER        assert user.permission_expiry is None
```

***3.3***

### 

****案例 3：Claude Code 应用开发****

****📝 需求文档****

```
Feature: Claude Code 对话功能  作为开发者  我希望能够与 Claude Code 进行对话  以便获得编程帮助和代码生成  
Scenario: 发送简单的代码请求    Given 用户已打开 Claude Code 界面    And 对话历史为空    When 用户输入"写一个 Python 函数计算斐波那契数列"    And 点击发送按钮    Then 应该显示加载动画    And Claude 应该返回包含代码的响应    And 代码应该包含在代码块中    And 响应时间应该少于 10 秒  
Scenario: 上下文对话    Given 用户已与 Claude 进行过 2 轮对话    And 前一轮对话是关于"React 组件"    When 用户输入"如何测试这个组件？"    And 点击发送按钮    Then Claude 应该理解"这个组件"指的是之前的 React 组件    And 返回相关的测试代码    And 对话历史应该包含 3 轮对话    Scenario: 代码高亮显示    Given Claude 返回了包含代码的响应    And 代码语言为 JavaScript    When 页面渲染响应内容    Then 代码块应该有语法高亮    And 应该显示语言标签"JavaScript"    And 应该有复制按钮    And 点击复制按钮应该复制代码到剪贴板    Scenario: 错误处理    Given 用户已输入消息    When 网络请求失败    Then 应该显示错误提示"网络连接失败，请重试"    And 应该显示重试按钮    And 点击重试按钮应该重新发送请求    And 用户的输入不应该丢失    Scenario: 流式响应显示    Given 用户已发送消息    When Claude 开始返回流式响应    Then 响应应该逐字显示    And 每收到一个 token 就更新界面    And 显示打字机效果    And 在响应完成前显示"生成中..."状态
```

****💻 Claude Code 生成的代码示例****

```
// typescript// claude-chat.service.ts  
interface Message {  role: 'user' | 'assistant';  content: string;  timestamp: Date;  codeBlocks?: CodeBlock[];}  
interface CodeBlock {  language: string;  code: string;  id: string;}  
interface ChatResponse {  success: boolean;  message?: Message;  error?: string;  isStreaming?: boolean;}  
class ClaudeChatService {  private conversationHistory: Message[] = [];  private isProcessing: boolean = false;  private abortController: AbortController | null = null;  /**   * Scenario: 发送简单的代码请求   * Given: 用户已打开界面，对话历史为空   * When: 用户输入消息并发送   * Then: 显示加载，返回代码响应   */  async sendMessage(    userInput: string,    onProgress?: (chunk: string) => void  ): Promise<ChatResponse> {    // Given: 验证前置条件    if (!userInput.trim()) {      return {        success: false,        error: '请输入消息'      };    }    if (this.isProcessing) {      return {        success: false,        error: '正在处理中，请稍候'      };    }    // When: 发送请求    this.isProcessing = true;    this.abortController = new AbortController();    // 添加用户消息到历史    const userMessage: Message = {      role: 'user',      content: userInput,      timestamp: new Date()    };    this.conversationHistory.push(userMessage);    try {      const startTime = Date.now();      // 调用 Claude API (流式)      const response = await this.callClaudeAPI(        this.conversationHistory,        onProgress      );      const responseTime = Date.now() - startTime;      // Then: 验证响应时间和内容      if (responseTime > 10000) {        console.warn('响应时间超过 10 秒:', responseTime);      }      // 解析代码块      const codeBlocks = this.extractCodeBlocks(response.content);      const assistantMessage: Message = {        role: 'assistant',        content: response.content,        timestamp: new Date(),        codeBlocks      };      this.conversationHistory.push(assistantMessage);      return {        success: true,        message: assistantMessage      };    } catch (error) {      // Scenario: 错误处理      return this.handleError(error, userInput);    } finally {      this.isProcessing = false;      this.abortController = null;    }  }  /**   * Scenario: 上下文对话   * Given: 用户已进行过多轮对话   * When: 用户发送新消息   * Then: Claude 理解上下文并返回相关响应   */  async sendContextualMessage(userInput: string): Promise<ChatResponse> {    // Given: 检查对话历史    const previousContext = this.conversationHistory.slice(-4); // 最近 2 轮  
    console.log(`上下文对话 - 历史记录: ${previousContext.length} 条消息`);  
    // When: 发送带上下文的请求    const response = await this.sendMessage(userInput);  
    // Then: 验证对话历史更新    if (response.success) {      const totalRounds = Math.floor(this.conversationHistory.length / 2);      console.log(`对话历史已更新 - 共 ${totalRounds} 轮对话`);    }  
    return response;  }  /**   * Scenario: 流式响应显示   * Given: 用户已发送消息   * When: Claude 返回流式响应   * Then: 逐字显示打字机效果   */  private async callClaudeAPI(    messages: Message[],    onProgress?: (chunk: string) => void  ): Promise<{ content: string }> {    let fullContent = '';    // 模拟流式 API 调用    const response = await fetch('/api/claude/stream', {      method: 'POST',      headers: { 'Content-Type': 'application/json' },      body: JSON.stringify({ messages }),      signal: this.abortController?.signal    });    if (!response.ok) {      throw new Error(`HTTP ${response.status}: ${response.statusText}`);    }    const reader = response.body?.getReader();    const decoder = new TextDecoder();    if (!reader) {      throw new Error('无法读取响应流');    }    // Then: 流式处理响应    while (true) {      const { done, value } = await reader.read();  
      if (done) break;      const chunk = decoder.decode(value, { stream: true });      fullContent += chunk;      // 触发进度回调 - 打字机效果      if (onProgress) {        onProgress(chunk);      }    }    return { content: fullContent };  }  /**   * Scenario: 代码高亮显示   * 提取代码块用于语法高亮   */  private extractCodeBlocks(content: string): CodeBlock[] {    const codeBlockRegex = /```(\w+)?\n([\s\S]*?)```/g;    const blocks: CodeBlock[] = [];    let match;    while ((match = codeBlockRegex.exec(content)) !== null) {      blocks.push({        language: match[1] || 'plaintext',        code: match[2].trim(),        id: this.generateId()      });    }    return blocks;  }  /**   * Scenario: 错误处理   * Given: 请求失败   * Then: 返回错误信息和重试选项   */  private handleError(error: any, userInput: string): ChatResponse {    console.error('请求失败:', error);    let errorMessage = '未知错误';    if (error.name === 'AbortError') {      errorMessage = '请求已取消';    } else if (error.message.includes('Failed to fetch')) {      errorMessage = '网络连接失败，请重试';    } else {      errorMessage = error.message || '请求失败';    }    // 保留用户输入，允许重试    return {      success: false,      error: errorMessage,      message: {        role: 'user',        content: userInput,        timestamp: new Date()      }    };  }  /**   * 重试最后一条消息   */  async retryLastMessage(onProgress?: (chunk: string) => void): Promise<ChatResponse> {    const lastUserMessage = this.conversationHistory      .filter(m => m.role === 'user')      .pop();    if (!lastUserMessage) {      return {        success: false,        error: '没有可重试的消息'      };    }    // 移除最后一轮对话（用户消息 + 可能的失败响应）    this.conversationHistory = this.conversationHistory.filter(      m => m.timestamp < lastUserMessage.timestamp    );    return this.sendMessage(lastUserMessage.content, onProgress);  }  /**   * 获取对话历史   */  getConversationHistory(): Message[] {    return [...this.conversationHistory];  }  /**   * 清空对话历史   */  clearHistory(): void {    this.conversationHistory = [];  }  private generateId(): string {    return `code-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;  }}export default ClaudeChatService;
```

****🎨 React 组件示例****

```
// typescript// ClaudeChatComponent.tsx  
import React, { useState, useRef, useEffect } from 'react';import ClaudeChatService from './claude-chat.service';import { Prism as SyntaxHighlighter } from 'react-syntax-highlighter';import { vscDarkPlus } from 'react-syntax-highlighter/dist/esm/styles/prism';interface Message {  role: 'user' | 'assistant';  content: string;  timestamp: Date;  codeBlocks?: CodeBlock[];}interface CodeBlock {  language: string;  code: string;  id: string;}const ClaudeChatComponent: React.FC = () => {  const [messages, setMessages] = useState<Message[]>([]);  const [input, setInput] = useState('');  const [isLoading, setIsLoading] = useState(false);  const [error, setError] = useState<string | null>(null);  const [streamingContent, setStreamingContent] = useState('');  
  const chatService = useRef(new ClaudeChatService());  const messagesEndRef = useRef<HTMLDivElement>(null);  /**   * Scenario: 发送简单的代码请求   * 处理用户发送消息   */  const handleSendMessage = async () => {    if (!input.trim() || isLoading) return;    // Given: 用户已输入消息    const userInput = input;    setInput(''); // 清空输入框    setError(null);    setStreamingContent('');    // When: 点击发送按钮    setIsLoading(true); // Then: 显示加载动画    try {      const response = await chatService.current.sendMessage(        userInput,        // Scenario: 流式响应显示 - 打字机效果        (chunk) => {          setStreamingContent(prev => prev + chunk);        }      );      // Then: 更新消息列表      if (response.success && response.message) {        setMessages(chatService.current.getConversationHistory());        setStreamingContent('');      } else if (response.error) {        // Scenario: 错误处理        setError(response.error);      }    } catch (err) {      setError('发送失败，请重试');    } finally {      setIsLoading(false);    }  };  /**   * Scenario: 错误处理 - 重试   */  const handleRetry = async () => {    setError(null);    setIsLoading(true);    try {      const response = await chatService.current.retryLastMessage(        (chunk) => setStreamingContent(prev => prev + chunk)      );      if (response.success) {        setMessages(chatService.current.getConversationHistory());        setStreamingContent('');      } else if (response.error) {        setError(response.error);      }    } catch (err) {      setError('重试失败');    } finally {      setIsLoading(false);    }  };  /**   * Scenario: 代码高亮显示   * 渲染代码块组件   */  const renderCodeBlock = (block: CodeBlock) => {    const handleCopy = () => {      navigator.clipboard.writeText(block.code);      // 可以添加复制成功提示    };    return (      <div key={block.id} className="code-block-container">        {/* Then: 显示语言标签 */}        <div className="code-header">          <span className="language-label">{block.language}</span>          {/* Then: 显示复制按钮 */}          <button onClick={handleCopy} className="copy-button">            📋 复制          </button>        </div>        {/* Then: 代码应该有语法高亮 */}        <SyntaxHighlighter          language={block.language}          style={vscDarkPlus}          customStyle={{ margin: 0, borderRadius: '0 0 8px 8px' }}        >          {block.code}        </SyntaxHighlighter>      </div>    );  };  /**   * 渲染消息内容   */  const renderMessageContent = (message: Message) => {    const parts = message.content.split(/(```[\s\S]*?```)/g);  
    return parts.map((part, index) => {      if (part.startsWith('```')) {        const block = message.codeBlocks?.find(b =>           part.includes(b.code)        );        return block ? renderCodeBlock(block) : null;      }      return <p key={index}>{part}</p>;    });  };  // 自动滚动到底部  useEffect(() => {    messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' });  }, [messages, streamingContent]);  return (    <div className="claude-chat-container">      {/* 对话历史区域 */}      <div className="messages-container">        {messages.map((msg, index) => (          <div key={index} className={`message ${msg.role}`}>            <div className="message-header">              <strong>{msg.role === 'user' ? '👤 您' : '🤖 Claude'}</strong>              <span className="timestamp">                {msg.timestamp.toLocaleTimeString()}              </span>            </div>            <div className="message-content">              {renderMessageContent(msg)}            </div>          </div>        ))}        {/* Scenario: 流式响应显示 - 打字机效果 */}        {streamingContent && (          <div className="message assistant streaming">            <div className="message-header">              <strong>🤖 Claude</strong>              <span className="status">生成中...</span>            </div>            <div className="message-content">              <p>{streamingContent}</p>            </div>          </div>        )}        {/* Scenario: 错误处理 - 显示错误和重试按钮 */}        {error && (          <div className="error-container">            <p className="error-message">❌ {error}</p>            <button onClick={handleRetry} className="retry-button">              🔄 重试            </button>          </div>        )}        {/* Then: 显示加载动画 */}        {isLoading && !streamingContent && (          <div className="loading-indicator">            <div className="spinner"></div>            <span>思考中...</span>          </div>        )}        <div ref={messagesEndRef} />      </div>      {/* 输入区域 */}      <div className="input-container">        <textarea          value={input}          onChange={(e) => setInput(e.target.value)}          onKeyDown={(e) => {            if (e.key === 'Enter' && !e.shiftKey) {              e.preventDefault();              handleSendMessage();            }          }}          placeholder="输入消息... (Shift+Enter 换行)"          disabled={isLoading}          className="message-input"        />        <button          onClick={handleSendMessage}          disabled={isLoading || !input.trim()}          className="send-button"        >          {isLoading ? '发送中...' : '发送'}        </button>      </div>    </div>  );};export default ClaudeChatComponent;
```

***4、**完整工作流示例*****

## 

***4.1***

### 

****📊 从需求到上线的完整流程****

```
graph TD    A[产品需求] --> B[在 Obsidian 中编写 BDD 场景]    B --> C[团队 Review 场景]    C --> D{场景是否清晰?}    D -->|否| B    D -->|是| E[使用 Claude Code 生成代码]    E --> F[生成测试用例]    F --> G[运行测试]    G --> H{测试是否通过?}    H -->|否| I[修复代码]    I --> G    H -->|是| J[代码 Review]    J --> K[部署到测试环境]    K --> L[验收测试]    L --> M{验收是否通过?}    M -->|否| N[更新 BDD 场景]    N --> B    M -->|是| O[部署到生产环境]    O --> P[更新 Obsidian 文档]
```

***4.2***

### 

****🔄 实际操作步骤****

****第 1 步：在 Obsidian 中创建需求文档****

创建文件：features/用户登录.md

```
# 用户登录功能## 功能描述用户可以通过邮箱和密码登录系统## BDD 场景### Scenario: 成功登录Given 用户"test@example.com"已注册And 密码为"Password123!"When 用户访问登录页面And 输入邮箱"test@example.com"And 输入密码"Password123!"And 点击"登录"按钮Then 应该跳转到首页And 显示欢迎消息"欢迎回来，test@example.com"And 在导航栏显示用户头像### Scenario: 密码错误Given 用户"test@example.com"已注册And 正确密码为"Password123!"When 用户输入密码"WrongPassword"And 点击"登录"按钮Then 应该显示错误消息"邮箱或密码错误"And 保持在登录页面And 输入框应该清空密码And 登录失败次数加 1## 关联需求- [[用户注册]]- [[密码重置]]## 技术实现- 使用 JWT Token- 密码使用 bcrypt 加密- 失败 5 次后锁定账号 30 分钟## 测试清单- [ ] 单元测试- [ ] 集成测试- [ ] E2E 测试- [ ] 安全测试
```

****第 2 步：将 BDD 场景发送给 Claude Code****

在 Claude Code 中输入：

```
根据以下 BDD 场景，生成完整的登录功能代码：[粘贴上面的 BDD 场景]请生成：1. 登录服务类 (login.service.ts)2. 单元测试 (login.service.test.ts)3. API 路由 (login.controller.ts)4. 前端组件 (LoginForm.tsx)
```

****第 3 步：Claude 生成代码并自动关联 BDD 场景****

Claude 会生成带有 BDD 注释的代码，确保每个功能点都可追溯到原始需求。

****第 4 步：运行测试验证****

```
npm test -- login.service.test.ts
```

测试输出会清楚地显示每个 BDD 场景的测试结果：

```
✓ Scenario: 成功登录  ✓ Given 用户已注册  ✓ When 用户输入正确的凭证  ✓ Then 应该返回 JWT Token  ✓ And 跳转到首页✓ Scenario: 密码错误  ✓ Given 用户已注册  ✓ When 用户输入错误密码  ✓ Then 应该返回错误消息  ✓ And 登录失败次数加 1
```

****第 5 步：更新 Obsidian 文档****

在原文档中添加实现状态：

```
## 实现状态- [x] 成功登录场景 - 2025-11-10- [x] 密码错误场景 - 2025-11-10- [x] 单元测试覆盖率 95%- [ ] E2E 测试（待完成）## 代码位置- 服务: `src/services/login.service.ts`- 测试: `src/services/login.service.test.ts`- 组件: `src/components/LoginForm.tsx`
```

***5、**价值对比*****

## 

***5.1***

### 

****📈 传统方法 vs BDD 方法****

| 对比维度 | 传统方法 | Obsidian + BDD + Claude Code |
| --- | --- | --- |
| **需求文档** | Word/Confluence，格式自由 | Markdown + Given-When-Then 结构化 |
| **开发理解** | 需要多次沟通确认 | 场景清晰，一次读懂 |
| **代码生成** | 手动编写，耗时长 | Claude Code 自动生成，提效 80% |
| **测试用例** | 单独编写，容易遗漏 | 基于场景自动生成，覆盖完整 |
| **文档同步** | 容易过时，需手动更新 | 代码与文档同源，自动同步 |
| **团队协作** | PM→开发→测试 串行流程 | 所有人基于同一份 BDD 场景并行工作 |
| **需求变更** | 改文档→改代码→改测试，多处修改 | 更新 BDD 场景，重新生成代码 |
| **新人上手** | 需要理解多份文档 | 阅读 BDD 场景即可理解业务 |
| **可追溯性** | 需求与代码难以关联 | 代码注释直接引用 BDD 场景 |
| **自动化程度** | 低，大量手动工作 | 高，从需求到代码高度自动化 |

***5.2***

### 

****💰 效率提升数据****

基于实际项目经验的估算：

| 环节 | 传统耗时 | BDD 方法耗时 | 提效比例 |
| --- | --- | --- | --- |
| 需求澄清 | 2 小时 | 30 分钟 | 75% ⬇️ |
| 编写代码 | 4 小时 | 1 小时 | 75% ⬇️ |
| 编写测试 | 2 小时 | 30 分钟 | 75% ⬇️ |
| 代码 Review | 1 小时 | 30 分钟 | 50% ⬇️ |
| Bug 修复 | 3 小时 | 1 小时 | 66% ⬇️ |
| 文档更新 | 1 小时 | 10 分钟 | 83% ⬇️ |
| **总计** | **13 小时** | **3.5 小时** | **73% ⬇️** |

***5.3***

### 

****🎯 质量提升指标****

| 指标 | 传统方法 | BDD 方法 | 改善幅度 |
| --- | --- | --- | --- |
| 需求理解偏差率 | 30% | 5% | 83% ⬇️ |
| 测试覆盖率 | 65% | 95%+ | 46% ⬆️ |
| 生产环境 Bug 率 | 15/千行 | 3/千行 | 80% ⬇️ |
| 文档过时率 | 60% | 0% | 100% ⬇️ |
| 新人上手时间 | 2 周 | 3 天 | 78% ⬇️ |

***6、**给 PM/设计师的启发*****

## 

***6.1***

### 

****🎨 为什么非技术人员也应该学习 BDD？****

****1. **统一沟通语言******

**问题**：

> * PM 说"用户可以添加商品"
> * 开发理解为"无限制添加"
> * 实际需求是"库存范围内添加"

**BDD 解决方案**：

```
Scenario: 添加商品到购物车  Given 商品库存为 5 件  When 用户尝试添加 10 件  Then 应该显示"库存不足"错误  And 仅允许添加最多 5 件
```

明确的 Given-When-Then 让所有人理解一致。

****2. **场景化思维训练******

BDD 强制你从**用户场景**出发，而不是功能列表：

**传统 PRD**：

```
需求：用户可以发表评论功能：- 输入框- 提交按钮- 字数限制 500 字
```

**BDD 思维**：

```
Scenario: 普通用户发表评论  Given 用户已登录  And 评论内容为 100 字  When 点击"发表"按钮  Then 评论应该立即显示  And 显示"发表成功"提示Scenario: 游客尝试发表评论  Given 用户未登录  When 点击评论框  Then 应该弹出登录提示  And 评论内容应该被保存（登录后恢复）
```

看出区别了吗？BDD 让你考虑**更多边界情况**。

****3. **设计稿的可验证性******

**设计师的痛点**：

> * 设计稿和最终实现不一致
> * 交互细节被开发忽略
> * 无法验证实现是否符合设计

**BDD 解决方案**：设计师可以编写"交互场景"作为验收标准：

```
Feature: 按钮交互动画Scenario: 按钮 Hover 效果  Given 用户鼠标移动到"提交"按钮上方  Then 按钮背景色应该在 0.3 秒内过渡到 #007AFF  And 鼠标光标变为 pointer  And 按钮阴影应该增强Scenario: 按钮点击反馈  Given 用户点击"提交"按钮  When 按钮被按下的瞬间  Then 按钮应该有 0.1 秒的缩放动画（scale 0.95）  And 显示涟漪效果从点击位置扩散  And 触发触觉反馈（移动端）
```

开发可以根据这些场景编写 **E2E 测试**来验证设计还原度。

****4. **提前发现产品逻辑漏洞******

在编写 BDD 场景时，你会被迫思考：

```
Scenario: 用户购买会员后立即取消  Given 用户购买了年度会员  And 支付成功后 5 分钟内  When 用户点击"取消订阅"  Then ？？？  # 钱退不退？会员立即失效还是用到期？
```

这些问题在写代码前就能发现，避免返工。

****5. **用户故事的升级版******

BDD 场景 = 可执行的用户故事

**传统用户故事**：

```
作为用户，我希望能够筛选商品，以便快速找到想要的内容
```

**BDD 场景（可执行）**：

```
Feature: 商品筛选  作为用户，我希望能够筛选商品  
Scenario: 按价格区间筛选    Given 页面显示 50 件商品    When 用户选择价格区间"100-500 元"    And 点击"应用"按钮    Then 应该只显示价格在 100-500 元的商品    And 显示筛选结果数量"共 12 件商品"    And URL 应该更新为 ?price=100-500    Scenario: 多条件组合筛选    Given 用户已选择品牌"Apple"    When 用户再选择价格"5000 元以上"    Then 应该显示符合两个条件的商品    And 筛选标签显示"Apple × 5000 元以上"
```

后者可以直接转换为测试用例和代码。

***6.2***

### 

****📚 PM/设计师如何开始？****

****Step 1：学会 Given-When-Then 思维****

练习用三段式描述日常场景：

```
Scenario: 早上买咖啡  Given 我有 50 元现金  And 咖啡店正常营业  When 我点了一杯拿铁（35 元）  Then 我应该收到一杯拿铁  And 找回 15 元零钱  And 我的现金余额为 15 元
```

****Step 2：在 Obsidian 中建立 BDD 模板****

创建模板文件：templates/BDD-场景模板.md

```
# 功能名称## 业务价值作为 [角色]，我希望 [功能]，以便 [目标]## 核心场景### Scenario: 场景名称**Given** [前置条件]**When** [触发动作]**Then** [预期结果]## 边界场景### Scenario: 异常情况**Given** [异常条件]**When** [用户操作]**Then** [系统响应]## 验收标准- [ ] 场景 1 通过- [ ] 场景 2 通过- [ ] UI/UX 还原度 > 95%## 设计稿![[设计稿链接]]## 相关文档- [[相关需求]]- [[技术方案]]
```

****Step 3：与开发协作 Review BDD 场景****

> * PM 编写初版 BDD 场景
> * 开发 Review 并补充技术细节
> * 设计师补充交互场景
> * 测试人员补充边界场景

最终得到一份**所有人都认可的场景文档**。

****Step 4：使用 BDD 场景作为验收标准****

在项目管理工具（Jira/Notion）中，直接引用 BDD 场景：

```
验收标准：✅ Scenario: 成功登录（已通过）✅ Scenario: 密码错误（已通过）❌ Scenario: 账号被锁定（测试失败，返工中）
```

***6.3***

### 

****💡 关键提示****

**不要害怕技术术语**  
Given-When-Then 是自然语言，不需要编程基础

**从简单场景开始**  
先写"正常流程"，再逐步补充"异常情况"

**多用具体例子**  
"Given 用户余额为 100 元" 比 "Given 用户有余额" 更清晰

**场景独立性**  
每个场景应该能独立运行，不依赖其他场景

**与开发结对编写**  
初期可以和开发一起写，学习如何描述得更精确

***7、**🎯 总结*****

## 

***7.1***

### 

****核心要点****

**BDD 不只是开发方法，更是思维方式**  
用结构化的方式思考产品，减少沟通成本

**Obsidian 是知识管理的中枢**  
所有 BDD 场景、设计稿、技术文档都关联在一起

**Claude Code 是生产力放大器**  
将清晰的需求场景转换为高质量代码

**从需求到代码的无缝衔接**  
文档即代码，代码即文档，永不过时

***7.2***

### 

****适用场景****

✅ **适合**：

> * 需求频繁变更的项目
> * 团队协作复杂的项目
> * 需要高测试覆盖率的项目
> * 需要长期维护的项目

❌ **不适合**：

> * 一次性的 Demo 项目
> * 需求极其简单的项目
> * 单人独立开发的小工具

***7.3***

### 

****下一步行动****

> * 在 Obsidian 中创建第一个 BDD 场景
> * 使用 Claude Code 生成对应代码
> * 运行测试验证场景
> * 在团队中推广这套工作流

***8、**📖 延伸阅读*****

## 

> * BDD 官方文档 ( https://cucumber.io/docs/bdd/ )
> * Obsidian 官方文档 ( https://help.obsidian.md/ )
> * Given-When-Then 最佳实践 ( https://martinfowler.com/bliki/GivenWhenThen.html )
> * 如何编写好的 BDD 场景 ( https://automationpanda.com/2017/01/30/bdd-101-writing-good-gherkin/ )

---

**记住**：BDD 的价值在于**协作**和**沟通**，不只是技术实践！

如果您有任何问题或改进建议，欢迎沟通交流~~~