---
layout: post
title: mysql数据库逆向java pojo脚本
categories: 脚本
keywords: java spring 脚本
---

# mysql数据库逆向java pojo脚本
### 

>groovy不熟悉,修改脚本非常费力.在网上找到一个契合度比较高的模板,稍作修改.记录一下脚本.

```groovy
import com.intellij.database.model.DasTable
import com.intellij.database.model.ObjectKind
import com.intellij.database.util.Case
import com.intellij.database.util.DasUtil

import java.io.*
import java.text.SimpleDateFormat

/*
 * Available context bindings:
 *   SELECTION   Iterable<DasObject>
 *   PROJECT     project
 *   FILES       files helper
 */
packageName = "com.weng.zebra.model.po;"
classSufix ="PO"
typeMapping = [
        (~/(?i)tinyint|smallint|mediumint|int/)  : "Integer",
        (~/(?i)bigint/)                          : "Long",
        (~/(?i)bool|bit/)                        : "Boolean",
        (~/(?i)float/)                           : "Float",
        (~/(?i)decimal/)                         : "BigDecimal",
        (~/(?i)double|real/)                     : "Double",
        (~/(?i)timestamp/)                       : "Date",
        (~/(?i)datetime|date|time/)              : "Date",
        (~/(?i)blob|binary|bfile|clob|raw|image/): "InputStream",
        (~/(?i)/)                                : "String"
]


FILES.chooseDirectoryAndSave("Choose directory", "Choose where to store generated files") { dir ->
    SELECTION.filter { it instanceof DasTable && it.getKind() == ObjectKind.TABLE }.each { generate(it, dir) }
}

def generate(table, dir) {
    def className = javaClassName(table.getName(), true)
    def fields = calcFields(table)
//    packageName = getPackageName(dir)
    PrintWriter printWriter = new PrintWriter(new OutputStreamWriter(new FileOutputStream(new File(dir, className + classSufix+".java")), "UTF-8"))
    printWriter.withPrintWriter { out -> generate(out, className, fields, table) }

//    new File(dir, className + ".java").withPrintWriter { out -> generate(out, className, fields,table) }
}

// 获取包所在文件夹路径
def getPackageName(dir) {
    return dir.toString().replaceAll("\\\\", ".").replaceAll("/", ".").replaceAll("^.*src(\\.main\\.java\\.)?", "") + ";"
}
/// 输出文件内容
def generate(out, className, fields, table) {
    out.println "package $packageName"
    out.println ""
    out.println "import com.baomidou.mybatisplus.annotation.IdType;"
    out.println "import com.baomidou.mybatisplus.annotation.TableId;"
    out.println "import com.baomidou.mybatisplus.annotation.TableName;"
    out.println "import java.io.Serializable;"
    out.println "import lombok.Getter;"
    out.println "import lombok.Setter;"
    Set types = new HashSet()

    fields.each() {
        types.add(it.type)
    }

    if (types.contains("Date")) {
        out.println "import java.util.Date;"
    }
    if (types.contains("BigDecimal")) {
        out.println "import java.math.BigDecimal;"
    }

    if (types.contains("InputStream")) {
        out.println "import java.io.InputStream;"
    }
    out.println ""
    out.println "/**\n" +
            " * ${table.comment}\n" +
            " * @author auto\n" +
            " * @date " + new SimpleDateFormat("yyyy-MM-dd").format(new Date()) + " \n" +
            " */"
    out.println "@Data"
    out.println "@Accessors(chain = true)"
    out.println "@JsonInclude(JsonInclude.Include.NON_EMPTY)"
    out.println "@TableName(\"" + table.getName() + "\" )"
    out.println "public class ${className}${classSufix}  implements Serializable {"
    out.println ""
    out.println genSerialID()
    fields.each() {
        out.println ""
        // 输出注释
        if (isNotEmpty(it.comment)) {
            out.println "\t/** ${it.comment.toString()} */"
        }
        // 输出注解
        if (it.annos != "") out.println "${it.annos}"

        // 输出成员变量
        out.println "\tprivate ${it.type} ${it.name};"
    }

    // 输出get/set方法
//    fields.each() {
//        out.println ""
//        out.println "\tpublic ${it.type} get${it.name.capitalize()}() {"
//        out.println "\t\treturn this.${it.name};"
//        out.println "\t}"
//        out.println ""
//
//        out.println "\tpublic void set${it.name.capitalize()}(${it.type} ${it.name}) {"
//        out.println "\t\tthis.${it.name} = ${it.name};"
//        out.println "\t}"
//    }
    out.println ""
    out.println "}"
}

def calcFields(table) {
    DasUtil.getColumns(table).reduce([]) { fields, col ->
        def spec = Case.LOWER.apply(col.getDataType().getSpecification())

        def typeStr = typeMapping.find { p, t -> p.matcher(spec).find() }.value
        def comm = [
                colName: col.getName(),
                name   : javaName(col.getName(), false),
                type   : typeStr,
                comment: col.getComment(),
                annos  : ""]
        //annos   : "\t@Column(name = \"" + col.getName() + "\" )"]
        if ("id".equals(Case.LOWER.apply(col.getName())))
            comm.annos += "\t@TableId(type = IdType.AUTO)\n"
        if("Date".equals(typeStr))
            comm.annos += "\t@JsonFormat(shape = JsonFormat.Shape.NUMBER_INT)\n"
        def colname = col.getName()
        def comment = col.getComment()
        if("created_time".equals(colname) || "updated_time".equals(colname) || "updated_by".equals(colname) ){
            comm.annos += "\t@ApiModelProperty(hidden = true)\n"
        }else {
            comm.annos += "\t@ApiModelProperty(value = \"${comment}\")\n"
        }
        if (comm.annos != "")
            comm.annos = comm.annos[0..-2]

        fields += [comm]
    }
}

// 处理类名（这里是因为我的表都是以t_命名的，所以需要处理去掉生成类名时的开头的T，
// 如果你不需要那么请查找用到了 javaClassName这个方法的地方修改为 javaName 即可）
def javaClassName(str, capitalize) {
    def s = com.intellij.psi.codeStyle.NameUtil.splitNameIntoWords(str)
            .collect { Case.LOWER.apply(it).capitalize() }
            .join("")
            .replaceAll(/[^\p{javaJavaIdentifierPart}[_]]/, "_")
    // 去除开头的T  http://developer.51cto.com/art/200906/129168.htm
    //删除前缀,前缀ze 所以从2开始截取
    s = s[2.. - 1]
    capitalize || s.length() == 1 ? s : Case.LOWER.apply(s[0]) + s[1..-1]
}

//这个用于 参数
def javaName(str, capitalize) {
    def s = com.intellij.psi.codeStyle.NameUtil.splitNameIntoWords(str)
            .collect { Case.LOWER.apply(it).capitalize() }
            .join("")
            .replaceAll(/[^\p{javaJavaIdentifierPart}[_]]/, "_")
    capitalize || s.length() == 1 ? s : Case.LOWER.apply(s[0]) + s[1..-1]
}

def isNotEmpty(content) {
    return content != null && content.toString().trim().length() > 0
}

static String changeStyle(String str, boolean toCamel) {
    if (!str || str.size() <= 1)
        return str

    if (toCamel) {
        String r = str.toLowerCase().split('_').collect { cc -> Case.LOWER.apply(cc).capitalize() }.join('')
        return r[0].toLowerCase() + r[1..-1]
    } else {
        str = str[0].toLowerCase() + str[1..-1]
        return str.collect { cc -> ((char) cc).isUpperCase() ? '_' + cc.toLowerCase() : cc }.join('')
    }
}

static String genSerialID() {
    return "\tprivate static final long serialVersionUID =  " + Math.abs(new Random().nextLong()) + "L;"
}

```


