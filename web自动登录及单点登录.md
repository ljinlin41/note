1. 采用cookie
    1. 请求时自动附带cookie，服务端读取cookie，直接返回登录后的网页
    
2. 采用localStorge
    1. 由于请求时不会自动附带localStorage，因此get请求得到一个未登录的网页，网页中包含js代码，js读取本地localStorge，异步请求登录
    
3. 单点登录
    1. 部署一个单点登录服务器，当登录到应用服务器时，重定向到单点登录服务器。
    2. 登录流程
        1. 访问www.A.com, 由于未登录，跳转到登录服务器www.login.com，并带上重定向地址。www.login.com?redirect=www.A.com
        2. 在www.login.com登录后，客户端localStorage存储token，并跳转到www.A.com，并带上token。www.A.com?token=123
        3. www.A.com得到token，客户端localStorage存储token，并登录系统
        4. www.B.com访问系统，由于未登录，跳转到登录服务器www.login.com，并带上重定向地址。www.login.com?redirect=www.B.com
        5. 访问www.login.com后，发现已经登录，获取localStorage中的token，并跳转到www.B.com，并带上token。www.B.com?token=123
        6. 这样www.B.com就免登录获取到了token
        7. 本质上是所有的系统都通过www.login.com登录，token存在www.login.com的localStorage中，将token作为重定向链接的参数传给其他系统  
        8. 注意：每个域名间的localStorage不是同一个，同源策略。