## 标准字段
	这个章节描述了一系列定义近似概念时候应该被使用的标准消息字段。这可以用来保证跨不同APIs之间的相同概念有着相同的名称和语义。
	* name
		* string
		* name字段应该包含相对资源名
	* parent
		* string
		* 在资源定义和List/Create请求里，parent应该包含其父级的相对资源名
	* create_time
		* Timestamp
		* 一个实体的创建时间戳
	* update_time
		* Timestamp
		* 一个实体最后一次被修改的时间戳. 提示: update_time会在create/patch/delete操作发生作用时被更新.
	* delete_time
		* Timestamp
		* 一个实体的被删除时间。只有支持还原操作时存在.
	* expire_time
		* Timestamp
		* 一个实体的过期时间戳，如果它会过期的话。
	* start_time
		* Timestamp
		* 标记某个时间段开始的时间戳。
	* end_time
		* Timestamp
		* 标记某个时间段或操作结束的时间戳(不管成功与否)。
	* read_time
		* Timestamp
		* 某个特定资源被获取（在请求中被使用）或者被读取（在响应中使用）的时间戳。
	* time_zone
		* string
		* 时区名称. 它应该是一个 IANA TZ 名称，形如"America/Los_Angeles”。更多信息，请查看 https://en.wikipedia.org/wiki/List_of_tz_database_time_zones。
	* region_code
		* string
		* 一个地区的Unicode country/region code (CLDR) ，形如”US" and "419”。更多信息，请查看 http://www.unicode.org/reports/tr35/#unicode_region_subtag。
	* language_code
		* string
		* The BCP-47 language code，形如”en-US”或者"sr-Latn”。更多信息，请查看 http://www.unicode.org/reports/tr35/#Unicode_locale_identifier。
	* mime_type
		* string
		* 一个IANA发布的MIME类型（也被称为媒体类型）。更多信息，请查看 https://www.iana.org/assignments/media-types/media-types.xhtml.
	* display_name
		* string
		* 一个实体的显示名称。
	* title
		* string
		* 一个实体的官方名称，例如公司名字。 它应该被视为display_name的正式版本。
	* description
		* string
		* 一个实体的一段或多段文字描述。
	* filter
		* string
		* List方法的标准过滤参数。
	* query
		* string
		* 如果应用于搜索方法，则与过滤器相同（即：搜索）
	* page_token
		* string
		* List请求中的分页标记。
	* page_size
		* int32
		* List请求中的分页大小。
	* total_size
		* int32
		* 不分页时列表中的项目总数。
	* next_page_token
		* string
		* 列表响应中的下一个分页标记。 它应该作为page_token用于下一个请求。 一个空值意味着没有更多的结果。
	* order_by
		* string
		* 指定List请求结果的排序规则。
	* request_id
		* string
		* 一个用于检测重复请求的唯一字符串。
	* resume_token
		* string
		* 用于恢复流媒体请求的不透明令牌。
	* labels
		* map<string, string>
		* 代表云资源标签。
	* deleted
		* bool
		* 如果某个资源允许取消删除行为，则必须有一个deleted的字段，指示该资源是否已被删除。
	* show_deleted
		* bool
		* 如果某个资源允许取消删除行为，则对应的List方法必须有一个show_deleted的参数，这样客户端才能查看那些被删除的资源。
	* update_mask
		* FieldMask
		* 它用于更新请求消息，以便对资源执行部分更新。 该掩码是相对于资源而不是请求消息。
	* validate_only
		* bool
		* 如果为true，则表示只想验证给定的请求，而并不实际执行。
