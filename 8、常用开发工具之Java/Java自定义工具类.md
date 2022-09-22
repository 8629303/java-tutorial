# 姓名脱敏，复姓做特殊处理

> 只展示姓的姓名脱敏，针对复姓做特殊处理

```java
import com.google.common.collect.ImmutableList;
import org.apache.commons.lang3.StringUtils;

import java.util.ArrayList;
import java.util.List;


/**
 * 名字处理工具类
 *
 * @author Neo
 * @version 1.0
 * @since 2021/6/18 16:54
 */
public class NameUtils {

    public static List<String> DOUBLE_SURNAME = new ArrayList<>();

    public static final String PADDING_CHAR = "*";

    public static final int MAX_LENGTH = 5;

    static {
        DOUBLE_SURNAME.add("欧阳");
        DOUBLE_SURNAME.add("太史");
        DOUBLE_SURNAME.add("端木");
        DOUBLE_SURNAME.add("上官");
        DOUBLE_SURNAME.add("司马");
        DOUBLE_SURNAME.add("东方");
        DOUBLE_SURNAME.add("独孤");
        DOUBLE_SURNAME.add("南宫");
        DOUBLE_SURNAME.add("万俟");
        DOUBLE_SURNAME.add("闻人");
        DOUBLE_SURNAME.add("夏侯");
        DOUBLE_SURNAME.add("诸葛");
        DOUBLE_SURNAME.add("尉迟");
        DOUBLE_SURNAME.add("公羊");
        DOUBLE_SURNAME.add("赫连");
        DOUBLE_SURNAME.add("澹台");
        DOUBLE_SURNAME.add("皇甫");
        DOUBLE_SURNAME.add("宗政");
        DOUBLE_SURNAME.add("濮阳");
        DOUBLE_SURNAME.add("公冶");
        DOUBLE_SURNAME.add("太叔");
        DOUBLE_SURNAME.add("申屠");
        DOUBLE_SURNAME.add("公孙");
        DOUBLE_SURNAME.add("慕容");
        DOUBLE_SURNAME.add("仲孙");
        DOUBLE_SURNAME.add("钟离");
        DOUBLE_SURNAME.add("长孙");
        DOUBLE_SURNAME.add("宇文");
        DOUBLE_SURNAME.add("司徒");
        DOUBLE_SURNAME.add("鲜于");
        DOUBLE_SURNAME.add("司空");
        DOUBLE_SURNAME.add("闾丘");
        DOUBLE_SURNAME.add("子车");
        DOUBLE_SURNAME.add("亓官");
        DOUBLE_SURNAME.add("司寇");
        DOUBLE_SURNAME.add("巫马");
        DOUBLE_SURNAME.add("公西");
        DOUBLE_SURNAME.add("颛孙");
        DOUBLE_SURNAME.add("壤驷");
        DOUBLE_SURNAME.add("公良");
        DOUBLE_SURNAME.add("漆雕");
        DOUBLE_SURNAME.add("乐正");
        DOUBLE_SURNAME.add("宰父");
        DOUBLE_SURNAME.add("谷梁");
        DOUBLE_SURNAME.add("拓跋");
        DOUBLE_SURNAME.add("夹谷");
        DOUBLE_SURNAME.add("轩辕");
        DOUBLE_SURNAME.add("令狐");
        DOUBLE_SURNAME.add("段干");
        DOUBLE_SURNAME.add("百里");
        DOUBLE_SURNAME.add("呼延");
        DOUBLE_SURNAME.add("东郭");
        DOUBLE_SURNAME.add("南门");
        DOUBLE_SURNAME.add("羊舌");
        DOUBLE_SURNAME.add("微生");
        DOUBLE_SURNAME.add("公户");
        DOUBLE_SURNAME.add("公玉");
        DOUBLE_SURNAME.add("公仪");
        DOUBLE_SURNAME.add("梁丘");
        DOUBLE_SURNAME.add("公仲");
        DOUBLE_SURNAME.add("公上");
        DOUBLE_SURNAME.add("公门");
        DOUBLE_SURNAME.add("公山");
        DOUBLE_SURNAME.add("公坚");
        DOUBLE_SURNAME.add("左丘");
        DOUBLE_SURNAME.add("公伯");
        DOUBLE_SURNAME.add("西门");
        DOUBLE_SURNAME.add("公祖");
        DOUBLE_SURNAME.add("第五");
        DOUBLE_SURNAME.add("公乘");
        DOUBLE_SURNAME.add("贯丘");
        DOUBLE_SURNAME.add("公皙");
        DOUBLE_SURNAME.add("南荣");
        DOUBLE_SURNAME.add("东里");
        DOUBLE_SURNAME.add("东宫");
        DOUBLE_SURNAME.add("仲长");
        DOUBLE_SURNAME.add("子书");
        DOUBLE_SURNAME.add("子桑");
        DOUBLE_SURNAME.add("即墨");
        DOUBLE_SURNAME.add("淳于");
        DOUBLE_SURNAME.add("达奚");
        DOUBLE_SURNAME.add("褚师");
        DOUBLE_SURNAME.add("吴铭");
        DOUBLE_SURNAME.add("纳兰");
        DOUBLE_SURNAME.add("归海");
        DOUBLE_SURNAME.add("刘付");
    }


    /**
     * 马云 -> 马*
     * 马化腾 -> 马**
     * 李彦宏宏宏宏宏宏宏宏 -> 李****
     * 诸葛 -> 诸*
     * 诸葛亮 -> 诸葛*
     * 诸葛孔明 -> 诸葛**
     * 刘付 -> 刘*
     * 刘付阳 -> 刘付*
     *
     * @author Neo
     * @since 2021/6/18 16:54
     */
    public static String process(String name) {
        if (StringUtils.isBlank(name)) {
            return StringUtils.EMPTY;
        }

        String lastName = StringUtils.EMPTY;
        int countLength = StringUtils.length(name);
        if (countLength > 2) {
            for (String s : DOUBLE_SURNAME) {
                if (StringUtils.startsWith(name, s)) {
                    lastName = s;
                    break;
                }
            }
        }

        if (StringUtils.isBlank(lastName)) {
            lastName += StringUtils.substring(name, 0, 1);
        }

        int length = Math.min(countLength, MAX_LENGTH);

        return StringUtils.rightPad(lastName, length, PADDING_CHAR);
    }


    public static void main(String[] args) {
        List<String> names = ImmutableList.of("马云", "马化腾", "李彦宏宏宏宏宏宏宏宏", "诸葛", "诸葛亮", "诸葛孔明", "刘付", "刘付阳");
        for (String name : names) {
            System.out.println(name + " -> " + process(name));
        }
    }
}

```





# Java 万物皆可 Tree 工具类

> https://blog.csdn.net/after95/article/details/112188070