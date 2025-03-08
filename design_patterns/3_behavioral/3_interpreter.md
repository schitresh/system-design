## Interpreter
- Defines a way to interpret and evaluate language grammer or operations

## Example
```rb
# Abstract Expression
class HtmlExpression
  def initialize(expression)
    @expression = expression
  end

  def parse
    raise_not_implemented_error
  end
end

# Non-terminal Expressions
class HeaderBeginExpression < HtmlExpression
  def parse
    '<head>'
  end
end

class HeaderEndExpression < HtmlExpression
  def parse
    '</head>'
  end
end

class TitleBeginExpression < HtmlExpression
  def parse
    '<title>'
  end
end

class TitleEndExpression < HtmlExpression
  def parse
    '</title>'
  end
end

# Parser
class HtmlParser
  def initialize(expressions)
    @expression_list = Array.new
    expressions.split("\n").each do |expression|
      case expression
      when 'he'
        @expression_list << HeaderBeginExpression.new(expression)
      when '/he'
        @expression_list << HeaderEndExpression.new(expression)
      when 'ti'
        @expression_list << TitleBeginExpression.new(expression)
      when '/ti'
        @expression_list << TitleEndExpression.new(expression)
      else
        @expression_list << expression
      end
    end
  end

  def parse
    html = "<!DOCTYPE html>\n"
    @expression_list.each do |expression|
      if expression.class.ancestors.include?(HtmlExpression)
        html << expression.parse
      else
        html << expression
      end
      html << "\n"
    end
    html << '</html>'
  end
end

# Context
class HtmlContext
  attr_accessor :expressions

  def initialize(expressions)
    @expressions = expressions
  end

  def parse
    HtmlParser.new(@expressions).parse
  end
end

# Client
text = <<"EOS"
he
ti
My Title
/ti
/he
EOS

context = HtmlContext.new(text)
html = context.parse
puts html
#=> <!DOCTYPE html>
#=> <head>
#=> <title>
#=> My Title
#=> </title>
#=> </head>
#=> </html>
```
