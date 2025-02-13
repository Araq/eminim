# Eminim — JSON deserialization macro for Nim

## About

This package provides a ``to`` macro which deserializes the specified type from a ``Stream``. It
generates code, in compile time, to use directly the JsonParser, without creating intermediate JsonNode(s).

For example:

```nim
   type Foo = ref object
      value: int
   let s = newStringStream("{\"value\": 1}")
   let a = s.to(Foo)
```

Produces this code:

```nim
   proc pack(s: Stream): Foo =
      var p: JsonParser
      open(p, s, "unknown file")
      discard getTok(p)
      new(result)
      eat(p, tkCurlyLe)
      while p.tok != tkCurlyRi:
         if p.tok != tkString:
            raiseParseErr(p, "string literal as key")
         let key = move(p.a)
         discard getTok(p)
         eat(p, tkColon)
         case key
         of "value":
            if p.tok == tkInt:
               result.value = int(parseInt(p.a))
               discard getTok(p)
            else:
               raiseParseErr(p, "int")
         else:
            raiseParseErr(p, "object field")
         if p.tok != tkComma:
            break
         discard getTok(p)
      eat(p, tkCurlyRi)
      eat(p, tkEof)
      close(p)

   pack(s)
```
