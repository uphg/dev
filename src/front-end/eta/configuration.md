# 配置选项

```ts
type config = {
  /** 是否自动对插值进行 XML 转义。默认为 true */
  autoEscape: boolean
  /** 对每个插值或原始插值应用类上定义的过滤器函数 */
  autoFilter: boolean
  /** 配置自动空白修剪。默认 `[false, 'nl']` */
  autoTrim: trimConfig | [trimConfig, trimConfig]
  /** 如果传入了 `name` 或 `filename`，是否缓存模板 */
  cache: boolean
  /** 保存已解析文件路径的缓存。设置为 `false` 可禁用 */
  cacheFilepaths: boolean
  /** 是否美化错误消息格式（会引入运行时性能损耗） */
  debug: boolean
  /** 用于对插值进行 XML 消毒的函数 */
  escapeFunction: (str: unknown) => string
  /** 当 autoFilter 为 true 时，应用于所有插值的函数 */
  filterFunction: (val: unknown) => string
  /** 插入模板函数中的原始 JS 代码。用于为用户模板声明全局变量 */
  functionHeader: string
  /** 解析选项 */
  parse: {
    /** 用于求值的前缀。默认 `""`，不支持 `"-"` 或 `"_"` */
    exec: string
    /** 用于插值的前缀。默认 `"="`，不支持 `"-"` 或 `"_"` */
    interpolate: string
    /** 用于原始插值的前缀。默认 `"~"`，不支持 `"-"` 或 `"_"` */
    raw: string
  }
  /** 插件数组 */
  plugins: Array<{
    processFnString?: Function
    processAST?: Function
    processTemplate?: Function
  }>
  /** 移除所有可安全移除的空白字符 */
  rmWhitespace: boolean
  /** 分隔符：默认为 `['<%', '%>']` */
  tags: [string, string]
  /** 使数据在全局对象上可用，而不是通过 varName */
  useWith: boolean
  /** 数据对象的名称。默认 `it` */
  varName: string
  /** 包含模板的目录 */
  views?: string
  /** 控制模板文件扩展名的默认值。默认 `.eta` */
  defaultExtension?: string;
}
```