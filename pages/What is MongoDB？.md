#### MongoDB 是一种文档数据库，它所具备的可扩展性和灵活性可以满足您对查询和索引的需求
-
- ## IDEA
	- Based document
		- 存储数据在documets（field-value pair data structures， NoSQL）
	- Scalable
		- 非常容易部署数据在不同的平台
	- Flexible
		- no document data schema required
		- 如果你想，每个文档都可以有不同的字段
	- Performant
		- 都基于文档了，随意的定义字段，性能上肯定优于SQL，特别是可以通过横向拓展，副本集等
- ## 官方描述
	- ### 1. 面向文档的存储
		- MongoDB使用类似于JSON（JavaScript Object Notation）格式的BSON（Binary JSON）来存储数据。每个记录称为一个文档，文档是以键值对（key-value pair）的形式组织的。这种结构使得数据模式非常灵活，可以方便地进行嵌套和复杂的数据表示。
	- ### 2. 可扩展性和高性能
		- MongoDB设计之初就考虑到了横向扩展（scaling out），支持通过分片（sharding）来扩展数据库的容量和性能。分片是一种将数据分散到多个服务器上的技术，确保在数据量和用户访问量增加时，数据库仍然能够高效地运行。
	- ### 3. 灵活的查询语言
		- MongoDB提供了强大的查询语言，支持丰富的查询操作，包括过滤、排序、聚合、索引和地理空间查询等。开发者可以使用这些功能来构建复杂的数据检索和分析应用。
	- ### 4. 高可用性和自动故障恢复，负载平衡
		- 通过副本集（replica set），MongoDB实现了数据的高可用性和自动故障恢复。副本集由多个MongoDB实例组成，其中一个是主节点（primary），其余的是副节点（secondary）。主节点处理所有的写操作，而副节点实时复制主节点的数据，并在主节点故障时自动选举出新的主节点，保证服务的连续性。
	- ### 5. JSON-like 数据模型
		- MongoDB使用的BSON格式与JSON非常相似，使得它在处理Web应用程序的数据时非常自然和高效。这种文档模型非常适合处理复杂的嵌套结构和动态模式的数据。
- ### 应用场景
	- MongoDB适用于多种应用场景，特别是那些需要处理大规模、快速变化的数据的应用，例如：
		- **内容管理系统（CMS）**：因为其灵活的数据模式，MongoDB非常适合用于存储和管理复杂的内容数据。
		- **大数据和实时分析**：MongoDB的高性能和可扩展性使得它能够处理大数据集和实时数据分析。
		- **物联网（IoT）**：MongoDB可以有效地存储和处理来自各种设备的大量数据。
		- **社交网络和用户数据管理**：由于其灵活的模式和强大的查询功能，MongoDB适合存储和分析社交网络数据和用户行为数据。
- 总之，MongoDB作为一种NoSQL数据库，提供了与传统关系型数据库不同的存储和管理数据的方式，具有高度的灵活性和扩展性，适用于现代应用程序的多种数据需求。
-
- | SQL术语/概念 | MongoDB术语/概念 | 解释/说明 |
  | --- | --- | --- |
  | database | database | 数据库 |
  | table | collection | 数据库表/集合 |
  | row | document | 数据记录行/文档 |
  | column | field | 数据字段/域 |
  | index | index | 索引 |
  | table joins |  | 表连接,MongoDB不支持 |
  | primary key | primary key | 主键,MongoDB自动将_id字段设置为主键 |