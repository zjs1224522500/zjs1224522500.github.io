---
title: 'Java.String'
date: 2017-5-6 16:57:41
tags: JavaSE
---
# Java.String
### String 类的两种赋值方式
- String 可以表示出一个字符串,根据String的源码我们会发现String类实际上是使用字符数组char[]存储的.

- 如下为`String`的源码实现
<!-- more -->

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];

    /** Cache the hash code for the string */
    private int hash; // Default to 0

    /** use serialVersionUID from JDK 1.0.2 for interoperability */
    private static final long serialVersionUID = -6849794470754667710L;

    public String() {
        this.value = "".value;
    }

    public String(String original) {
        this.value = original.value;
        this.hash = original.hash;
    }

    public String(char value[]) {
        this.value = Arrays.copyOf(value, value.length);
    }

    public String(char value[], int offset, int count) {
        if (offset < 0) {
            throw new StringIndexOutOfBoundsException(offset);
        }
        if (count <= 0) {
            if (count < 0) {
                throw new StringIndexOutOfBoundsException(count);
            }
            if (offset <= value.length) {
                this.value = "".value;
                return;
            }
        }
        // Note: offset or count might be near -1>>>1.
        if (offset > value.length - count) {
            throw new StringIndexOutOfBoundsException(offset + count);
        }
        this.value = Arrays.copyOfRange(value, offset, offset+count);
    }

    public String(int[] codePoints, int offset, int count) {
        if (offset < 0) {
            throw new StringIndexOutOfBoundsException(offset);
        }
        if (count <= 0) {
            if (count < 0) {
                throw new StringIndexOutOfBoundsException(count);
            }
            if (offset <= codePoints.length) {
                this.value = "".value;
                return;
            }
        }
        // Note: offset or count might be near -1>>>1.
        if (offset > codePoints.length - count) {
            throw new StringIndexOutOfBoundsException(offset + count);
        }

        final int end = offset + count;

        // Pass 1: Compute precise size of char[]
        int n = count;
        for (int i = offset; i < end; i++) {
            int c = codePoints[i];
            if (Character.isBmpCodePoint(c))
                continue;
            else if (Character.isValidCodePoint(c))
                n++;
            else throw new IllegalArgumentException(Integer.toString(c));
        }

        // Pass 2: Allocate and fill in char[]
        final char[] v = new char[n];

        for (int i = offset, j = 0; i < end; i++, j++) {
            int c = codePoints[i];
            if (Character.isBmpCodePoint(c))
                v[j] = (char)c;
            else
                Character.toSurrogates(c, v, j++);
        }

        this.value = v;
    }

    public String(byte bytes[], int offset, int length, String charsetName)
            throws UnsupportedEncodingException {
        if (charsetName == null)
            throw new NullPointerException("charsetName");
        checkBounds(bytes, offset, length);
        this.value = StringCoding.decode(charsetName, bytes, offset, length);
    }

    public String(byte bytes[], int offset, int length, Charset charset) {
        if (charset == null)
            throw new NullPointerException("charset");
        checkBounds(bytes, offset, length);
        this.value =  StringCoding.decode(charset, bytes, offset, length);
    }

    public String(byte bytes[], String charsetName)
            throws UnsupportedEncodingException {
        this(bytes, 0, bytes.length, charsetName);
    }

    public String(byte bytes[], Charset charset) {
        this(bytes, 0, bytes.length, charset);
    }

    public String(byte bytes[], int offset, int length) {
        checkBounds(bytes, offset, length);
        this.value = StringCoding.decode(bytes, offset, length);
    }

    public String(byte bytes[]) {
        this(bytes, 0, bytes.length);
    }

    public String(StringBuffer buffer) {
        synchronized(buffer) {
            this.value = Arrays.copyOf(buffer.getValue(), buffer.length());
        }
    }

    public String(StringBuilder builder) {
        this.value = Arrays.copyOf(builder.getValue(), builder.length());
    }
```
- 常见的两种String赋值方式
    - String str1 = new String("name");
    - String str2 = "name";

- 两种赋值方式分析
```java
//会创建两个字符串对象，一个在常量池中创建便于下一次使用，一个在堆内存中创建
String str1 = new String("name");
//最多创建一个字符串对象，有可能不用创建对象（如果常量池中已经存在相应的常量）
String str2 = "name";//推荐使用
```
[内存分析](http://img.blog.csdn.net/20170723130814511?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM5ODM2MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![内存分析](http://img.blog.csdn.net/20170723130814511?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM5ODM2MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### String类编译期与运行期分析
- 首先明确对象之间 "==" 是比较两个对象的地址。
```java
public class StringDemo {
	
	public static void main(String[] args) {
		
		// 情况一   结果为true
		String aString = "a1";
		String a1String = "a" + 1;
		System.out.println(aString == a1String);
		
		
		// 情况二    结果为false
		String bString = "b1";
		int bb = 1;
		String b1String = "b" + bb;
		System.out.println(bString == b1String);
		
		
		// 情况三    结果为true
		String cString = "c1";
		final int cc = 1;
		String c1String = "c" + cc;
		System.out.println(cString == c1String);
	
	
		// 情况四     结果为false
		String dString = "dd";
		final int dd = getDD();
		String d1String = dString + dd;
		System.out.println(dString == d1String);
		
	}
	
	public static int getDD(){
		return 1;
	}
}
```

- 情况二为false，因为 b1String 在编译期时是还没有完全确定下来的，因为 bb 是一个变量，只有在执行期才能加载到。
- 情况四为false，注意与情况三进行对比，虽然使用了final关键字，但是根据函数返回值确定下来的，而函数要执行仍然是在运行期的时候确定的，编译期仍然无法确定。

### String 类字符与字符串操作方法

- 字符与字符串操作

![image](http://img.blog.csdn.net/20170723131120545?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM5ODM2MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- 字节与字符串操作 

![image](http://img.blog.csdn.net/20170723131221770?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM5ODM2MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- 是否以指定内容开头或结尾

![!\[image\](http://f.hiphotos.baidu.com/image/pic/item/9f510fb30f2442a7444789b1db43ad4bd113025c.jpg](http://img.blog.csdn.net/20170723131306585?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM5ODM2MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast))

- String 类替换操作

![image](http://img.blog.csdn.net/20170723131514431?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM5ODM2MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- String 类字符串截取操作

![image](http://img.blog.csdn.net/20170723131533882?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM5ODM2MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- 字符串拆分操作

![image](http://img.blog.csdn.net/20170723131613731?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM5ODM2MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- String 类字符串查找操作

![image](http://img.blog.csdn.net/20170723131637723?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM5ODM2MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- String 类其他操作方法

![这里写图片描述](http://img.blog.csdn.net/20170723131703342?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM5ODM2MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- 更多请见相关api文档

- 常见用例
```java
System.out.println("abcde".charAt(3));//d
System.out.println("abcde".toCharArray()[2]);//c
byte[] bytes = "abcde".getBytes();
for(int i = 0; i < bytes.length; i++){
	System.out.println(bytes[i]);
}
System.out.println(new String(bytes).toString());
System.out.println(new String(bytes,0,2).toString());//ab
System.out.println(new String(bytes,"utf-8").toString());

System.out.println("abcde".startsWith("ab"));//true
System.out.println("abcde".startsWith("cd",2));//true
System.out.println("abcde".endsWith("de"));//true

System.out.println("abc,de".replace(",", ":").toString());//abc:de
System.out.println("abcde".replace("ab", "AB"));//ABcde
		
//匹配正则表达式中26个字母替换为相应的字符串
System.out.println("abcde".replaceAll("[a-z]", "*"));//*****
System.out.println("abcde".replaceFirst("[a-z]", "*"));//*bcde

//包括起始位置
System.out.println("abcde".substring(2));//cde
//包括起始位置，不包括结束位置
System.out.println("abcde".substring(1, 3));//bc 

String[] value = "ab-cde".split("-");//传入的参数为正则表达式
String[] value2 = "ab-c-de".split("-",3);//拆分成三个字符串数组
for(String s:value){
	System.out.print(s);
}
System.out.println();
for(String s:value2){
	System.out.print(s);
}

System.out.println("abcde".contains("abc"));//true
System.out.println("abcde".indexOf("a"));//0  参数虽然为整形，但会转为unicode字符
System.out.println("abcde".indexOf("bc"));//0 返回首地址
System.out.println("abcded".lastIndexOf("d"));//5
System.out.println("abcdede".lastIndexOf("de"));//5

System.out.println("abcde".isEmpty());//false
System.out.println("abcde".length());//5
System.out.println("abcde".toUpperCase());//ABCDE
System.out.println("ABCDE".toLowerCase());//abcde
System.out.println(" abcde ".trim());//abcde
System.out.println("abc".concat("de"));//abcde
```

- 源码
```java
    public char charAt(int index) {
        if ((index < 0) || (index >= value.length)) {
            throw new StringIndexOutOfBoundsException(index);
        }
        return value[index];
    }
    
    public char[] toCharArray() {
    // Cannot use Arrays.copyOf because of class initialization order issues
        char result[] = new char[value.length];
        System.arraycopy(value, 0, result, 0, value.length);
        return result;
    }
    
    // 使用平台默认的编码转换成字节数组，默认gbk，根据虚拟机
    public byte[] getBytes(String charsetName)
            throws UnsupportedEncodingException {
        if (charsetName == null) throw new NullPointerException();
        return StringCoding.encode(charsetName, value, 0, value.length);
    }
    
    public String(byte bytes[]) {
        this(bytes, 0, bytes.length);
    }
    
    public String(byte bytes[], int offset, int length) {
        checkBounds(bytes, offset, length);
        this.value = StringCoding.decode(bytes, offset, length);
    }
    
    public String(byte bytes[], String charsetName)
            throws UnsupportedEncodingException {
        this(bytes, 0, bytes.length, charsetName);
    }
    
    public boolean startsWith(String prefix) {
        return startsWith(prefix, 0);
    }
    
    public boolean startsWith(String prefix, int toffset) {
        char ta[] = value;//调用该方法的对象的value
        int to = toffset;//偏移量
        char pa[] = prefix.value;//前缀的value
        int po = 0;//前缀的偏移量
        int pc = prefix.value.length;//前缀的总数
        // Note: toffset might be near -1>>>1.
        if ((toffset < 0) || (toffset > value.length - pc)) {
            return false;
        }
        while (--pc >= 0) {
            if (ta[to++] != pa[po++]) {
                return false;
            }
        }
        return true;
    }
    
    public boolean endsWith(String suffix) {
        return startsWith(suffix, value.length - suffix.value.length);
    }
    
    
    public String replace(char oldChar, char newChar) {
        if (oldChar != newChar) {
            int len = value.length;
            int i = -1;
            char[] val = value; /* avoid getfield opcode */

            while (++i < len) {
                if (val[i] == oldChar) {
                    break;
                }
            }
            if (i < len) {
                char buf[] = new char[len];
                for (int j = 0; j < i; j++) {
                    buf[j] = val[j];
                }
                while (i < len) {
                    char c = val[i];
                    buf[i] = (c == oldChar) ? newChar : c;
                    i++;
                }
                return new String(buf, true);
            }
        }
        return this;
    }
    
    public String replaceFirst(String regex, String replacement) {
        return Pattern.compile(regex).matcher(this).replaceFirst(replacement);
    }
    
    public String replaceAll(String regex, String replacement) {
        return Pattern.compile(regex).matcher(this).replaceAll(replacement);
    }
    
    public String replace(CharSequence target, CharSequence replacement) {
        return Pattern.compile(target.toString(), Pattern.LITERAL).matcher(
                this).replaceAll(Matcher.quoteReplacement(replacement.toString()));
    }
    
    public String substring(int beginIndex) {
        if (beginIndex < 0) {
            throw new StringIndexOutOfBoundsException(beginIndex);
        }
        int subLen = value.length - beginIndex;
        if (subLen < 0) {
            throw new StringIndexOutOfBoundsException(subLen);
        }
        return (beginIndex == 0) ? this : new String(value, beginIndex, subLen);
    }
    
    public String substring(int beginIndex, int endIndex) {
        if (beginIndex < 0) {
            throw new StringIndexOutOfBoundsException(beginIndex);
        }
        if (endIndex > value.length) {
            throw new StringIndexOutOfBoundsException(endIndex);
        }
        int subLen = endIndex - beginIndex;
        if (subLen < 0) {
            throw new StringIndexOutOfBoundsException(subLen);
        }
        return ((beginIndex == 0) && (endIndex == value.length)) ? this
                : new String(value, beginIndex, subLen);
    }
    
    public String[] split(String regex, int limit) {
        /* fastpath if the regex is a
         (1)one-char String and this character is not one of the
            RegEx's meta characters ".$|()[{^?*+\\", or
         (2)two-char String and the first char is the backslash and
            the second is not the ascii digit or ascii letter.
         */
        char ch = 0;
        if (((regex.value.length == 1 &&
             ".$|()[{^?*+\\".indexOf(ch = regex.charAt(0)) == -1) ||
             (regex.length() == 2 &&
              regex.charAt(0) == '\\' &&
              (((ch = regex.charAt(1))-'0')|('9'-ch)) < 0 &&
              ((ch-'a')|('z'-ch)) < 0 &&
              ((ch-'A')|('Z'-ch)) < 0)) &&
            (ch < Character.MIN_HIGH_SURROGATE ||
             ch > Character.MAX_LOW_SURROGATE))
        {
            int off = 0;
            int next = 0;
            boolean limited = limit > 0;
            ArrayList<String> list = new ArrayList<>();
            while ((next = indexOf(ch, off)) != -1) {
                if (!limited || list.size() < limit - 1) {
                    list.add(substring(off, next));
                    off = next + 1;
                } else {    // last one
                    //assert (list.size() == limit - 1);
                    list.add(substring(off, value.length));
                    off = value.length;
                    break;
                }
            }
            // If no match was found, return this
            if (off == 0)
                return new String[]{this};

            // Add remaining segment
            if (!limited || list.size() < limit)
                list.add(substring(off, value.length));

            // Construct result
            int resultSize = list.size();
            if (limit == 0) {
                while (resultSize > 0 && list.get(resultSize - 1).length() == 0) {
                    resultSize--;
                }
            }
            String[] result = new String[resultSize];
            return list.subList(0, resultSize).toArray(result);
        }
        return Pattern.compile(regex).split(this, limit);
    }
    
    public String[] split(String regex) {
        return split(regex, 0);
    }
    
    public int indexOf(int ch, int fromIndex) {
        final int max = value.length;
        if (fromIndex < 0) {
            fromIndex = 0;
        } else if (fromIndex >= max) {
            // Note: fromIndex might be near -1>>>1.
            return -1;
        }

        if (ch < Character.MIN_SUPPLEMENTARY_CODE_POINT) {
            // handle most cases here (ch is a BMP code point or a
            // negative value (invalid code point))
            final char[] value = this.value;
            for (int i = fromIndex; i < max; i++) {
                if (value[i] == ch) {
                    return i;
                }
            }
            return -1;
        } else {
            return indexOfSupplementary(ch, fromIndex);
        }
    }
    
    public int indexOf(String str) {
        return indexOf(str, 0);
    }
    
    public int indexOf(String str, int fromIndex) {
        return indexOf(value, 0, value.length,
                str.value, 0, str.value.length, fromIndex);
    }
    
    public int indexOf(int ch) {
        return indexOf(ch, 0);
    }
    
    public boolean contains(CharSequence s) {
        return indexOf(s.toString()) > -1;
    }
    
    public int lastIndexOf(int ch, int fromIndex) {
        if (ch < Character.MIN_SUPPLEMENTARY_CODE_POINT) {
            // handle most cases here (ch is a BMP code point or a
            // negative value (invalid code point))
            final char[] value = this.value;
            int i = Math.min(fromIndex, value.length - 1);
            for (; i >= 0; i--) {
                if (value[i] == ch) {
                    return i;
                }
            }
            return -1;
        } else {
            return lastIndexOfSupplementary(ch, fromIndex);
        }
    }
    
    public int lastIndexOf(int ch) {
        return lastIndexOf(ch, value.length - 1);
    }
    
    public boolean isEmpty() {
        return value.length == 0;
    }
    
    public int length() {
        return value.length;
    }
    
    public String trim() {
        int len = value.length;
        int st = 0;
        char[] val = value;    /* avoid getfield opcode */

        while ((st < len) && (val[st] <= ' ')) {
            st++;
        }
        while ((st < len) && (val[len - 1] <= ' ')) {
            len--;
        }
        return ((st > 0) || (len < value.length)) ? substring(st, len) : this;
    }
    
    public String concat(String str) {
        int otherLen = str.length();
        if (otherLen == 0) {
            return this;
        }
        int len = value.length;
        char buf[] = Arrays.copyOf(value, len + otherLen);
        str.getChars(buf, len);
        return new String(buf, true);
    }
```

