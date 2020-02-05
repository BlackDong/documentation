<blockquote>
<p>Talk is cheap. Show me the code<br>
   —<em>Linus Torvalds</em>—</p>
</blockquote>
<blockquote>
<p><a href="http://www.baidu.com" title="Cornerstone Report">Cornerstone Report</a></p>
</blockquote>
<h2 id="关于报表设计的一点感想--font-size2-----技术框架font">关于报表设计的一点感想  <font size="2">—  技术框架</font></h2>
<p>一个基于spring boot /Maven构建的, spring cloud 微服务的, docker 容器化部署的, 前后端分离的, VUE渲染的, JQuery插件化的 报表工具.</p>
<h2 id="关于报表设计的一丝哲学--font-size2-----设计思想font">关于报表设计的一丝哲学  <font size="2">—  设计思想</font></h2>
<ul>
<li>认为所有报表均由三部分组成, 横向表头, 纵向表头, 数据区域</li>
<li>横向表头/纵向表头的数据结构为完美树</li>
<li>数据区域由单元格组成,  单元格由纵向表头/横向表头二重循环生成,其数据由编织函数进行计算</li>
<li>所有报表组成部分数据加载, 均可以通过手动定义, SQL查询, Web API接口等完成</li>
</ul>
<h2 id="逻辑部署图">逻辑部署图</h2>
<img src="http://cornerstone.org.cn:8100/png/TLBBRjD05DtdAsO94f5uVW0YHS0A2YiGYyIU1el8HXDRQeMG0ccS0AxJG9I8XAfOj6earAHLGn9eWZy6ptYooYzmxQ6LNIkpcD5zd7lEN9q2NjEHhY6Yv5_eI5O8bdH5KrEg9cF4VjRvGOr-wOdmZaPKJ8gwfk9L_4eZEdgTGaWgQ8QCdXf6uJnOCGX-cOKmc9PLymMCCbaz-rWpY8HZwKS69XiPlAQeIaYRg3sGSzBAEfQCH5b8M89DICtWefA7LA0nIsHnFOC8eMj_Is_fNcjUxj7VJR_PyqztlKbfNkfmjthxuNr_rg2LayMq5dnnCNtFTlPvtvst-zIi08PztQ8Vd73GTHYKwBRtMND4QqTyC6R7x-YslPXQONTul5ERdVIeQNcJG-gM0ITs1qXkZsczHUi3a1GXkH6GD8zNbLnEKxdRvOv5TwhSSVXmbyzc5mbepQJr1l13mVyny3kRj3BsPf_fdtvawCrW1MSPxzV7UNie_dyNmzO42oGyrw5xc_wswKqwq7Wbye-P2pBlRHHregm7IeRVHQSOoKSLrXxvPfEr3aFGBI1yOf2SecA_ss3sWUW3Nb38ZyemZ7wo1C3lJc2bpAgoRrK0Fq-zIS4vzoPAfuNxWWkbBrszd3upe7Z-vo8EZEKlMZ1PZ7imLa-OC5PTxiA8rgLD_m00">
## 功能交互设计
### 报表数据展示流程
```mermaid
sequenceDiagram
participant 浏览器
participant 报表微服务
participant 报表定义微服务
participant 查询执行器微服务
participant 查询定义微服务
participant 数据源管理微服务
participant 数据仓库
浏览器-&gt;&gt;+报表微服务: 发起报表查询请求
报表微服务-&gt;&gt;+报表定义微服务: 报表定义是否存在?
<p>报表定义微服务 --x 报表微服务: 返回空<br>
报表微服务 --x 浏览器: 显示错误信息<br>
报表定义微服务 --&gt;&gt;-报表微服务: 返回定义信息</p>
<p>报表微服务 -&gt;&gt;报表微服务: 获得报表定义中的查询定义</p>
<p>报表微服务 -&gt;&gt;+ 查询执行器微服务: 执行查询<br>
查询执行器微服务 -&gt;&gt;+ 查询定义微服务: 查询定义是否存在?</p>
<p>查询定义微服务 --x 查询执行器微服务: 返回空<br>
查询执行器微服务 --x 报表微服务: 错误信息<br>
报表微服务 --x 浏览器: 显示错误信息<br>
查询定义微服务 --&gt;&gt;-查询执行器微服务: 返回查询定义信息</p>
<p>查询执行器微服务 -&gt;&gt;查询执行器微服务: 获取数据源定义及组装查询语句</p>
<p>查询执行器微服务 -&gt;&gt;+ 数据源管理微服务: 数据源定义是否存在?</p>
<p>数据源管理微服务 --x 查询执行器微服务: 返回空<br>
查询执行器微服务 --x 报表微服务: 错误信息<br>
报表微服务 --x 浏览器: 显示错误信息<br>
数据源管理微服务 --&gt;&gt;- 查询执行器微服务: 返回数据源定义</p>
<p>查询执行器微服务 -&gt;&gt; 数据仓库: 测试数据库连接</p>
<p>查询执行器微服务 --x 报表微服务: 数据库连接失败<br>
报表微服务 --x 浏览器: 显示错误信息<br>
查询执行器微服务 -&gt;&gt; 查询执行器微服务: 建立数据库连接池</p>
<p>查询执行器微服务 -&gt;&gt;+ 数据仓库: 执行查询<br>
数据仓库 --&gt;&gt;- 查询执行器微服务: 返回数据<br>
查询执行器微服务 -&gt;&gt; 查询执行器微服务: 数据封装<br>
查询执行器微服务 --&gt;&gt;- 报表微服务: 返回结果<br>
报表微服务 --&gt;&gt;- 浏览器: 返回数据结果<br>
浏览器 --&gt;&gt; 浏览器: 渲染数据</p>
<pre><code>### 报表定义流程
```mermaid
sequenceDiagram
participant 浏览器
participant 报表微服务
participant 报表定义微服务
participant 查询定义微服务
participant 数据库
浏览器 -&gt;&gt;+ 报表微服务: 新建报表
报表微服务 -&gt;&gt; 报表微服务: 检查报表定义编码是否存在
报表微服务 --x 浏览器: 报表定义编码已存在
报表微服务 -&gt;&gt;+报表定义微服务: 新建报表定义
报表定义微服务-&gt;&gt; 数据库: 存储
报表定义微服务 --&gt;&gt;-报表微服务: 新建成功
报表微服务 --&gt;&gt;- 浏览器: 新建成功
浏览器 -&gt;&gt; 浏览器:定义报表表头
浏览器 -&gt;&gt; 浏览器:定义查询
浏览器 -&gt;&gt;+报表微服务: 保存查询定义
报表微服务 -&gt;&gt;+ 查询定义微服务: 保存查询定义
查询定义微服务 -&gt;&gt; 数据库: 存储
查询定义微服务 --x 报表微服务: 保存失败
报表微服务 --x 浏览器: 保存成功
查询定义微服务 --&gt;&gt;- 报表微服务: 保存成功
报表微服务 --&gt;&gt;-浏览器: 保存成功
</code></pre>
<h2 id="报表数据结构表示">报表数据结构表示</h2>
<h3 id="报表定义json结构">报表定义JSON结构</h3>
<h3 id="查询定义json结构">查询定义JSON结构</h3>
<h3 id="数据查询resultset结构">数据查询ResultSet结构</h3>

