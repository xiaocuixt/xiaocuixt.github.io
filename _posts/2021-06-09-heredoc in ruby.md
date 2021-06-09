---
layout: post
title:  "Heredoc In Ruby"
date:   2021-06-08 19:30:00
categories: ruby
comments: true
---

<p>heredoc是一种定义多行字符串的方法,该字符串中可以包含换行，缩进等。可以用来以文档的方式嵌套文本片段，如Sql, Html等多行代码片段。</p>

<p>简单的写法</p>
```ruby
query = <<-SQL
  SELECT * FROM users
  WHERE male = true
SQL

#=> "  SELECT * FROM users\n  WHERE male = true\n"
```

这就是在ruby中定义一个heredoc的语法，以<<-开头，后面跟的SQL是该heredoc的名字，中间是heredoc的内容，然后使用名字SQL来结束该heredoc. 这里SQL可以替换成任何名字(如HTML, TEXT, HEREDOC等)


另一种实现heredoc的方式是%Q

```ruby
query = %Q(
  SELECT * FROM users
  WHERE male = true
)

#=>  "\n  SELECT * FROM users\n  WHERE male = true\n"
```
<p>%Q方法和<<-功能一样，但%Q会在字符串的首尾分别生成一个换行符。</p>

<p>heredoc提供了删除的换行符的方式</p>
```ruby
query = <<-SQL.chomp
  SELECT * FROM users
  WHERE male = true
SQL

#=> "  SELECT * FROM users\n  WHERE male = true"  # chomp删除字符串结尾的分隔符
```

<p>heredoc还支持插值</p>

```ruby
type  = "male"
table = "users"
query = <<-SQL
  SELECT * FROM #{table}
  WHERE #{type} = true
SQL

#=> "  SELECT * FROM users\n  WHERE male = true\n"
```

<p>还可以让插值失效，直接以字符串展示(给heredoc文档名字加单引号)</p>

```ruby
type  = "male"
table = "users"
query = <<-'SQL'
  SELECT * FROM #{table}
  WHERE #{type} = true
SQL

#=> "  SELECT * FROM \#{table}\n  WHERE \#{type} = true\n"
```

<p>我们看到上面生成的字符串前面都有一行缩进，实际中可能并不需要，怎么移除呢? ruby提供了~来实现这个（ruby2.3版本之后，该版本之前请使用rails active support提供的strip_heredoc, 下文会提及）</p>

```ruby
query = <<~SQL
  SELECT * FROM users
  WHERE male = true
SQL
#=> "SELECT * FROM users\nWHERE male = true\n"

query = <<~SQL.strip
  SELECT * FROM users
  WHERE male = true
SQL

#=> "SELECT * FROM users\nWHERE male = true"  # strip删除字符串中的空格
```

<p>rails中也提供了一个方法来删除不需要的空格strip_heredoc</p>

```ruby
query = <<-SQL.strip_heredoc
  SELECT * FROM users
  WHERE male = true
SQL
#=> "SELECT * FROM users\nWHERE male = true\n"
```

<p>但是strip_heredoc只是删除“最少缩进的非空行”，所以会出现下面的结果: third line.的缩进并没有删除</p>

```ruby
puts <<-USAGE.strip_heredoc
  first line.
  second line.
    third line.
USAGE
# 打印结果
first line.
second line.
  third line.
```

<p>也可以将多行文本转化为一行, 这种在组织sql语句时比较常用。</p>

```ruby
query = <<-SQL.squish
  SELECT * FROM users
  WHERE male = true
SQL

#=> "SELECT * FROM users WHERE male = true"
```
<p>还可以通过``符号调用shell命令</p>

```ruby
str = <<~`HEREDOC`
        date
  HEREDOC

  #=> "2021年 6月 9日 星期三 18时52分50秒 CST\n"
```

以上基本上涵盖了heredoc的常规用法，后面应用到新的实现方式再做更新。