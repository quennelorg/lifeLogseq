- JWT（JSON Web Token）是一种紧凑的、URL 安全的令牌格式，用于在各方之间传递经过签名和加密的数据。JWT 通常用于认证和授权系统中，用于安全地传输信息。
	- ![image.png](../assets/image_1717163863989_0.png)
	- ![image.png](../assets/image_1717226367763_0.png)
- ### JWT 的结构
  collapsed:: true
	- JWT 由三个部分组成，每个部分之间用点（`.`）分隔：
		- 1. **Header（头部）**：包含令牌的类型（JWT）和签名算法（例如 HMAC SHA256 或 RSA）。
		- 2. **Payload（负载）**：包含声明（claims），即传递的数据。声明可以包括预定义的标准声明（如 `iss`、`exp`、`sub` 等），也可以包括自定义声明。
		- 3. **Signature（签名）**：用于验证令牌的发送方和数据的完整性。
		- JWT 示例：
		  ```
		  eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
		  ```
		- **Header**：`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9`
		- **Payload**：`eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ`
		- **Signature**：`SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c`
- ### JWT 的组成部分详细说明
  collapsed:: true
	- collapsed:: true
	  1. **Header**
		- ```json
		   {
		     "alg": "HS256",
		     "typ": "JWT"
		   }
		   ```
		- `alg`：签名算法（如 HMAC SHA256）。
		- `typ`：令牌类型，通常为 JWT。
	- collapsed:: true
	  2. **Payload**
		- ```json
		  {
		  "sub": "1234567890",
		  "name": "John Doe",
		  "iat": 1516239022
		  }
		  ```
		- `sub`（Subject）：令牌面向的用户。
		- `name`：用户的名字。
		- `iat`（Issued At）：令牌的签发时间（时间戳）。
	- collapsed:: true
	  3. **Signature**
		- ```
		  HMACSHA256(
		  base64UrlEncode(header) + "." + base64UrlEncode(payload),
		  secret
		  )
		  ```
		- 签名是对 header 和 payload 进行编码并拼接起来，然后用指定的算法和密钥进行哈希生成的。
- ### JWT 的工作原理
	- 1. **客户端请求登录**：用户在客户端登录并向服务器发送凭据（例如用户名和密码）。
	- 2. **服务器验证凭据**：服务器验证用户凭据是否正确。如果验证通过，服务器生成一个 JWT。
	- 3. **服务器返回 JWT**：服务器将生成的 JWT 返回给客户端。
	- 4. **客户端存储 JWT**：客户端通常会将 JWT 存储在本地存储（如 `localStorage` 或 `sessionStorage`）。
	- 5. **客户端请求受保护资源**：客户端每次请求受保护的资源时，都在请求头中包含 JWT（通常在 `Authorization` 头中，格式为 `Bearer <token>`）。
	- 6. **服务器验证 JWT**：服务器验证 JWT 的签名和有效性。如果验证通过，服务器允许访问受保护资源。
- ### JWT 的优点
  collapsed:: true
	- **紧凑和 URL 安全**：JWT 的结构紧凑，可以通过 URL、安全地在 HTTP 请求和响应中传递。
	- **自包含**：JWT 包含了所有必要的信息，不需要在服务器上存储会话数据，方便分布式系统中的无状态认证。
	- **易于使用**：JWT 可以方便地生成、解析和验证，有许多现成的库和工具可供使用。
- ### JWT 的缺点
  collapsed:: true
	- **不易撤销**：一旦 JWT 被签发，直到过期时间到达之前，无法单独撤销某个令牌的权限。
	- **负载数据的大小限制**：由于 JWT 负载包含所有声明信息，负载的数据量不宜过大，否则会影响传输效率。
- ### 示例代码
	- #### 生成和验证 JWT 的示例代码（Node.js 和 `jsonwebtoken` 库）
	  collapsed:: true
		- ```javascript
		  const jwt = require('jsonwebtoken');
		  
		  // 秘钥
		  const secretKey = 'your-256-bit-secret';
		  
		  // 生成 JWT
		  const token = jwt.sign({ userId: 123, role: 'admin' }, secretKey, { expiresIn: '1h' });
		  console.log('Generated JWT:', token);
		  
		  // 验证 JWT
		  jwt.verify(token, secretKey, (err, decoded) => {
		  if (err) {
		    console.error('JWT verification failed:', err);
		  } else {
		    console.log('Decoded JWT payload:', decoded);
		  }
		  });
		  ```
- ### 总结
	- JWT 是一种用于认证和授权的紧凑、安全的令牌格式，适用于现代 Web 应用程序。它允许在各方之间安全地传递信息，并且由于其无状态的特点，非常适合分布式系统。尽管 JWT 具有诸多优点，但在使用过程中需要注意安全性，特别是在令牌管理和撤销机制上。